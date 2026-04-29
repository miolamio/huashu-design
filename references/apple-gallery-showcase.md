# Apple Gallery Showcase · Gallery Wall Animation Style

> Inspiration: the Claude Design homepage hero video + Apple product-page "work wall" display
> Field origin: huashu-design launch hero v5
> Suitable scenarios: **product-launch hero animations, skill capability demos, portfolio showcases** - any scene that needs to exhibit many high-quality outputs at once while guiding audience attention

---

## Trigger: When to Use This Style

**Suitable**:
- You have 10+ real outputs to show on one screen: PPT, app, website, infographic, and similar work
- The audience is professional: developers, designers, product managers; they are sensitive to "quality feel"
- The intended tone is restrained, exhibition-like, premium, and spatial
- You need focus and the whole view to coexist: inspect details without losing the overall field

**Not suitable**:
- Single-product focus; use the frontend-design product hero template instead
- Emotion-led or strongly story-driven animations; use a timeline narrative template
- Small screens / vertical video; the tilted perspective becomes muddy in small frames

---

## Core Visual Tokens

```css
:root {
  /* Light gallery palette */
  --bg:         #F5F5F7;   /* Main canvas background - Apple website gray */
  --bg-warm:    #FAF9F5;   /* Warm off-white variant */
  --ink:        #1D1D1F;   /* Primary text color */
  --ink-80:     #3A3A3D;
  --ink-60:     #545458;
  --muted:      #86868B;   /* Secondary text */
  --dim:        #C7C7CC;
  --hairline:   #E5E5EA;   /* 1px card border */
  --accent:     #D97757;   /* Terracotta orange - Claude brand */
  --accent-deep:#B85D3D;

  --serif-cn: "Noto Serif SC", "Songti SC", Georgia, serif;
  --serif-en: "Source Serif 4", "Tiempos Headline", Georgia, serif;
  --sans:     "Inter", -apple-system, "PingFang SC", system-ui;
  --mono:     "JetBrains Mono", "SF Mono", ui-monospace;
}
```

**Key Principles**:
1. **Never use a pure black background**. Black makes the work look like film, not like usable professional output.
2. **Terracotta orange is the only hue accent**; everything else is grayscale plus white.
3. **Three type stacks** (English serif + Chinese serif + sans + mono) create a "publication" feel rather than an "internet product" feel.

---

## Core Layout Patterns

### 1. Floating Card (The Basic Unit of the Whole Style)

```css
.gallery-card {
  background: #FFFFFF;
  border-radius: 14px;
  padding: 6px;                          /* Inner padding is the mat board */
  border: 1px solid var(--hairline);
  box-shadow:
    0 20px 60px -20px rgba(29, 29, 31, 0.12),   /* Primary shadow: soft and long */
    0 6px 18px -6px rgba(29, 29, 31, 0.06);     /* Secondary near light for floating feel */
  aspect-ratio: 16 / 9;                  /* Unified slide ratio */
  overflow: hidden;
}
.gallery-card img {
  width: 100%; height: 100%;
  object-fit: cover;
  border-radius: 9px;                    /* Slightly smaller than card radius for visual nesting */
}
```

**Counterexample**: do not use edge-to-edge tiles with no padding, border, or shadow. That expresses infographic density, not exhibition.

### 2. 3D Tilted Work Wall

```css
.gallery-viewport {
  position: absolute; inset: 0;
  overflow: hidden;
  perspective: 2400px;                   /* Deeper perspective, so the tilt is not exaggerated */
  perspective-origin: 50% 45%;
}
.gallery-canvas {
  width: 4320px;                         /* Canvas = 2.25× viewport */
  height: 2520px;                        /* Leave room for pan */
  transform-origin: center center;
  transform: perspective(2400px)
             rotateX(14deg)              /* Tilt backward */
             rotateY(-10deg)             /* Turn left */
             rotateZ(-2deg);             /* Slight skew to remove over-regularity */
  display: grid;
  grid-template-columns: repeat(8, 1fr);
  gap: 40px;
  padding: 60px;
}
```

**Parameter sweet spot**:
- rotateX: 10-15deg; more feels like a VIP event backdrop
- rotateY: ±8-12deg; creates left/right symmetry feel
- rotateZ: ±2-3deg; adds human irregularity
- perspective: 2000-2800px; below 2000 becomes fisheye, above 3000 approaches orthographic

### 3. 2×2 Corner Convergence (Selection Scene)

