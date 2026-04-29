# Gallery Ripple + Multi-Focus · Scene Choreography Philosophy

> A reusable visual choreography structure distilled from the huashu-design hero animation v9: 25 seconds, 8 scenes.
> This is not an animation production pipeline; it explains **when this choreography is the right choice**.
> Field reference: [demos/hero-animation-v9.mp4](../demos/hero-animation-v9.mp4) · [https://www.huasheng.ai/huashu-design-hero/](https://www.huasheng.ai/huashu-design-hero/)

## One-Sentence Rule

> **When you have 20+ homogeneous visual assets and the scene needs to express scale and depth, prefer Gallery Ripple + Multi-Focus over stacked layout.**

General SaaS feature animations, product launches, skill promotion, and series portfolio showcases can all benefit from this structure when there are enough assets and the style is consistent.

---

## What This Technique Actually Expresses

It is not "showing off assets." It tells a narrative through **two rhythm changes**:

**Beat 1 · Ripple Expansion (~1.5s)**: 48 cards spread outward from the center. The audience is struck by quantity: "Oh, this thing has produced that much."

**Beat 2 · Multi-Focus (~8s, 4 loops)**: while the camera pans slowly, the background dims and desaturates 4 times, and one card at a time expands to the screen center. The audience shifts from "impact of quantity" to "gaze at quality"; each focus holds a stable 1.7s rhythm.

**Core narrative structure**: **Scale (Ripple) → Gaze (Focus × 4) → Fade-out (Walloff)**. Together, the three beats express "Breadth × Depth": not only can it produce a lot, each output is also worth stopping to inspect.

Compare the counterexamples:

| Approach | Audience Perception |
|------|---------|
| 48 cards in a static grid, with no Ripple | Looks good but has no narrative; feels like a grid screenshot |
| Fast cuts one card at a time, with no gallery context | Feels like a slideshow and loses scale |
| Ripple only, with no Focus | Impressive but leaves no specific card in memory |
| **Ripple + Focus × 4 (this recipe)** | **First shocked by quantity, then drawn into quality, then calmly faded out: a complete emotional arc** |

---

## Preconditions (All Required)

This choreography **is not universal**. All 4 conditions below must be met:

1. **Asset count ≥ 20, ideally 30+**
   Below 20, Ripple feels empty. The density comes from every one of the 48 cells moving. v9 used 48 cells × 32 images, loop-filled.

2. **Consistent visual style across assets**
   All 16:9 slide previews / all app screenshots / all cover designs. Aspect ratio, tone, and layout must feel like "one set." Mixed formats make the gallery look like a clipboard.

3. **Each asset still contains readable information when enlarged**
   Focus scales one card up to 960px wide. If the original is blurry or information-poor after enlargement, that beat fails. Reverse test: can you pick 4 cards from the 48 as the "most representative" ones? If not, the asset quality is uneven.

4. **The scene itself is landscape or square, not vertical**
   The gallery's 3D tilt (`rotateX(14deg) rotateY(-10deg)`) needs horizontal extension. Vertical video makes the tilt feel narrow and awkward.

**Fallback paths when conditions are missing**:

| Missing Condition | Degrade To |
|-------|-----------|
| Assets < 20 | "3-5 cards side by side + individual focus" |
| Inconsistent style | Keynote-style "cover + 3 chapter hero images" |
| Thin information | "Data-driven dashboard" or "quote + large type" |
| Vertical scene | "Vertical scroll + sticky cards" |

---

## Technical Recipe (v9 Field Parameters)

### 4-Layer Structure

```
viewport (1920×1080, perspective: 2400px)
  └─ canvas (4320×2520, oversized overflow) → 3D tilt + pan
      └─ 8×6 grid = 48 cards (gap 40px, padding 60px)
          └─ img (16:9, border-radius 9px)
      └─ focus-overlay (absolute center, z-index 40)
          └─ img (matches selected slide)
```

**Key**: the canvas is 2.25× larger than the viewport, which makes pan feel like peeking into a larger world.

### Ripple Expansion (Distance-Delayed Algorithm)

```js
// Each card's entry time = distance from center × 0.8s delay.
const col = i % 8, row = Math.floor(i / 8);
const dc = col - 3.5, dr = row - 2.5;       // Offset from center
const dist = Math.hypot(dc, dr);
const maxDist = Math.hypot(3.5, 2.5);
const delay = (dist / maxDist) * 0.8;       // 0 → 0.8s
const localT = Math.max(0, (t - rippleStart - delay) / 0.7);
const opacity = expoOut(Math.min(1, localT));
```

**Core parameters**:
- Total duration 1.7s (`T.s3_ripple: [8.3, 10.0]`)
- Max delay 0.8s: center appears first, corners last
- Per-card entry duration 0.7s
- Easing: `expoOut`, for impact rather than smoothness

**Simultaneous action**: canvas scale changes from 1.25 → 0.94, a zoom-out-to-reveal that synchronizes with the appearing cards.

### Multi-Focus (4 Beats)

```js
T.focuses = [
  { start: 11.0, end: 12.7, idx: 2  },  // 1.7s
  { start: 13.3, end: 15.0, idx: 3  },  // 1.7s
  { start: 15.6, end: 17.3, idx: 10 },  // 1.7s
  { start: 17.9, end: 19.6, idx: 16 },  // 1.7s
];
```

**Rhythm pattern**: each focus lasts 1.7s, with a 0.6s breath between them. Total: 8s, from 11.0 to 19.6s.

**Inside each focus**:
- In ramp: 0.4s (`expoOut`)
- Hold: middle 0.9s (`focusIntensity = 1`)
- Out ramp: 0.4s (`easeOut`)

**Background change, which is critical**:

```js
if (focusIntensity > 0) {
  const dimOp = entryOp * (1 - 0.6 * focusIntensity);  // dim to 40%
  const brt = 1 - 0.32 * focusIntensity;                // brightness 68%
  const sat = 1 - 0.35 * focusIntensity;                // saturate 65%
  card.style.filter = `brightness(${brt}) saturate(${sat})`;
}
```

**Not opacity alone: desaturate and darken at the same time.** This makes the foreground overlay's color pop, rather than merely appearing "a little brighter."

**Focus overlay size animation**:
- 400×225 on entry → 960×540 in hold state
- 3 layers of shadow + a 3px accent-color outline ring to create a framed feeling

### Pan (Continuous Motion Keeps Stillness Interesting)

```js
const panT = Math.max(0, t - 8.6);
const panX = Math.sin(panT * 0.12) * 220 - panT * 8;
const panY = Math.cos(panT * 0.09) * 120 - panT * 5;
```

- Sine wave + linear drift creates two layers of motion. It is not a pure loop, so every moment has a different position.
- X/Y frequencies differ (0.12 vs 0.09) to keep the pattern from looking obviously cyclic.
- Clamp to ±900/500px to prevent drifting out of bounds.

**Why not pure linear pan**: with pure linear movement, the audience can predict where the image will be next second. Sine + drift makes each second feel new. Under 3D tilt, it creates a slight spatial sway, the useful kind that holds attention.

---

## 5 Reusable Patterns (Distilled from v6→v9 Iteration)

### 1. **Use expoOut as the Main Easing, Not cubicOut**

`easeOut = 1 - (1-t)³` (smooth) vs `expoOut = 1 - 2^(-10t)` (explosive, then rapidly settles).

**Reason**: in the first 30%, expoOut quickly reaches 90%, which feels more like physical damping and matches the intuition of "a heavy object landing." It is especially suitable for:
- Card entry, for weight
- Ripple spread, for shockwave impact
- Brand rise, for a settled landing

**When to still use cubicOut**: focus out ramps and symmetrical micro-motion.

### 2. **Paper-Tone Background + Terracotta Orange Accent (Anthropic Lineage)**

```css
--bg: #F7F4EE;        /* Warm paper */
--ink: #1D1D1F;       /* Nearly black */
--accent: #D97757;    /* Terracotta orange */
--hairline: #E4DED2;  /* Warm line */
```

**Why**: a warm background keeps a sense of breathing space after GIF compression, unlike pure white, which feels too screen-like. Terracotta orange is the single accent running through the terminal prompt, selected dir-card, cursor, brand hyphen, and focus ring. All visual anchors are strung together by one color.

**v5 lesson**: adding a noise overlay to simulate paper grain destroyed GIF compression because every frame differed. v6 switched to "background color + warm shadow only." Paper feel retained 90%, and GIF size dropped 60%.

### 3. **Two Shadow Levels Simulate Depth; Avoid Real 3D**

```css
.gallery-card.depth-near { box-shadow: 0 32px 80px -22px rgba(60,40,20,0.22), ... }
.gallery-card.depth-far  { box-shadow: 0 14px 40px -16px rgba(60,40,20,0.10), ... }
```

Use a deterministic algorithm, `sin(i × 1.7) + cos(i × 0.73)`, to assign each card one of three shadow levels: near/mid/far. It gives a **visual sense of 3D stacking**, while each frame's transform remains unchanged and GPU cost stays 0.

**Cost of real 3D**: giving each card its own `translateZ` means the GPU calculates 48 transforms plus shadow blur every frame. v4 tried this and struggled even with Playwright recording at 25fps. v6's two-level shadow looks less than 5% different to the eye, but costs 10× less.

### 4. **Weight Changes (`font-variation-settings`) Feel More Cinematic Than Size Changes**

```js
const wght = 100 + (700 - 100) * morphP;  // 100 → 700 over 0.9s
wordmark.style.fontVariationSettings = `"wght" ${wght.toFixed(0)}`;
```

The brand wordmark morphs from Thin → Bold over 0.9s, paired with subtle letter-spacing adjustment (-0.045 → -0.048em).

**Why this is better than scaling**:
- Audiences have seen scale-up/scale-down too many times; expectations are fixed
- Weight change feels like "inner filling," as if the word is gaining substance, not being pushed closer
- Variable fonts became common only after 2020, so audiences subconsciously read it as modern

**Limit**: requires a font that supports variable axes, such as Inter, Roboto Flex, or Recursive. Static fonts can only imitate the effect by switching between fixed weights, which causes jumps.

### 5. **Low-Intensity Persistent Corner Brand**

During the Gallery stage, add a small `HUASHU · DESIGN` mark in the upper-left corner, 16% opacity, 12px type, wide letter spacing.

**Why add it**:
- After the Ripple burst, audiences can lose the anchor of what they are watching. A light upper-left label helps orient them.
- It feels more premium than a full-screen logo. People who do brand work know the brand signature does not need to shout.
- When the GIF is screenshotted and shared, ownership remains visible.

**Rule**: show it only in the middle segment when the frame is busy. Hide it at the opening so it does not cover the terminal. Hide it at the ending because the brand reveal is the protagonist.

---

## Counterexamples: When Not to Use This Choreography

**❌ Product demo where functionality must be shown**: the gallery flashes each card too quickly, so the audience will not remember any one feature. Use "single-screen focus + tooltip annotations" instead.

**❌ Data-driven content**: the audience needs time to read numbers, and the gallery rhythm does not allow that. Use "data chart + staged reveal" instead.

**❌ Story narrative**: the gallery is a parallel structure, while story needs causality. Use keynote-style chapter transitions instead.

**❌ Only 3-5 assets**: Ripple lacks density and looks patched together. Use "static arrangement + one-by-one highlight" instead.

**❌ Vertical 9:16**: 3D tilt needs horizontal extension; in vertical format the tilt feels crooked rather than expansive.

---

## How to Judge Whether Your Task Fits

Three-step quick check:

**Step 1 · Asset count**: count how many same-type visual assets you have. < 15 → stop; 15-25 → gather more; 25+ → use it directly.

**Step 2 · Consistency test**: place 4 random assets side by side. Do they look like "one set"? If not, unify the style first or choose another plan.

**Step 3 · Narrative fit**: are you trying to express "Breadth × Depth" (quantity × quality)? Or are you expressing "process," "function," or "story"? If it is not Breadth × Depth, do not force this pattern.

If all three are yes, fork the v6 HTML directly and update the `SLIDE_FILES` array plus the timeline. Change the palette through `--bg / --accent / --ink`; this reskins the whole piece without changing the underlying structure.

---

## Related Reference

- Complete technical workflow: [references/animations.md](animations.md) · [references/animation-best-practices.md](animation-best-practices.md)
- Animation export pipeline: [references/video-export.md](video-export.md)
- Audio setup (BGM + SFX dual track): [references/audio-design-rules.md](audio-design-rules.md)
- Cross-reference for the Apple gallery style: [references/apple-gallery-showcase.md](apple-gallery-showcase.md)
- Source HTML (v6 + audio-integrated version): `www.huasheng.ai/huashu-design-hero/index.html`
