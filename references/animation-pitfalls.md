# Animation Pitfalls: Bugs and Rules from HTML Animation Work

The most common bugs when building animations, and how to avoid them. Every rule comes from a real failure case.

Read this before writing animation and you can save an iteration.

## Contents

- [1. Layered Layout — `position: relative` is a default obligation](#1-layered-layout----position-relative-is-a-default-obligation)
- [2. Character Trap — do not depend on rare Unicode](#2-character-trap----do-not-depend-on-rare-unicode)
- [3. Data-driven Grid/Flex Templates](#3-data-driven-gridflex-templates)
- [4. Transition Gaps — scene switches must be continuous](#4-transition-gaps----scene-switches-must-be-continuous)
- [5. Pure Render Principle — animation state should be seekable](#5-pure-render-principle----animation-state-should-be-seekable)
- [6. Measuring before fonts load = wrong measurements](#6-measuring-before-fonts-load--wrong-measurements)
- [7. Recording preparation — leave handles for video export](#7-recording-preparation----leave-handles-for-video-export)
- [8. Batch export — tmp directories must include PID](#8-batch-export----tmp-directories-must-include-pid-to-avoid-parallel-conflicts)
- [9. Progress bar / replay button appears in recording](#9-progress-bar--replay-button-appears-in-recording----chrome-elements-pollute-the-video)
- [10. Animation repeats at the start of recording — warmup frames leaked](#10-animation-repeats-at-the-beginning-of-recording----warmup-frames-leaked)
- [11. Do not draw "fake chrome" inside the frame](#11-do-not-draw-fake-chrome-inside-the-frame----decorative-player-ui-collides-with-real-chrome)
- [12. Leading blank recording + offset start — `__ready` × tick × lastTick triple trap](#12-leading-blank-recording--offset-recording-start----the-__ready--tick--lasttick-triple-trap)
- [13. Disable loop during recording — `window.__recording` signal](#13-disable-loop-during-recording----the-window__recording-signal)
- [14. Default 60fps video uses frame duplication — minterpolate compatibility](#14-default-60fps-video-uses-frame-duplication----minterpolate-has-poor-compatibility)
- [15. `file://` + external `.jsx` CORS trap — inline the engine for single-file delivery](#15-file--external-jsx-cors-trap----inline-the-engine-for-single-file-delivery)
- [16. Cross-scene inverted context — do not hardcode colors for in-frame elements](#16-cross-scene-inverted-context----do-not-hardcode-colors-for-in-frame-elements)
- [Quick Self-check Checklist (5 seconds before starting)](#quick-self-check-checklist-5-seconds-before-starting)

---

## 1. Layered Layout -- `position: relative` Is a Default Obligation

**Pitfall**: A `sentence-wrap` element contained 3 `bracket-layer` elements (`position: absolute`). `sentence-wrap` did not have `position: relative`, so the absolute brackets used `.canvas` as their coordinate system and drifted 200px below the bottom of the screen.

**Rules**:
- Any container with `position: absolute` children **must** explicitly set `position: relative`.
- Even if you do not visually need an "offset," write `position: relative` as the coordinate-system anchor.
- If you are writing `.parent { ... }` and one of its children has `.child { position: absolute }`, instinctively add `relative` to the parent.

**Quick check**: every time you see `position: absolute`, walk up the ancestors and ensure the nearest positioned ancestor is the coordinate system you *intended*.

## 2. Character Trap -- Do Not Depend on Rare Unicode

**Pitfall**: We wanted to use `␣` (U+2423 OPEN BOX) to visualize a "space token." Noto Serif SC / Cormorant Garamond did not contain that glyph, so it rendered as blank / tofu and viewers could not see it at all.

**Rules**:
- **Every character that appears in an animation must exist in the chosen font.**
- Common rare-character blacklist: `␣ ␀ ␐ ␋ ␨ ↩ ⏎ ⌘ ⌥ ⌃ ⇧ ␦ ␖ ␛`
- To represent metacharacters such as "space / return / tab," use a **semantic box built with CSS**:
  ```html
  <span class="space-key">Space</span>
  ```
  ```css
  .space-key {
    display: inline-flex;
    padding: 4px 14px;
    border: 1.5px solid var(--accent);
    border-radius: 4px;
    font-family: monospace;
    font-size: 0.3em;
    letter-spacing: 0.2em;
    text-transform: uppercase;
  }
  ```
- Emoji also need verification. Some emoji fall back to gray boxes outside Noto Emoji, so preferably use an `emoji` font-family or SVG.

## 3. Data-driven Grid/Flex Templates

**Pitfall**: The code had `const N = 6` tokens, but CSS hardcoded `grid-template-columns: 80px repeat(5, 1fr)`. The 6th token had no column, so the entire matrix misaligned.

**Rules**:
- When the count comes from a JS array (`TOKENS.length`), the CSS template should also be data-driven.
- Option A: inject a CSS variable from JS
  ```js
  el.style.setProperty('--cols', N);
  ```
  ```css
  .grid { grid-template-columns: 80px repeat(var(--cols), 1fr); }
  ```
- Option B: use `grid-auto-flow: column` and let the browser expand automatically.
- **Disallow the combination of "fixed number + JS constant."** When N changes, CSS will not update with it.

## 4. Transition Gaps -- Scene Switches Must Be Continuous

**Pitfall**: Between zoom1 (13-19s) and zoom2 (19.2-23s), the main sentence was already hidden. zoom1 fade out (0.6s) + zoom2 fade in (0.6s) + stagger delay (0.2s+) produced about 1 second of pure blank screen. Viewers thought the animation froze.

**Rules**:
- During continuous scene switches, fade out and fade in should **overlap as a cross-fade**, not wait for the previous scene to disappear completely before starting the next.
  ```js
  // Bad:
  if (t >= 19) hideZoom('zoom1');      // 19.0s out
  if (t >= 19.4) showZoom('zoom2');    // 19.4s in -> 0.4s blank gap

  // Good:
  if (t >= 18.6) hideZoom('zoom1');    // start fade out 0.4s early
  if (t >= 18.6) showZoom('zoom2');    // fade in at the same time (cross-fade)
  ```
- Or use an "anchor element" (such as the main sentence) as a visual connection between scenes; it briefly reappears during zoom switches.
- Calculate CSS transition durations clearly so the next event is not triggered before the current transition finishes.

## 5. Pure Render Principle -- Animation State Should Be Seekable

**Pitfall**: Animation state was triggered through chained `setTimeout` + `fireOnce(key, fn)`. Normal playback worked, but during frame-by-frame recording or seeking to an arbitrary time, previous `setTimeout`s had already run and could not "go back in time."

**Rules**:
- Ideally, the `render(t)` function is a **pure function**: given `t`, it outputs one unique DOM state.
- If side effects are required (such as class toggles), pair a `fired` set with an explicit reset:
  ```js
  const fired = new Set();
  function fireOnce(key, fn) { if (!fired.has(key)) { fired.add(key); fn(); } }
  function reset() { fired.clear(); /* clear all .show classes */ }
  ```
- Expose `window.__seek(t)` for Playwright / debugging:
  ```js
  window.__seek = (t) => { reset(); render(t); };
  ```
- Animation-related `setTimeout`s should not span more than 1 second, or seeking backward will scramble state.

## 6. Measuring Before Fonts Load = Wrong Measurements

**Pitfall**: The page called `charRect(idx)` on `DOMContentLoaded` to measure bracket positions, but fonts had not loaded yet. Every character width came from the fallback font, so all positions were wrong. When the font loaded about 500ms later, the bracket `left: Xpx` values were still the old ones and remained permanently offset.

**Rules**:
- Any layout code that depends on DOM measurement (`getBoundingClientRect`, `offsetWidth`) **must** be wrapped in `document.fonts.ready.then()`.
  ```js
  document.fonts.ready.then(() => {
    requestAnimationFrame(() => {
      buildBrackets(...);  // Font is ready now, so measurements are accurate
      tick();              // Start animation
    });
  });
  ```
- The extra `requestAnimationFrame` gives the browser one frame to commit layout.
- If using the Google Fonts CDN, add `<link rel="preconnect">` to speed up first load.

## 7. Recording Preparation -- Leave Handles for Video Export

**Pitfall**: Playwright `recordVideo` defaults to 25fps and starts recording as soon as the context is created. The first 2 seconds of page load and font loading were recorded. The delivered video had 2 seconds of blank / white flash at the beginning.

**Rules**:
- Provide the `render-video.js` tool to handle: warmup navigate -> reload to restart animation -> wait duration -> ffmpeg trim head + convert to H.264 MP4.
- Animation **frame 0** should be a complete initial state with the final layout already in place (not blank or loading).
- Want 60fps? Use ffmpeg `minterpolate` as post-processing; do not rely on browser source frame rate.
- Want GIF? Use a two-stage palette (`palettegen` + `paletteuse`); a 30s 1080p animation can compress to 3MB.

See `video-export.md` for complete script usage.

## 8. Batch Export -- tmp Directories Must Include PID to Avoid Parallel Conflicts

**Pitfall**: We used `render-video.js` to record 3 HTML files in parallel with 3 processes. Because `TMP_DIR` was named only with `Date.now()`, the 3 processes that started in the same millisecond shared the same tmp directory. The first process to finish cleaned tmp, and the other two hit `ENOENT` when reading the directory, crashing all exports.

**Rules**:
- Any temporary directory that may be shared by multiple processes must include a **PID or random suffix**:
  ```js
  const TMP_DIR = path.join(DIR, '.video-tmp-' + Date.now() + '-' + process.pid);
  ```
- If you truly want multiple files in parallel, use shell `&` + `wait` instead of forking inside one node script.
- When recording multiple HTML files in a batch, the conservative approach is to run **serially** (2 or fewer can be parallel; 3 or more should queue).

## 9. Progress Bar / Replay Button Appears in Recording -- Chrome Elements Pollute the Video

**Pitfall**: The animation HTML included a `.progress` bar, `.replay` button, and `.counter` timestamp for human debugging. When recorded as a delivered MP4, these elements appeared at the bottom of the video, as if developer tools had been captured.

**Rules**:
- Manage human-facing "chrome elements" (progress bar / replay button / footer / masthead / counter / phase labels) separately from the video content itself.
- **Convention class name** `.no-record`: any element with this class is automatically hidden by the recording script.
- The script side (`render-video.js`) injects CSS by default to hide common chrome class names:
  ```
  .progress .counter .phases .replay .masthead .footer .no-record [data-role="chrome"]
  ```
- Use Playwright `addInitScript` for injection (it takes effect before every navigate, including reloads).
- Add the `--keep-chrome` flag when you want to see the original HTML with chrome.

## 10. Animation Repeats at the Beginning of Recording -- Warmup Frames Leaked

**Pitfall**: The old `render-video.js` flow was `goto -> wait fonts 1.5s -> reload -> wait duration`. Recording starts when the context is created, so the animation had already played during warmup, then restarted from 0 after reload. The video began with "middle of animation + switch + animation from 0," creating a strong sense of repetition.

**Rules**:
- **Warmup and Record must use separate contexts**:
  - Warmup context (without `recordVideo`): only loads the URL, waits for fonts, then closes
  - Record context (with `recordVideo`): starts from a fresh state and records animation from t=0
- ffmpeg `-ss trim` can only cut a small amount of Playwright startup latency (~0.3s). It **cannot** hide warmup frames; the source must be clean.
- Closing the recording context writes the WebM file to disk; this is a Playwright constraint.
- Related code pattern:
  ```js
  // Phase 1: warmup (throwaway)
  const warmupCtx = await browser.newContext({ viewport });
  const warmupPage = await warmupCtx.newPage();
  await warmupPage.goto(url, { waitUntil: 'networkidle' });
  await warmupPage.waitForTimeout(1200);
  await warmupCtx.close();

  // Phase 2: record (fresh)
  const recordCtx = await browser.newContext({ viewport, recordVideo });
  const page = await recordCtx.newPage();
  await page.goto(url, { waitUntil: 'networkidle' });
  await page.waitForTimeout(DURATION * 1000);
  await page.close();
  await recordCtx.close();
  ```

## 11. Do Not Draw "Fake Chrome" Inside the Frame -- Decorative Player UI Collides with Real Chrome

**Pitfall**: The animation used the `Stage` component, which already includes a scrubber + timecode + pause button (all `.no-record` chrome, automatically hidden on export). I also drew a decorative "magazine page-number style progress bar" at the bottom: "`00:60 ──── CLAUDE-DESIGN / ANATOMY`". It felt good in the moment. **Result**: users saw two progress bars: the Stage controller and my decorative one. Visually they collided completely and were judged as a bug. "Why is there another progress bar inside the video?"

**Rules**:

- Stage already provides: scrubber + timecode + pause / replay buttons. **Do not draw** progress indicators, current timecodes, copyright signature bars, or chapter counters inside the frame. They either collide with chrome or are filler slop (violating the "earn its place" principle).
- "Page-number feel," "magazine feel," and "bottom signature bar" are high-frequency filler that AI tends to add automatically. Be alert whenever they appear: do they really communicate irreplaceable information, or do they merely fill empty space?
- If you strongly believe a bottom bar must exist (for example, the animation topic is player UI), it must be **narratively necessary** and **visually distinct from the Stage scrubber** (different position, form, and tone).

**Element ownership test** (every element drawn into the canvas must answer this):

| What It Belongs To | Treatment |
|------------|-----------|
| Narrative content of a specific scene | OK, keep it |
| Global chrome (control / debugging) | Add `.no-record` class and hide it on export |
| **Neither any scene nor chrome** | **Delete it**. This is ownerless content and inevitably filler slop |

**Self-check (3 seconds before delivery)**: take one static screenshot and ask yourself:

- Is there anything in the frame that "looks like video player UI" (horizontal progress line, timecode, control-button shape)?
- If so, would deleting it damage the narrative? If not, delete it.
- Does the same kind of information (progress / time / signature) appear twice? Merge it into chrome in one place.

**Counterexamples**: drawing `00:42 ──── PROJECT NAME` at the bottom, drawing a "CH 03 / 06" chapter counter at the bottom-right, or drawing a version number such as "v0.3.1" at the frame edge. All are fake-chrome filler.

## 12. Leading Blank Recording + Offset Recording Start -- The `__ready` × tick × lastTick Triple Trap

**Pitfall (A · leading blank)**: A 60-second animation exported to MP4 had 2-3 seconds of blank page at the beginning. `ffmpeg --trim=0.3` could not remove it.

**Pitfall (B · start offset, real incident on 2026-04-20)**: A 24-second video export felt to the user like "the first frame only starts playing at 19 seconds." In reality, the animation started recording at t=5, recorded until t=24, looped back to t=0, then recorded 5 more seconds to the end. So the last 5 seconds of the video were the real beginning of the animation.

**Root cause** (shared by both pitfalls):

Playwright `recordVideo` starts writing WebM from the moment `newContext()` is created. Babel / React / font loading takes L seconds total (2-6s). The recording script waits for `window.__ready = true` as the anchor for "animation starts here," and it must pair strictly with animation `time = 0`. There are two common wrong patterns:

| Wrong Pattern | Symptom |
|------|---------|
| `__ready` is set in `useEffect` or synchronous setup (before the first tick frame) | The recording script thinks the animation has started, but the WebM is still recording a blank page -> **leading blank** |
| tick's `lastTick = performance.now()` is initialized at the **top level** | Font loading time L is included in the first-frame `dt`, so `time` instantly jumps to L -> recording lags by L seconds throughout -> **start offset** |

**✅ Correct complete starter tick template** (handwritten animation must use this skeleton):

```js
// ━━━━━━ state ━━━━━━
let time = 0;
let playing = false;   // ❗ Do not play by default; start after fonts are ready
let lastTick = null;   // ❗ sentinel: force dt to 0 on first tick frame (do not use performance.now())
const fired = new Set();

// ━━━━━━ tick ━━━━━━
function tick(now) {
  if (lastTick === null) {
    lastTick = now;
    window.__ready = true;   // ✅ pair: "recording start" and "animation t=0" are the same frame
    render(0);               // render once more to ensure DOM is ready (fonts are ready now)
    requestAnimationFrame(tick);
    return;
  }
  const dt = (now - lastTick) / 1000;   // dt only advances after the first frame
  lastTick = now;

  if (playing) {
    let t = time + dt;
    if (t >= DURATION) {
      t = window.__recording ? DURATION - 0.001 : 0;  // do not loop while recording; keep 0.001s to preserve final frame
      if (!window.__recording) fired.clear();
    }
    time = t;
    render(time);
  }
  requestAnimationFrame(tick);
}

// ━━━━━━ boot ━━━━━━
// Do not rAF immediately at top level; wait for fonts to load before starting
document.fonts.ready.then(() => {
  render(0);                 // draw the initial frame first (fonts are ready)
  playing = true;
  requestAnimationFrame(tick);  // first tick pairs __ready + t=0
});

// ━━━━━━ seek interface (for defensive correction by render-video) ━━━━━━
window.__seek = (t) => { fired.clear(); time = t; lastTick = null; render(t); };
```

**Why this template is correct**:

| Step | Why It Must Be This Way |
|------|--------------------------|
| `lastTick = null` + first-frame `return` | Prevents the L seconds between script load and first tick from being counted as animation time |
| `playing = false` by default | Even if `tick` runs during font loading, time does not advance, preventing render offset |
| `__ready` set on the first tick frame | The recording script starts timing at the exact frame corresponding to true animation t=0 |
| Start tick only inside `document.fonts.ready.then(...)` | Avoids font fallback width measurement and first-frame font jumps |
| `window.__seek` exists | Lets `render-video.js` actively correct the timeline as a second line of defense |

**Corresponding defense on the recording script side**:
1. Inject `window.__recording = true` via `addInitScript` (before page goto)
2. `waitForFunction(() => window.__ready === true)`, recording this moment as the ffmpeg trim offset
3. **Additionally**: after `__ready`, actively call `page.evaluate(() => window.__seek && window.__seek(0))` to force any possible HTML time drift back to zero. This is the second line of defense for HTML that does not strictly follow the starter template

**Verification method**: after exporting MP4
```bash
ffmpeg -i video.mp4 -ss 0 -vframes 1 frame-0.png
ffmpeg -i video.mp4 -ss $DURATION-0.1 -vframes 1 frame-end.png
```
The first frame must be the animation's t=0 initial state (not the middle, not black), and the final frame must be the animation's final state (not a moment from a second loop).

**Reference implementation**: the Stage component in `assets/animations.jsx` and `scripts/render-video.js` already implement this protocol. Handwritten HTML must use the starter tick template; every line prevents a specific bug.

## 13. Disable loop During Recording -- The `window.__recording` Signal

**Pitfall**: Animation Stage defaults to `loop=true` for convenient browser preview. `render-video.js` waits an extra 300ms buffer after recording `duration` seconds before stopping. Those 300ms let Stage enter the next loop. When ffmpeg clips with `-t DURATION`, the final 0.5-1s falls into the next loop, so the video suddenly returns to the first frame (Scene 1). Viewers think the video is broken.

**Root cause**: There is no "I am recording" handshake between the recording script and HTML. The HTML does not know it is being recorded, so it still loops as in an interactive browser session.

**Rules**:

1. **Recording script**: inject `window.__recording = true` in `addInitScript` (before page goto):
   ```js
   await recordCtx.addInitScript(() => { window.__recording = true; });
   ```

2. **Stage component**: detect this signal and force loop=false:
   ```js
   const effectiveLoop = (typeof window !== 'undefined' && window.__recording) ? false : loop;
   // ...
   if (next >= duration) return effectiveLoop ? 0 : duration - 0.001;
   //                                                       ↑ keep 0.001 to prevent Sprite end=duration from being hidden
   ```

3. **Ending Sprite fadeOut**: in recording scenes, set `fadeOut={0}`. Otherwise the end of the video fades to transparent / dark, while users expect a clear final frame, not a fade-out. For handwritten HTML, set all ending Sprites to `fadeOut={0}`.

**Reference implementation**: Stage in `assets/animations.jsx` and `scripts/render-video.js` both include this handshake. Handwritten Stage must implement `__recording` detection or recording will hit this pitfall.

**Verification**: after exporting MP4, run `ffmpeg -ss 19.8 -i video.mp4 -frames:v 1 end.png` and check that the final 0.2 seconds are still the expected last frame, with no sudden switch to another scene.

## 14. Default 60fps Video Uses Frame Duplication -- minterpolate Has Poor Compatibility

**Pitfall**: The 60fps MP4 generated by `convert-formats.sh` using `minterpolate=fps=60:mi_mode=mci...` could not open in some macOS QuickTime / Safari versions (black screen or refused playback). VLC / Chrome could open it.

**Root cause**: the H.264 elementary stream output by minterpolate contains some SEI / SPS fields that certain players parse incorrectly.

**Rules**:

- Default 60fps uses the simple `fps=60` filter (frame duplication) for broad compatibility (QuickTime / Safari / Chrome / VLC all open it).
- High-quality interpolation must be explicitly enabled with the `--minterpolate` flag, and the target player **must be tested locally** before delivery.
- The value of the 60fps label is **upload-platform algorithm recognition** (the 60fps label on Bilibili / YouTube can get priority streaming). For CSS animation, the perceptual smoothness improvement is slight.
- Add `-profile:v high -level 4.0` to improve general H.264 compatibility.

**`convert-formats.sh` now defaults to compatibility mode**. If you need high-quality frame interpolation, add the `--minterpolate` flag:
```bash
bash convert-formats.sh input.mp4 --minterpolate
```

## 15. `file://` + External `.jsx` CORS Trap -- Inline the Engine for Single-file Delivery

**Pitfall**: The animation HTML loaded the engine externally with `<script type="text/babel" src="animations.jsx"></script>`. Double-clicking locally (`file://` protocol) made Babel Standalone fetch `.jsx` via XHR, and Chrome reported `Cross origin requests are only supported for protocol schemes: http, https, chrome, chrome-extension...`. The whole page was black. It did not emit a `pageerror`, only a console error, so it was easy to misdiagnose as "animation did not trigger."

Starting an HTTP server does not always save you either. If the local machine has a global proxy, `localhost` may also go through the proxy and return 502 / connection failure.

**Rules**:

- **Single-file delivery (HTML that works by double-clicking)** -> `animations.jsx` must be **inlined** inside a `<script type="text/babel">...</script>` tag. Do not use `src="animations.jsx"`.
- **Multi-file project (demo via HTTP server)** -> external loading is fine, but delivery must clearly include the `python3 -m http.server 8000` command.
- Decision rule: are you delivering an "HTML file" or a "project directory with a server"? Use inline for the former.
- Stage component / animations.jsx is often 200+ lines. Pasting it into an HTML `<script>` block is completely acceptable; do not worry about size.

**Minimal verification**: double-click the generated HTML and open it **without** any server. It passes only if Stage displays the animation's first frame correctly.

## 16. Cross-scene Inverted Context -- Do Not Hardcode Colors for In-frame Elements

**Pitfall**: In a multi-scene animation, `ChapterLabel` / `SceneNumber` / `Watermark` and other elements that appear **across scenes** hardcoded `color: '#1A1A1A'` (dark text) inside the component. The first 4 scenes had light backgrounds and looked fine, but in the 5th black-background scene, "05" and the watermark disappeared. No error, no check triggered, key information became invisible.

**Rules**:

- **In-frame elements reused across multiple scenes** (chapter label / scene number / timecode / watermark / copyright bar) **must not hardcode color values**.
- Use one of these three approaches:
  1. **`currentColor` inheritance**: the element only writes `color: currentColor`; the parent scene container sets `color` to a computed value
  2. **invert prop**: the component accepts `<ChapterLabel invert />` to switch light / dark manually
  3. **Automatic calculation from background**: `color: contrast-color(var(--scene-bg))` (CSS 4 new API, or JS judgment)
- Before delivery, use Playwright to sample **representative frames from every scene** and visually inspect whether "cross-scene elements" are visible in all of them.

This pitfall is hidden because **there is no bug alarm**. Only human eyes or OCR will catch it.

## Quick Self-check Checklist (5 Seconds Before Starting)

- [ ] Does every parent of `position: absolute` children have `position: relative`?
- [ ] Do all special characters in the animation (`␣` `⌘` `emoji`) exist in the font?
- [ ] Does the Grid/Flex template count match the JS data length?
- [ ] Are scene switches cross-faded, with no >0.3s pure blank gap?
- [ ] Is DOM measurement code wrapped in `document.fonts.ready.then()`?
- [ ] Is `render(t)` pure, or is there a clear reset mechanism?
- [ ] Is frame 0 a complete initial state, not blank?
- [ ] Is there no decorative "fake chrome" inside the frame (progress bar / timecode / bottom signature bar colliding with Stage scrubber)?
- [ ] Does the first animation tick set `window.__ready = true` synchronously? (Built into animations.jsx; add it yourself in handwritten HTML.)
- [ ] Does Stage detect `window.__recording` and force loop=false? (Required for handwritten HTML.)
- [ ] Is the ending Sprite's `fadeOut` set to 0 (video ends on a clear frame)?
- [ ] Does 60fps MP4 default to frame duplication mode (compatibility), with `--minterpolate` only for high-quality interpolation?
- [ ] After export, did you sample frame 0 + final frame to verify they are the animation's initial / final states?
- [ ] For specific brands (Stripe / Anthropic / Lovart / ...): did you complete the "brand asset protocol" (SKILL.md §1.a five steps)? Did you write `brand-spec.md`?
- [ ] Single-file delivery HTML: is `animations.jsx` inlined instead of `src="..."`? (External `.jsx` causes CORS black screen under `file://`.)
- [ ] Cross-scene elements (chapter labels / watermarks / scene numbers) have no hardcoded colors and remain visible on every scene background?