```css
.grid22 {
  display: grid;
  grid-template-columns: repeat(2, 800px);
  gap: 56px 64px;
  align-items: start;
}
```

Each card slides in from its corresponding corner (tl/tr/bl/br) toward the center and fades in. Corresponding `cornerEntry` vectors:

```js
const cornerEntry = {
  tl: { dx: -700, dy: -500 },
  tr: { dx:  700, dy: -500 },
  bl: { dx: -700, dy:  500 },
  br: { dx:  700, dy:  500 },
};
```

---

## Five Core Animation Patterns

### Pattern A · Corner Convergence (0.8-1.2s)

Four elements slide in from the viewport corners while scaling 0.85→1.0 with ease-out. Best as an opening that presents multiple directions of choice.

```js
const inP = easeOut(clampLerp(t, start, end));
card.style.transform = `translate3d(${(1-inP)*ce.dx}px, ${(1-inP)*ce.dy}px, 0) scale(${0.85 + 0.15*inP})`;
card.style.opacity = inP;
```

### Pattern B · Selected Zoom + Others Slide Out (0.8s)

The selected card scales 1.0→1.28; the others fade out, blur, and drift back toward the corners:

```js
// Selected
card.style.transform = `translate3d(${cellDx*outP}px, ${cellDy*outP}px, 0) scale(${1 + 0.28*easeOut(zoomP)})`;
// Unselected
card.style.opacity = 1 - outP;
card.style.filter = `blur(${outP * 1.5}px)`;
```

**Key**: unselected cards should blur, not just fade. Blur simulates depth of field and visually pushes the selected card forward.

### Pattern C · Ripple Expansion (1.7s)

From the center outward, each card fades in with a distance-based delay and shrinks from 1.25x to 0.94x, creating a "camera pulls back" feeling:

```js
const col = i % COLS, row = Math.floor(i / COLS);
const dc = col - (COLS-1)/2, dr = row - (ROWS-1)/2;
const dist = Math.sqrt(dc*dc + dr*dr);
const delay = (dist / maxDist) * 0.8;
const localT = Math.max(0, (t - rippleStart - delay) / 0.7);
card.style.opacity = easeOut(Math.min(1, localT));

// Overall scale simultaneously changes 1.25→0.94
const galleryScale = 1.25 - 0.31 * easeOut(rippleProgress);
```

### Pattern D · Sinusoidal Pan (Continuous Drift)

Combine sine waves with linear drift to avoid a marquee-like loop with an obvious start and end:

```js
const panX = Math.sin(panT * 0.12) * 220 - panT * 8;    // Drift left horizontally
const panY = Math.cos(panT * 0.09) * 120 - panT * 5;    // Drift upward vertically
const clampedX = Math.max(-900, Math.min(900, panX));   // Prevent edge exposure
```

**Parameters**:
- Sine period `0.09-0.15 rad/s`: slow, about one swing every 30-50 seconds
- Linear drift `5-8 px/s`: slower than a blink
- Amplitude `120-220 px`: large enough to feel, small enough not to cause discomfort

### Pattern E · Focus Overlay (Focus Switching)

**Key design**: the focus overlay is a **flat element**, not tilted, floating above the tilted canvas. The selected slide scales from its tile position (about 400×225) to screen center (960×540). The background canvas does not change tilt but **dims to 45%**:

```js
// Focus overlay (flat, centered)
focusOverlay.style.width = (startW + (endW - startW) * focusIntensity) + 'px';
focusOverlay.style.height = (startH + (endH - startH) * focusIntensity) + 'px';
focusOverlay.style.opacity = focusIntensity;

// Background cards dim but remain visible. Key: do not cover them 100%.
card.style.opacity = entryOp * (1 - 0.55 * focusIntensity);   // 1 → 0.45
card.style.filter = `brightness(${1 - 0.3 * focusIntensity})`;
```

**Clarity rules**:
- The focus overlay `<img>` must use `src` pointing directly to the original image. **Do not reuse compressed thumbnails from the gallery**.
- Preload all originals into a `new Image()[]` array.
- Compute overlay `width/height` on every frame so the browser resamples the original image each frame.

---

## Timeline Architecture (Reusable Skeleton)

