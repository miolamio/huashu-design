# Video Export: Exporting HTML Animations to MP4/GIF

After an animation HTML is complete, users often ask, "Can this be exported as a video?" This guide provides the full workflow.

## When to Export

**Export timing**:
- The animation runs end to end and has been visually verified (Playwright screenshots confirm the state at key timestamps).
- The user has watched it in the browser at least once and approved the effect.
- **Do not** export while animation bugs are still unresolved. Once exported to video, changes become more expensive.

**Possible user trigger phrases**:
- "Can you export it as a video?"
- "Convert it to MP4."
- "Make it a GIF."
- "60fps."

## Output Specs

By default, provide three formats at once so the user can choose:

| Format | Spec | Best For | Typical Size (30s) |
|---|---|---|---|
| MP4 25fps | 1920×1080 · H.264 · CRF 18 | WeChat article embeds, video accounts, YouTube | 1-2 MB |
| MP4 60fps | 1920×1080 · minterpolate frame interpolation · H.264 · CRF 18 | High-frame-rate showcases, Bilibili, portfolios | 1.5-3 MB |
| GIF | 960×540 · 15fps · palette optimized | Twitter/X, README, Slack previews | 2-4 MB |

## Toolchain

Two scripts live in `scripts/`:

### 1. `render-video.js` -- HTML -> MP4

Records a base 25fps MP4. Requires global Playwright.

```bash
NODE_PATH=$(npm root -g) node /path/to/claude-design/scripts/render-video.js <html-file>
```

Optional parameters:
- `--duration=30` animation duration in seconds
- `--width=1920 --height=1080` resolution
- `--trim=2.2` seconds trimmed from the start of the video (removes reload + font loading time)
- `--fontwait=1.5` font loading wait time in seconds; increase it when many fonts are used

Output: a same-named `.mp4` in the same directory as the HTML.

### 2. `add-music.sh` -- MP4 + BGM -> MP4

Mixes background music into a silent MP4. It can pick from the built-in BGM library by scene mood, or use custom audio. It automatically matches duration and adds fade-in / fade-out.

```bash
bash add-music.sh <input.mp4> [--mood=<name>] [--music=<path>] [--out=<path>]
```

**Built-in BGM library** (in `assets/bgm-<mood>.mp3`):

| `--mood=` | Style | Suitable For |
|-----------|-------|--------------|
| `tech` (default) | Apple Silicon / Apple keynote, minimal synth + piano | Product launches, AI tools, skill promos |
| `ad` | Upbeat modern electronic, with build + drop | Social media ads, product teasers, promos |
| `educational` | Warm and bright, light guitar / electric piano, inviting | Science communication, tutorial intros, course teasers |
| `educational-alt` | Alternate in the same style; try another track | Same as above |
| `tutorial` | Lo-fi ambient, almost unobtrusive | Software demos, programming tutorials, long demos |
| `tutorial-alt` | Alternate in the same style | Same as above |

**Behavior**:
- Music is trimmed to the video duration.
- 0.3s fade-in + 1s fade-out to avoid hard cuts.
- The video stream uses `-c:v copy` with no re-encode; audio is AAC 192k.
- `--music=<path>` takes precedence over `--mood`, so any external audio can be specified directly.
- An invalid mood lists all available options instead of failing silently.

**Typical pipeline** (three-format animation export + music):
```bash
node render-video.js animation.html                        # Screen record
bash convert-formats.sh animation.mp4                      # Derive 60fps + GIF
bash add-music.sh animation-60fps.mp4                      # Add default tech BGM
# Or for different scenes:
bash add-music.sh tutorial-demo.mp4 --mood=tutorial
bash add-music.sh product-promo.mp4 --mood=ad --out=promo-final.mp4
```

### 3. `convert-formats.sh` -- MP4 -> 60fps MP4 + GIF

Generates a 60fps version and a GIF from an existing MP4.

```bash
bash /path/to/claude-design/scripts/convert-formats.sh <input.mp4> [gif_width] [--minterpolate]
```

Outputs (in the same directory as the input):
- `<name>-60fps.mp4` -- uses `fps=60` frame duplication by default for broad compatibility; add `--minterpolate` to enable high-quality frame interpolation
- `<name>.gif` -- palette-optimized GIF (default width 960, configurable)

**60fps mode selection**:

| Mode | Command | Compatibility | Use Case |
|---|---|---|---|
| Frame duplication (default) | `convert-formats.sh in.mp4` | Works in QuickTime / Safari / Chrome / VLC | General delivery, upload platforms, social media |
| minterpolate frame interpolation | `convert-formats.sh in.mp4 --minterpolate` | macOS QuickTime / Safari may refuse playback | Showcase contexts that require true interpolation, such as Bilibili. **Must be tested locally** in the target player before delivery |

