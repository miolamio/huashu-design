# Verification: Output Verification Workflow

Some design-agent native environments (such as Claude.ai Artifacts) have built-in `fork_verifier_agent` support that launches a subagent to inspect iframe screenshots. Most agent environments (Claude Code / Codex / Cursor / Trae / etc.) do not have this built-in capability. You can manually cover the same verification scenarios with Playwright.

## Verification Checklist

After every HTML output, go through this checklist:

### 1. Browser Rendering Check (Required)

The baseline: **can the HTML open at all?** On macOS:

```bash
open -a "Google Chrome" "/path/to/your/design.html"
```

Or take a Playwright screenshot (next section).

### 2. Console Error Check

The most common issue in HTML files is a JS error causing a blank screen. Run it once with Playwright:

```bash
python ~/.claude/skills/claude-design/scripts/verify.py path/to/design.html
```

This script will:
1. Open the HTML with headless Chromium.
2. Save a screenshot to the project directory.
3. Capture console errors.
4. Report status.

See `scripts/verify.py` for details.

### 3. Multi-Viewport Check

For responsive design, capture multiple viewports:

```bash
python verify.py design.html --viewports 1920x1080,1440x900,768x1024,375x667
```

### 4. Interaction Check

Tweaks, animations, and button toggles are not visible in default static screenshots. **Recommend that the user open it in the browser and click through it themselves**, or record the interaction with Playwright:

```python
page.video.record('interaction.mp4')
```

### 5. Slide-by-Slide Check

For deck-style HTML, screenshot each slide:

```bash
python verify.py deck.html --slides 10  # screenshot the first 10 slides
```

This generates `deck-slide-01.png`, `deck-slide-02.png`, and so on for quick review.

## Playwright Setup

First-time use requires:

```bash
# If not installed yet
npm install -g playwright
npx playwright install chromium

# Or the Python version
pip install playwright
playwright install chromium
```

If the user already has Playwright installed globally, use it directly.

## Screenshot Best Practices

### Capture the Full Page

```python
page.screenshot(path='full.png', full_page=True)
```

### Capture the Viewport

```python
page.screenshot(path='viewport.png')  # captures only the visible area by default
```

### Capture a Specific Element

```python
element = page.query_selector('.hero-section')
element.screenshot(path='hero.png')
```

### High-Resolution Screenshot

```python
page = browser.new_page(device_scale_factor=2)  # retina
```

### Wait for Animation to Finish Before Capture

```python
page.wait_for_timeout(2000)  # wait 2 seconds for animation to settle
page.screenshot(...)
```

## Sending Screenshots to the User

### Open Local Screenshots Directly

```bash
open screenshot.png
```

The user will view it in their own Preview/Figma/VSCode/browser.

### Upload to an Image Host and Share a Link

If a remote collaborator needs to see it (for example Slack/Feishu/WeChat), ask the user to upload it with their own image hosting tool or MCP:

```bash
python ~/Documents/writing/tools/upload_image.py screenshot.png
```

It returns a permanent ImgBB link that can be pasted anywhere.

## When Verification Fails

### Blank Page

There is almost certainly a console error. Check first:

1. Whether the React + Babel script tag integrity hash is correct (see `react-setup.md`).
2. Whether there is a naming conflict such as `const styles = {...}`.
3. Whether cross-file components have been exported to `window`.
4. JSX syntax errors (`babel.min.js` may not report them; switch to the unminified `babel.js`).

### Choppy Animation

- Record a run in the Chrome DevTools Performance tab.
- Look for layout thrashing (frequent reflow).
- Prefer `transform` and `opacity` for animation (GPU acceleration).

### Wrong Font

- Check whether the `@font-face` URL is accessible.
- Check fallback fonts.
- Chinese fonts load slowly: show fallback first, then switch after loading.

### Broken Layout

- Check whether `box-sizing: border-box` is applied globally.
- Check the `*  margin: 0; padding: 0` reset.
- Turn on gridlines in Chrome DevTools to inspect the actual layout.

## Verification = A Designer's Second Pair of Eyes

**Always review it yourself**. AI-written code often has issues like:

- Looks correct, but interactions are buggy.
- Static screenshot is fine, but layout breaks on scroll.
- Looks good on wide screens, but collapses on narrow screens.
- Dark mode was not tested.
- Some components do not respond after Tweaks are switched.

**The final 1 minute of verification can save 1 hour of rework**.

## Common Verification Script Commands

```bash
# Basic: open + screenshot + capture errors
python verify.py design.html

# Multiple viewports
python verify.py design.html --viewports 1920x1080,375x667

# Multiple slides
python verify.py deck.html --slides 10

# Output to a specific directory
python verify.py design.html --output ./screenshots/

# headless=false, open a real browser for you to inspect
python verify.py design.html --show
```
