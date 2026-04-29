# Repository Guidelines

## Project Structure & Module Organization

This repository is a markdown-based design skill, not a runtime application. `SKILL.md` is the main entry point loaded by agents. Put deeper, task-specific guidance in `references/*.md` and add router links from `SKILL.md` instead of expanding the main file. `assets/` contains inline-ready JSX/JS components, audio, SFX, and showcase HTML. `scripts/` holds standalone export and verification utilities. `demos/` contains capability demos (`c1`-`c6`) and workflow demos (`w1`-`w3`), with English variants ending in `-en.html`. `test-prompts.json` stores sample trigger prompts.

## Build, Test, and Development Commands

There is no package-level build, test runner, or linter. Use the standalone scripts as needed:

- `python scripts/verify.py demos/c1-ios-prototype.html --viewports 1440x900,375x667` checks HTML with Playwright and saves screenshots.
- `NODE_PATH=$(npm root -g) node scripts/render-video.js path/to/animation.html` records an animation to MP4; requires global Playwright and `ffmpeg`.
- `scripts/convert-formats.sh path/to/video.mp4 720 --minterpolate` creates GIF and 60fps derivatives.
- `scripts/add-music.sh path/to/video.mp4 --mood=tech` mixes bundled background music.
- `node scripts/export_deck_pptx.mjs path/to/deck.html` exports compatible HTML decks to editable PPTX.

## Coding Style & Naming Conventions

Keep skill paths relative to the repository root, for example `references/editable-pptx.md` or `assets/animations.jsx`; never hardcode local absolute paths. Prefer concise Markdown with clear headings and single-source rules. Asset starter components must remain usable when inlined into single-file HTML under `file://`, so avoid module-only assumptions. Demo naming should preserve existing prefixes and language suffixes, such as `c3-motion-design.html` and `c3-motion-design-en.html`.

## Testing Guidelines

Treat demos as showcase material, not exhaustive tests. For HTML, prototype, or deck changes, run `scripts/verify.py` against representative files and include desktop plus mobile viewports when layout is affected. For export scripts, test the exact conversion path changed, such as PDF, PPTX, MP4, GIF, or BGM mixing. Generated screenshots normally belong in temporary or ignored output directories, not committed as routine test artifacts.

## Commit & Pull Request Guidelines

Recent history uses short conventional-style prefixes such as `docs:`, `chore:`, `sfx:`, and `add:`. Keep commits focused and describe the changed surface, for example `docs: add animation export guidance`. Pull requests should explain intent, list touched areas (`SKILL.md`, `references/`, `assets/`, `scripts/`, or `demos/`), and include screenshots or exported samples for visual changes. Call out required tools such as Playwright, `ffmpeg`, or PowerPoint-compatible export validation.

## Security & Configuration Tips

`LICENSE` is Personal Use Only; do not imply commercial usage rights without explicit authorization. Keep personal asset indexes out of the repo: `assets/personal-asset-index.json` is ignored and only `assets/personal-asset-index.example.json` should be committed.