```js
const T = {
  DURATION: 25.0,
  s1_in: [0.0, 0.8],    s1_type: [1.0, 3.2],  s1_out: [3.5, 4.0],
  s2_in: [3.9, 5.1],    s2_hold: [5.1, 7.0],  s2_out: [7.0, 7.8],
  s3_hold: [7.8, 8.3],  s3_ripple: [8.3, 10.0],
  panStart: 8.6,
  focuses: [
    { start: 11.0, end: 12.7, idx: 2  },
    { start: 13.3, end: 15.0, idx: 3  },
    { start: 15.6, end: 17.3, idx: 10 },
    { start: 17.9, end: 19.6, idx: 16 },
  ],
  s4_walloff: [21.1, 21.8], s4_in: [21.8, 22.7], s4_hold: [23.7, 25.0],
};

// Core easing
const easeOut = t => 1 - Math.pow(1 - t, 3);
const easeInOut = t => t < 0.5 ? 4*t*t*t : 1 - Math.pow(-2*t+2, 3)/2;
function lerp(time, start, end, fromV, toV, easing) {
  if (time <= start) return fromV;
  if (time >= end) return toV;
  let p = (time - start) / (end - start);
  if (easing) p = easing(p);
  return fromV + (toV - fromV) * p;
}

// A single render(t) function reads the timestamp and writes every element.
function render(t) { /* ... */ }
requestAnimationFrame(function tick(now) {
  const t = ((now - startMs) / 1000) % T.DURATION;
  render(t);
  requestAnimationFrame(tick);
});
```

**Architectural essence**: **derive all state from timestamp t**. No state machine, no setTimeout. This gives you:
- Instant jumps to any time, such as `window.__setTime(12.3)`, which is useful for Playwright frame captures
- Naturally seamless loops through `t mod DURATION`
- The ability to freeze any frame during debugging

---

## Texture Details (Easy to Miss, but Critical)

### 1. SVG Noise Texture

Light backgrounds most easily become too flat. Add an extremely weak fractalNoise layer:

```html
<style>
.stage::before {
  content: '';
  position: absolute; inset: 0;
  background-image: url("data:image/svg+xml;utf8,<svg xmlns='http://www.w3.org/2000/svg' width='200' height='200'><filter id='n'><feTurbulence type='fractalNoise' baseFrequency='0.85' numOctaves='2' stitchTiles='stitch'/><feColorMatrix values='0 0 0 0 0.078  0 0 0 0 0.078  0 0 0 0 0.074  0 0 0 0.035 0'/></filter><rect width='100%' height='100%' filter='url(%23n)'/></svg>");
  opacity: 0.5;
  pointer-events: none;
  z-index: 30;
}
</style>
```

It looks almost identical at first glance; removing it reveals what it contributed.

### 2. Corner Brand Mark

```html
<div class="corner-brand">
  <div class="mark"></div>
  <div>HUASHU · DESIGN</div>
</div>
```

```css
.corner-brand {
  position: absolute; top: 48px; left: 72px;
  font-family: var(--mono);
  font-size: 12px;
  letter-spacing: 0.22em;
  text-transform: uppercase;
  color: var(--muted);
}
```

Show it only during the work-wall scene, with fade in/out. It should feel like a gallery label.

### 3. Brand Closing Wordmark

```css
.brand-wordmark {
  font-family: var(--sans);
  font-size: 148px;
  font-weight: 700;
  letter-spacing: -0.045em;   /* Negative letter spacing is key: it compacts the word into a mark */
}
.brand-wordmark .accent {
  color: var(--accent);
  font-weight: 500;           /* The accent character is thinner, creating visual contrast */
}
```

`letter-spacing: -0.045em` is standard for large type on Apple product pages.

---

## Common Failure Modes

| Symptom | Cause | Fix |
|---|---|---|
| Looks like a PPT template | Cards have no shadow / hairline | Add two-layer box-shadow + 1px border |
| Tilt feels cheap | Only rotateY was used, with no rotateZ | Add ±2-3deg rotateZ to break regularity |
| Pan feels "stuttery" | Used setTimeout or looping CSS keyframes | Use rAF + continuous sin/cos functions |
| Text is unclear during focus | Reused low-resolution gallery tile image | Independent overlay + original image src |
| Background feels empty | Solid `#F5F5F7` | Add SVG fractalNoise at 0.5 opacity |
| Typography feels too "internet" | Only Inter | Add serif stacks for Chinese and English, plus mono |

---

## References

- Complete implementation sample: `/Users/alchain/Documents/writing/01-wechat-writing/projects/2026.04-huashu-design-release/images/hero-animation-v5.html`
- Original inspiration: claude.ai/design hero video
- Reference aesthetics: Apple product pages, Dribbble shot collection pages

When you encounter an animation need that says "many high-quality outputs must be exhibited", copy the skeleton from this file directly, swap the content, and tune the timing.
