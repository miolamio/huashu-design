# scripts/

## Overview

This directory holds the export toolchain that the skill invokes at runtime — it is **not** a build system. There is no `package.json`, no test runner, no linter. Each script is standalone, self-documents its usage in its own file header, and is called directly by the agent when a deliverable (MP4, GIF, PDF, PPTX, screenshots) is requested.

Two pipelines live here:

- **Animation pipeline:** `render-video.js` -> `convert-formats.sh` -> `add-music.sh` (HTML animation -> MP4 -> 60fps MP4 + GIF -> MP4 with BGM).
- **Deck pipeline:** `export_deck_pdf.mjs` / `export_deck_stage_pdf.mjs` / `export_deck_pptx.mjs` (calls `html2pptx.js`) — multi-file deck or single-file `<deck-stage>` HTML to PDF or editable PPTX.

`verify.py` is a separate human-driven QA wrapper around Playwright (screenshots, multi-viewport, console errors).

## Prerequisites

Global, install once:

- **Node.js 18+**
- **Playwright (global):** `npm install -g playwright` (used by `render-video.js` via `NODE_PATH=$(npm root -g)`)
- **ffmpeg** on `PATH` (used by `render-video.js`, `convert-formats.sh`, `add-music.sh`)
- **Python 3** with Playwright: `pip install playwright && playwright install chromium` (only for `verify.py`)

Per-deck (install in the directory you run the deck-export script from):

- **PDF export (multi-file deck):** `npm install playwright pdf-lib`
- **PDF export (single-file deck-stage):** `npm install playwright`
- **PPTX export:** `npm install playwright pptxgenjs sharp`

## Scripts

### `render-video.js`

- **Purpose:** Render an HTML animation to MP4 via Playwright `recordVideo` plus ffmpeg post-processing.
- **Prereqs:** Global Playwright (`npm install -g playwright`); `ffmpeg` on `PATH`.
- **Invocation:**
  ```
  NODE_PATH=$(npm root -g) node scripts/render-video.js <html-file> \
    [--duration=30] [--width=1920] [--height=1080] \
    [--trim=<seconds>] [--fontwait=1.5] [--readytimeout=8] \
    [--keep-chrome]
  ```
- **Output:** MP4 written next to the input HTML, same basename with `.mp4` suffix.
- **Reference:** `references/video-export.md`, `references/animation-pitfalls.md` (items 7-13).

### `convert-formats.sh`

- **Purpose:** Convert an MP4 to a 60fps MP4 plus a palette-optimized GIF.
- **Prereqs:** `ffmpeg` on `PATH`.
- **Invocation:**
  ```
  bash scripts/convert-formats.sh <input.mp4> [gif_width] [--minterpolate]
  ```
- **Output:** Next to the input — `<name>-60fps.mp4` (1920x1080, 60fps, frame-duplicated by default) and `<name>.gif` (default 960px wide, 15fps, palette-optimized, two-pass).
- **Reference:** `references/video-export.md`, `references/animation-pitfalls.md` item 14 (60fps frame-duplication vs minterpolate compatibility).

### `add-music.sh`

- **Purpose:** Mix a BGM track into an MP4 (atrim to match video duration, 0.3s fade in, 1.0s fade out).
- **Prereqs:** `ffmpeg` and `ffprobe` on `PATH`. BGM presets live in `assets/bgm-<mood>.mp3`.
- **Invocation:**
  ```
  bash scripts/add-music.sh <input.mp4> [--mood=<name>] [--music=<path>] [--out=<path>]
  ```
  Mood library: `tech` (default), `ad`, `educational`, `educational-alt`, `tutorial`, `tutorial-alt`. `--music=<path>` overrides `--mood`. Legacy positional form `bash add-music.sh in.mp4 music.mp3 out.mp4` still works.
- **Output:** `<input-basename>-bgm.mp4` next to the input by default, or the path passed to `--out=`. Video stream is copied (no re-encode); audio is AAC 192k.
- **Reference:** `references/audio-design-rules.md`, `references/sfx-library.md`.

### `export_deck_pdf.mjs`

- **Purpose:** Export a multi-file slide deck (one HTML per slide) to a single vector PDF (text stays copyable and searchable).
- **Prereqs:** `npm install playwright pdf-lib` in the working directory.
- **Invocation:**
  ```
  node scripts/export_deck_pdf.mjs --slides <dir> --out <file.pdf> [--width 1920] [--height 1080]
  ```
  Slides are sorted by filename (`01-xxx.html` -> `02-xxx.html` -> ...).
- **Output:** Single PDF at `--out`. Backgrounds and graphics retain 1:1 fidelity via Playwright Chromium rendering.
- **Reference:** `references/slide-decks.md`.

### `export_deck_stage_pdf.mjs`

