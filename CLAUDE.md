# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repo is

`huashu-design` is a **markdown-based skill** (distributed via `npx skills add alchaincyf/huashu-design`), not a runtime application. The "product" is the skill content itself — `SKILL.md` plus its supporting `references/`, `assets/`, and `scripts/`. Future Claude instances editing this repo are usually maintaining or extending the skill, **not** invoking it. To invoke it from a fresh session you would install it as a skill, not run it from this checkout.

The skill is agent-agnostic by design (Claude Code, Codex, Cursor, etc.). Anything skill-runtime-specific belongs in the "Cross Agent" section of `SKILL.md` or `references/`, not hardcoded.

## Repo entry points (read first when editing)

- `SKILL.md` — main skill content; loaded into the agent at trigger time. Edits here have the largest blast radius.
- `references/*.md` — pulled in on-demand by the routing table at the bottom of `SKILL.md`. New deep content goes here, not into `SKILL.md`.
- `assets/*.jsx` — Starter Components (`ios_frame.jsx`, `animations.jsx`, `deck_stage.js`, `design_canvas.jsx`, `browser_window.jsx`, `macos_window.jsx`, `android_frame.jsx`, `deck_index.html`). These are copied **inline** into user-generated HTML (single-file React + Babel under `file://`), so they cannot rely on a module system or external loaders.
- `scripts/` — export toolchain invoked by the agent during skill execution (not a build step). See "Scripts" below.
- `demos/` — `c1`–`c6` capability demos and `w1`–`w3` workflow demos, both `.html` and `-en.html` variants. Used as showcase material in the README.
- `test-prompts.json` — sample prompts that should trigger the skill; useful for sanity-checking trigger phrasing in `SKILL.md` frontmatter.

## Architecture in one paragraph

`SKILL.md` is a large, layered document where order encodes priority: **Core Principle #0 (fact verification)** → **Core Asset Protocol (§1.a)** → **Junior Designer workflow** → **Anti-AI-slop rules** → **Design Direction Advisor (Fallback mode)** → **App-prototype rules** → **Workflow checkpoints** → **References router**. The router at the bottom maps task types to the `references/` files that should be loaded next. When extending the skill, prefer adding a new `references/X.md` and a router row over inflating `SKILL.md`. The `assets/` JSX components are designed to be inlined into a single HTML file because user-generated outputs run from `file://` (no HTTP server, no module imports). The `scripts/` toolchain converts HTML outputs into deliverables (PDF / PPTX / MP4 / GIF / video-with-audio) and is invoked by the agent when the relevant deliverable is requested.

## Scripts (invoked by the skill at runtime, not by a build system)

There is no `package.json`, no test runner, no linter. Each script is standalone and self-documents its usage in its own header. Common ones:

- `scripts/render-video.js` — HTML animation → MP4 via Playwright `recordVideo` + ffmpeg. Requires global Playwright (`npm install -g playwright`) and `ffmpeg` on PATH. Invoke with `NODE_PATH=$(npm root -g) node scripts/render-video.js <html-file>`. Animation HTML must set `window.__ready = true` after first paint (Stage in `assets/animations.jsx` does this); without it, output may have black-frame padding.
- `scripts/convert-formats.sh <input.mp4> [gif_width] [--minterpolate]` — derives `<name>-60fps.mp4` and palette-optimized `<name>.gif` next to the input.
- `scripts/add-music.sh <input.mp4> [--mood=<name>] [--music=<path>]` — mixes BGM from `assets/bgm-*.mp3` (`tech` / `ad` / `educational` / `educational-alt` / `tutorial` / `tutorial-alt`).
- `scripts/export_deck_pdf.mjs` / `scripts/export_deck_stage_pdf.mjs` — multi-file deck → PDF (vector text) and single-file `deck_stage.js` web-component → PDF respectively. Use the `_stage_pdf` variant only for `deck_stage.js`-based decks (handles shadow-DOM slot quirks).
- `scripts/export_deck_pptx.mjs` (calls `scripts/html2pptx.js`) — multi-file deck → editable PPTX. **The HTML must satisfy the 4 hard constraints in `references/editable-pptx.md`**; visual-freedom HTML (gradients, complex SVG, animations) cannot pass — route those through PDF instead.
- `scripts/verify.py` — Playwright wrapper for human-driven QA: screenshots, multi-viewport, per-slide capture, console-error scraping. Requires `pip install playwright && playwright install chromium`.

## Conventions when editing the skill

- **Don't paraphrase rules across files.** If a rule lives in `SKILL.md`, link to it from references; don't restate it. The skill leans on a single source of truth per rule.
- **Add to references, not SKILL.md.** `SKILL.md` is loaded eagerly; `references/*.md` is loaded on-demand. New content defaults to the latter, with a one-line entry added to the router table at the end of `SKILL.md`.
- **Skill paths are relative to the skill root** (`references/xxx.md`, `assets/xxx.jsx`, `scripts/xxx.sh`). Never hardcode absolute paths — the skill is installed into different locations per agent.
- **Bilingual posture.** Authoritative content lives in `SKILL.md` (Chinese-leaning bilingual) and `README.md`. `README.en.md` is the English mirror — keep them in sync when editing high-level claims, but day-to-day skill edits don't require translating both.
- **Demos are showcase material, not tests.** Editing a demo doesn't validate the skill end-to-end; the closest thing to a smoke test is running `scripts/verify.py` against a generated HTML or running the export scripts against a demo deck.
- **Personal data lives outside the repo.** `assets/personal-asset-index.json` is gitignored (only `.example.json` is committed). Real personal asset indexes go to the agent's private memory path (`~/.claude/memory/` for Claude Code) — never inside this skill directory.

## Non-negotiables that exist in SKILL.md and should survive any refactor

If you find yourself reorganizing `SKILL.md`, preserve these — they are the load-bearing rules:

- **Core Principle #0** — fact-verify (`WebSearch`) before any factual claim about a specific product/version/spec; runs *before* clarifying questions.
- **§1.a Core Asset Protocol** — 5 steps: ask → search by asset type → download (logo / product image / UI screenshot, three fallback paths each) → verify+extract → write `brand-spec.md`. The "5-10-2-8" quality gate (5 search rounds, 10 candidates, 2 picks, ≥8/10 each) is bedrock for non-logo assets; logos are exempt (have one or stop).
- **Anti-AI-slop list** — the regression target. Edits should not silently re-introduce a banned pattern.
- **Junior Designer checkpoints (🛑)** — the five explicit stop-and-confirm points in the workflow (1: questions sent, 2: asset self-check, 3: design system stated aloud, 4: junior pass shown early, 5: browser inspected before delivery).
- **App-prototype defaults** — single-file inline React (no external `<script src=...>`); `assets/ios_frame.jsx` is mandatory for iPhone mockups (no hand-rolled Dynamic Island / status bar).
- **Animation runtime contract** — when hand-rolling Stage/Sprite, set `window.__ready = true` on first paint and force `loop=false` when `window.__recording === true`. `scripts/render-video.js` depends on these.
- **Audio-default for animations** — animation deliverables default to MP4 with both BGM and SFX (dual-track). Silent video is a half-product. Verify with `ffprobe -select_streams a` before delivery unless the user opted out.

## License gotcha

`LICENSE` is "Personal Use Only." Commercial/enterprise integration requires explicit authorization from the author (花生 / @AlchainHust). Don't generate marketing copy that implies otherwise.