Why did the default change to frame duplication? The H.264 elementary stream output by minterpolate has a known compatibility bug. Earlier defaults using minterpolate repeatedly hit the issue where "macOS QuickTime cannot open the file." See `animation-pitfalls.md` §14.

`gif_width` parameter:
- 960 (default) -- general social-platform use
- 1280 -- clearer but larger file
- 600 -- loads faster on Twitter/X

## Full Workflow (Standard Recommendation)

After the user asks to "export video":

```bash
cd <project-directory>

# Assume $SKILL points to this skill's root directory (replace it with the actual install location)

# 1. Record the base 25fps MP4
NODE_PATH=$(npm root -g) node "$SKILL/scripts/render-video.js" my-animation.html

# 2. Derive the 60fps MP4 and GIF
bash "$SKILL/scripts/convert-formats.sh" my-animation.mp4

# Output list:
# my-animation.mp4         (25fps · 1-2 MB)
# my-animation-60fps.mp4   (60fps · 1.5-3 MB)
# my-animation.gif         (15fps · 2-4 MB)
```

## Technical Details (For Troubleshooting)

### Playwright recordVideo Pitfalls

- The frame rate is fixed at 25fps; it cannot directly record 60fps (Chromium headless compositor limit).
- Recording starts when the context is created, so loading time at the beginning must be removed with `trim`.
- The default format is WebM, so ffmpeg must convert it to H.264 MP4 for universal playback.

`render-video.js` already handles the above.

### ffmpeg minterpolate Parameters

Current configuration: `minterpolate=fps=60:mi_mode=mci:mc_mode=aobmc:me_mode=bidir:vsbmc=1`

- `mi_mode=mci` -- motion compensation interpolation
- `mc_mode=aobmc` -- adaptive overlapped block motion compensation
- `me_mode=bidir` -- bidirectional motion estimation
- `vsbmc=1` -- variable-size block motion compensation

Works well for CSS **transform animations** (translate / scale / rotate).
For **pure fades**, it can produce slight ghosting. If the user dislikes that, fall back to simple frame duplication:

```bash
ffmpeg -i input.mp4 -r 60 -c:v libx264 ... output.mp4
```

### Why GIF Palette Encoding Needs Two Passes

GIF supports only 256 colors. A one-pass GIF compresses the entire animation into one generic 256-color palette, which muddies subtle palettes such as beige backgrounds with orange accents.

Two passes:
1. `palettegen=stats_mode=diff` -- scan the full video first and generate an **optimal palette for this animation**
2. `paletteuse=dither=bayer:bayer_scale=5:diff_mode=rectangle` -- encode with that palette; rectangle diff updates only changed areas, greatly reducing file size

For fade transitions, `dither=bayer` is smoother than `none`, but produces a slightly larger file.

## Pre-flight Check (Before Export)

30-second self-check before export:

- [ ] HTML has run through once in the browser with no console errors.
- [ ] Animation frame 0 is the complete initial state (not blank or loading).
- [ ] The final frame is a stable ending state (not halfway through).
- [ ] Fonts / images / emoji all render correctly (see `animation-pitfalls.md`).
- [ ] The duration parameter matches the actual animation duration in the HTML.
- [ ] The HTML Stage detects `window.__recording` and forces `loop=false` (required for handwritten Stage; built into `assets/animations.jsx`).
- [ ] The ending Sprite uses `fadeOut={0}` (the final video frame should not fade out).
- [ ] Includes a "Created by Huashu-Design" watermark (required only for animation scenarios; add the prefix "Unofficial production · " for third-party brand work. See SKILL.md § "Skill promotion watermark").

## Delivery Notes

Standard message format after export:

```
**Complete Delivery**

| File | Format | Spec | Size |
|---|---|---|---|
| foo.mp4 | MP4 | 1920×1080 · 25fps · H.264 | X MB |
| foo-60fps.mp4 | MP4 | 1920×1080 · 60fps (motion-interpolated) · H.264 | X MB |
| foo.gif | GIF | 960×540 · 15fps · palette optimized | X MB |

**Notes**
- 60fps uses minterpolate motion-estimation frame interpolation, which works well for transform animations.
- GIF is palette optimized; a 30s animation can usually be compressed to around 3 MB.

Tell me if you want a different size or frame rate.
```

## Common Follow-up Requests

| User Says | Response |
|---|---|
| "It's too large." | MP4: raise CRF to 23-28; GIF: reduce resolution to 600 or fps to 10 |
| "The GIF is too blurry." | Increase `gif_width` to 1280, or recommend MP4 instead (WeChat Moments also supports video) |
| "I need vertical 9:16." | Change the HTML source to `--width=1080 --height=1920` and record again |
| "Add a watermark." | Use ffmpeg `-vf "drawtext=..."` or `overlay=` a PNG |
| "I need a transparent background." | MP4 does not support alpha; use WebM VP9 + alpha or APNG |
| "I need lossless." | Change CRF to 0 + preset veryslow (file size will be 10× larger) |