- **Purpose:** PDF export for the **single-file** `<deck-stage>` architecture (every slide is a `<section>` inside one `<deck-stage>` web component). Pulls all sections out of the Shadow DOM slot before printing.
- **Prereqs:** `npm install playwright` in the working directory. Fonts must be loadable by Chromium (local fonts or Google Fonts).
- **Invocation:**
  ```
  node scripts/export_deck_stage_pdf.mjs --html <deck.html> --out <file.pdf> [--width 1920] [--height 1080]
  ```
- **Output:** Single PDF at `--out`.
- **Reference:** `references/slide-decks.md`. Use `export_deck_pdf.mjs` (above) for multi-file decks; this script is **only** for `deck_stage.js`-based single-file decks.

### `export_deck_pptx.mjs`

- **Purpose:** Export a multi-file deck to **editable** PPTX. Calls `html2pptx.js` to translate the DOM element-by-element into native PowerPoint objects, so text becomes real text boxes.
- **Prereqs:** `npm install playwright pptxgenjs sharp` in the working directory.
- **Invocation:**
  ```
  node scripts/export_deck_pptx.mjs --slides <dir> --out <file.pptx>
  ```
  Slides are sorted by filename. Body size is 960pt x 540pt (`LAYOUT_WIDE`, 13.333" x 7.5").
- **Output:** Single PPTX at `--out`.
- **Reference:** `references/editable-pptx.md`. **The HTML must satisfy the four hard constraints in that file** — visual-freedom HTML (CSS gradients, complex SVG, animations) cannot pass; route those through PDF instead.

### `html2pptx.js`

- **Purpose:** DOM -> PPTX element translator. Converts an HTML slide to a `pptxgenjs` slide with positioned elements (text, images, shapes, bullet lists, gradients, borders, margins). Extracts placeholder elements (`class="placeholder"`) with `{ id, x, y, w, h }` for chart insertion.
- **Prereqs:** `playwright` and `sharp` (both pulled in by `export_deck_pptx.mjs`). Validates that body width/height match the presentation layout and throws if content overflows.
- **Invocation:** Called by `export_deck_pptx.mjs` — **not invoked directly**. Library entry point:
  ```js
  const { slide, placeholders } = await html2pptx('slide.html', pptx);
  ```
- **Output:** Mutates the passed `pptx` slide; returns `{ slide, placeholders }`.
- **Reference:** `references/editable-pptx.md`.

### `verify.py`

- **Purpose:** Playwright wrapper for human-driven QA of HTML design outputs — screenshots, multi-viewport, per-slide capture, console-error scraping.
- **Prereqs:** `pip install playwright && playwright install chromium`.
- **Invocation:**
  ```
  python scripts/verify.py path/to/design.html                      # basic
  python scripts/verify.py design.html --viewports 1920x1080,375x667  # multiple viewports
  python scripts/verify.py deck.html --slides 10                    # capture first N slides one by one
  python scripts/verify.py design.html --output ./screenshots/      # output directory
  python scripts/verify.py design.html --show                       # non-headless
  ```
- **Output:** PNG screenshots and console-error logs, by default under `<input-dir>/screenshots/` (override with `--output`).
- **Reference:** `references/verification.md`.

## Common gotchas

- **`render-video.js` requires `window.__ready = true`.** The recorder waits for that flag to mark t=0; without it, the head of the MP4 will be 1-2s of black frames (the script falls back to `--fontwait=1.5s`). `assets/animations.jsx` Stage sets it automatically — see the runtime contract at the top of that file. Hand-written Stage/Sprite must implement it manually plus disable `loop` when `window.__recording === true` (see `references/animation-pitfalls.md` items 12 and 13).
- **`export_deck_pptx.mjs` only handles HTML that satisfies four hard constraints** (text wrapped in `<p>`/`<h1>-<h6>`, no CSS gradients, `<p>`/`<h*>` carry no background/border/shadow, no `background-image` on divs). Visually driven HTML almost never passes retroactively — write to those constraints from the first line, or route to PDF via `export_deck_pdf.mjs` / `export_deck_stage_pdf.mjs`. See `references/editable-pptx.md`.
- **`html2pptx.js` is a library, not a CLI.** It is called by `export_deck_pptx.mjs`. Calling it standalone is unsupported.
- **`export_deck_stage_pdf.mjs` is only for single-file `<deck-stage>` decks.** Multi-file decks must use `export_deck_pdf.mjs` — the stage variant exists because deck-stage's Shadow DOM `::slotted(section) { display: none }` blocks the active-slide-only PDF that a naive `page.pdf()` produces.
- **`convert-formats.sh --minterpolate` has known QuickTime / Safari compatibility issues.** Default mode (`fps=60` frame duplication) is broadly compatible; only enable minterpolate for heavy translate/scale motion and test the output before delivering.
- **`add-music.sh` resolves moods relative to `../assets/`** (i.e. `assets/bgm-<mood>.mp3`). Run from the repo or pass an absolute `--music=<path>`. Video is stream-copied — input must already be an MP4 with an H.264 video stream.
- **Deck-export scripts each declare their own npm dependencies** — install them in the working directory before invoking. They are not pre-installed at the repo root (no `package.json` here).
