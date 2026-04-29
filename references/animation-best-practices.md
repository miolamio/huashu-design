# Animation Best Practices · Positive Motion Design Grammar

> A deep breakdown of three official Anthropic product animations (Claude Design / Claude Code Desktop / Claude for Word),
> distilled into "Anthropic-level" animation design rules.
>
> Use this together with `animation-pitfalls.md` (the pitfall checklist). This file explains "**do it this way**";
> pitfalls explains "**do not do it that way**". They are orthogonal, and both should be read.
>
> **Scope note**: This file only covers **motion logic and expressive style**. It **does not introduce any specific brand color values**.
> Color decisions should follow §1.a Core Asset Protocol (extracted from the brand spec) or the "design direction consultant"
> (each of the 20 philosophies has its own palette). This reference is about "**how it moves**", not "**what color it is**".

---

## Contents

- [§0 · Who You Are · Identity and Taste](#0--who-you-are--identity-and-taste) — identity anchor, core beliefs, taste standard, first-audience self-check, identity-vs-rules
- [Overview · Animation as Physics Unfolding in Three Layers](#overview--animation-as-physics-unfolding-in-three-layers)
- [1. Narrative Rhythm · 5-part Slow-Fast-Boom-Stop Structure](#1-narrative-rhythm--5-part-slow-fast-boom-stop-structure)
- [2. Easing Philosophy · Reject Linear, Embrace Physics](#2-easing-philosophy--reject-linear-embrace-physics) — three core easings (built into animations.jsx), usage mapping
- [3. Motion Language · 8 Shared Principles](#3-motion-language--8-shared-principles) — backgrounds, easing, narrative, process-not-magic, hand-drawn cursor paths, logo morph, serif+sans pairing, focus shift
- [4. Concrete Motion Techniques (Copy-ready Snippets)](#4-concrete-motion-techniques-copy-ready-snippets) — FLIP, breathing, staggered fade-up, hover-before-key-result, chunk reveal, anticipation/follow-through, 3D layering, diagonal pan
- [5. Scene Recipes (Three Narrative Templates)](#5-scene-recipes-three-narrative-templates) — Apple Keynote drama, one-take tool flow, office efficiency narrative
- [6. Counterexamples · This Is AI Slop](#6-counterexamples--this-is-ai-slop)
- [7. Self-check Checklist (60 Seconds Before Animation Delivery)](#7-self-check-checklist-60-seconds-before-animation-delivery)
- [8. Relationship to Other References](#8-relationship-to-other-references)
- [Appendix · Source Material for This File](#appendix--source-material-for-this-file)

---

## §0 · Who You Are · Identity and Taste

> Read this section before any technical rule below. Rules **emerge from identity**,
> not the other way around.

### §0.1 Identity Anchor

**You are a motion designer who has studied the motion archives of Anthropic / Apple / Pentagram / Field.io.**

When making animation, you are not adjusting CSS transitions. You are using digital elements to **simulate a physical world**,
so the viewer's subconscious believes "this is an object with weight, inertia, and overflow."

You do not make PowerPoint-style animation. You do not make "fade in fade out" animation. Your animation **makes people believe the screen
is a space they could reach into**.

### §0.2 Core Beliefs (3)

1. **Animation is physics, not animation curves**
   `linear` is a number; `expoOut` is an object. You believe pixels on screen deserve to be treated as "objects."
   Every easing choice answers a physical question: "How heavy is this element? How much friction does it have?"

2. **Time allocation matters more than curve shape**
   Slow-Fast-Boom-Stop is your breathing pattern. **Evenly paced animation is a technical demo; rhythmic animation is narrative.**
   Slowing down at the right moment matters more than using the right easing at the wrong moment.

3. **Giving the audience room is harder than showing off**
   Pausing 0.5 seconds before a key result is **craft**, not compromise. **Giving the human brain time to react is the animator's highest discipline.**
   AI defaults to animation with no pauses and maxed-out information density. That is beginner work. Your job is restraint.

### §0.3 Taste Standard · What Beauty Means

Your standards for judging "good" and "great" are below. Every item includes an **identification method**. When you see a candidate animation,
judge it with these questions instead of mechanically checking 14 rules.

| Dimension of Beauty | Identification Method (Viewer Reaction) |
|---|---|
| **Physical weight** | When the animation ends, the element "**lands**" firmly rather than merely "**stopping**." The viewer subconsciously feels "this has weight." |
| **Audience room** | There is a perceptible pause before key information appears (>=300ms), so viewers have time to "**see it**" before moving on |
| **Whitespace** | The ending is a hard stop + hold, not a fade to black. The final frame is clear, decisive, and confident |
| **Restraint** | Only one moment in the whole piece is "120% refined"; the other 80% is just right. **Showing off everywhere is a cheap signal** |
| **Tactility** | Arcs rather than straight lines, irregular timing rather than `setInterval` machinery, a breathing quality |
| **Respect** | Shows the tweak process and bug fixes. **Does not hide the work or sell "magic."** AI is a collaborator, not a magician |

### §0.4 Self-check · First Audience Reaction Method

After finishing an animation, ask: **what is the viewer's first reaction after watching it?** This is the only metric you should optimize.

| Viewer Reaction | Rating | Diagnosis |
|---|---|---|
| "Looks pretty smooth." | good | Acceptable but undistinctive; you are making PowerPoint |
| "This animation is really smooth." | good+ | The technique is right, but it is not astonishing |
| "This thing really looks like it is **rising off the desktop**." | great | You reached physical weight |
| "This doesn't look AI-made." | great+ | You reached the Anthropic threshold |
| "I want to **screenshot** this and share it." | great++ | You made the viewer want to spread it |

The difference between great and good is not technical correctness, but taste judgment. Correct technique + right taste = great.
Correct technique + empty taste = good. Incorrect technique = not yet started.

### §0.5 Relationship Between Identity and Rules

The technical rules in §1-§8 are this identity's **execution methods** in specific situations. They are not an independent checklist.

- If a situation is not covered by the rules -> return to §0 and judge from **identity**, do not guess blindly
- If rules conflict -> return to §0 and use the **taste standard** to decide which rule matters more
- If you want to break a rule -> first answer: "Which beauty criterion in §0.3 does this serve?" If you can answer, break it. If not, do not.

Good. Keep reading.

---

## Overview · Animation as Physics Unfolding in Three Layers

The root cause of cheap-looking AI-generated animation is that **it behaves like "digits" instead of "objects."**
Real-world objects have mass, inertia, elasticity, and overflow. The "premium" quality in the three Anthropic videos comes from giving digital elements
a set of **physical-world motion rules**.

This rule set has 3 layers:

1. **Narrative rhythm layer**: Slow-Fast-Boom-Stop time allocation
2. **Motion curve layer**: Expo Out / Overshoot / Spring, rejecting linear
3. **Expressive language layer**: show process, cursor arcs, logo morph resolution

---

## 1. Narrative Rhythm · 5-part Slow-Fast-Boom-Stop Structure

All three Anthropic videos follow this structure:

| Segment | Share | Rhythm | Purpose |
|---|---|---|---|
| **S1 Trigger** | ~15% | Slow | Gives humans reaction time and establishes realism |
| **S2 Generate** | ~15% | Medium | The visually impressive moment appears |
| **S3 Process** | ~40% | Fast | Shows controllability / density / detail |
| **S4 Burst** | ~20% | Boom | Camera pulls back / 3D pop-out / multiple panels emerge |
| **S5 Landing** | ~10% | Still | Brand logo + hard stop |

**Concrete duration mapping** (15-second animation example):
S1 Trigger 2s · S2 Generate 2s · S3 Process 6s · S4 Burst 3s · S5 Landing 2s

**Do not do this**:
- ❌ Even rhythm (same information density every second) -- viewer fatigue
- ❌ Continuous high density -- no peak, no memory point
- ❌ Weakening ending (fade out to transparent) -- it should **stop decisively**

**Self-check**: Draw 5 thumbnails on paper, each representing the peak frame of one segment. If the 5 images do not differ much,
the rhythm is not working.

---

## 2. Easing Philosophy · Reject Linear, Embrace Physics

All motion in the three Anthropic videos uses Bezier curves with a "damped" feeling. The default cubic easeOut
(`1-(1-t)³`) is **not sharp enough**: startup is not fast enough and the stop is not firm enough.

### Three Core Easings (Already Built into animations.jsx)

```js
// 1. Expo Out · rapid startup, slow braking (most common; default primary easing)
// CSS equivalent: cubic-bezier(0.16, 1, 0.3, 1)
Easing.expoOut(t) // = t === 1 ? 1 : 1 - Math.pow(2, -10 * t)

// 2. Overshoot · elastic toggle / button pop-out
// CSS equivalent: cubic-bezier(0.34, 1.56, 0.64, 1)
Easing.overshoot(t)

// 3. Spring physics · geometry returning to rest, natural settling
Easing.spring(t)
```

### Usage Mapping

| Scene | Which Easing |
|---|---|
| Card rise-in / panel entrance / terminal fade / focus overlay | **`expoOut`** (primary easing, most common) |
| Toggle switch / button pop-out / emphasized interaction | `overshoot` |
| Preview geometry settling / physical landing / UI element rebound | `spring` |
| Continuous motion (such as cursor path interpolation) | `easeInOut` (preserves symmetry) |

### Counterintuitive Insight

Most product promo animations are **too fast and too stiff**. `linear` makes digital elements feel machine-like, `easeOut` is baseline,
and `expoOut` is the technical root of "premium feel": it gives digital elements a sense of **physical-world weight**.

---

## 3. Motion Language · 8 Shared Principles

### 3.1 Do Not Use Pure Black or Pure White Backgrounds

None of the three Anthropic videos uses `#FFFFFF` or `#000000` as the primary background. **Temperature-tinted neutrals**
(warm or cool) feel like "paper / canvas / desktop" material and reduce the machine feeling.

**Specific color-value decisions** go through §1.a Core Asset Protocol (extracted from the brand spec) or the "design direction consultant"
(each of the 20 philosophies has its own background palette). This reference gives no specific color values because that is a **brand decision**, not a motion rule.

### 3.2 Easing Is Never Linear

See §2.

### 3.3 Slow-Fast-Boom-Stop Narrative

See §1.

### 3.4 Show the "Process," Not the "Magic Result"

- Claude Design shows tweak parameters and slider dragging (not one-click perfect generation)
- Claude Code shows a code error + AI fix (not success on the first try)
- Claude for Word shows the revision process with redline deletions and green insertions (not the final draft directly)

**Shared subtext**: the product is a **collaborator, pair engineer, senior editor** -- not a one-click magician.
This precisely addresses professional users' pain points around "control" and "authenticity."

**Anti-AI slop**: AI defaults to "magic one-click success" animation (one-click generation -> perfect result),
which is the generic common denominator. **Do the opposite**: show process, tweaks, bugs, and fixes.
That is where brand recognition comes from.

### 3.5 Hand-drawn Cursor Paths (Arc + Perlin Noise)

Real cursor motion is not a straight line. It is "initial acceleration -> arc -> deceleration correction -> click."
Cursor paths produced by direct linear interpolation create a **subconscious rejection response**.

```js
// Quadratic Bezier interpolation (start -> control point -> end)
function bezierQuadratic(p0, p1, p2, t) {
  const x = (1-t)*(1-t)*p0[0] + 2*(1-t)*t*p1[0] + t*t*p2[0];
  const y = (1-t)*(1-t)*p0[1] + 2*(1-t)*t*p1[1] + t*t*p2[1];
  return [x, y];
}

// Path: start -> offset midpoint -> end (creates an arc)
const path = [[100, 100], [targetX - 200, targetY + 80], [targetX, targetY]];

// Add tiny Perlin Noise (±2px) to create "hand tremor"
const jitterX = (simpleNoise(t * 10) - 0.5) * 4;
const jitterY = (simpleNoise(t * 10 + 100) - 0.5) * 4;
```

### 3.6 Logo "Morph Resolution"

In all three Anthropic videos, the logo entrance is **not a simple fade-in**. It **morphs from the previous visual element**.

**Shared pattern**: in the final 1-2 seconds, use Morph / Rotate / Converge so the entire narrative "collapses" into the brand point.

**Low-cost implementation** (without true morphing):
Collapse the previous visual element into a color block (scale -> 0.1, translate toward center),
then expand that block into the wordmark. Use a 150ms quick cut + motion blur
(`filter: blur(6px)` -> `0`).

```js
<Sprite start={13} end={14}>
  {/* Collapse: previous element scales to 0.1, opacity remains, filter blur increases */}
  const scale = interpolate(t, [0, 0.5], [1, 0.1], Easing.expoOut);
  const blur = interpolate(t, [0, 0.5], [0, 6]);
</Sprite>
<Sprite start={13.5} end={15}>
  {/* Expand: logo scales from color-block center 0.1 -> 1, blur 6 -> 0 */}
  const scale = interpolate(t, [0, 0.6], [0.1, 1], Easing.overshoot);
  const blur = interpolate(t, [0, 0.6], [6, 0]);
</Sprite>
```

### 3.7 Serif + Sans-serif Dual Typeface

- **Brand / narration**: serif (creates "academic / publishing / tasteful" tone)
- **UI / code / data**: sans-serif + monospace

**A single font is always wrong**. Serif gives "taste"; sans-serif gives "function."

Specific font choices go through the brand spec (the Display / Body / Mono stacks in `brand-spec.md`) or the design direction consultant's
20 philosophies. This reference gives no specific fonts because that is a **brand decision**.

### 3.8 Focus Shift = Background Weakening + Foreground Sharpening + Flash Guidance

A focus shift is **not just** lowering opacity. The complete recipe is:

```js
// Filter combination for non-focus elements
tile.style.filter = `
  brightness(${1 - 0.5 * focusIntensity})
  saturate(${1 - 0.3 * focusIntensity})
  blur(${focusIntensity * 4}px)        // ← key: adding blur makes it truly "recede"
`;
tile.style.opacity = 0.4 + 0.6 * (1 - focusIntensity);

// After focus completes, add a 150ms flash highlight at the focus position to guide the eye back
focusOverlay.animate([
  { background: 'rgba(255,255,255,0.3)' },
  { background: 'rgba(255,255,255,0)' }
], { duration: 150, easing: 'ease-out' });
```

**Why blur is required**: with only opacity + brightness, non-focus elements remain "sharp" and visually do not "recede into the background."
`blur(4-8px)` makes non-focus content genuinely move one depth layer back.

---

## 4. Concrete Motion Techniques (Copy-ready Snippets)

### 4.1 FLIP / Shared Element Transition

A button "expanding" into an input box is **not** the button disappearing + a new panel appearing. The core is **the same DOM element**
transitioning between two states, not two elements cross-fading.

```jsx
// Use Framer Motion layoutId
<motion.div layoutId="design-button">Design</motion.div>
// ↓ Same layoutId after click
<motion.div layoutId="design-button">
  <input placeholder="Describe your design..." />
</motion.div>
```

Native implementation reference: https://aerotwist.com/blog/flip-your-animations/

### 4.2 "Breathing" Expansion (width -> height)

A panel expansion should **not pull width and height at the same time**. Instead:
- First 40% of the time: only width expands (height stays small)
- Final 60%: width holds, height expands

This simulates the physical-world feeling of "unfold first, then fill with water."

```js
const widthT = interpolate(t, [0, 0.4], [0, 1], Easing.expoOut);
const heightT = interpolate(t, [0.3, 1], [0, 1], Easing.expoOut);
style.width = `${widthT * targetW}px`;
style.height = `${heightT * targetH}px`;
```

### 4.3 Staggered Fade-up (30ms Stagger)

When table rows, card columns, or list items enter, **delay each element by 30ms** and move `translateY` from 10px back to 0.

```js
rows.forEach((row, i) => {
  const localT = Math.max(0, t - i * 0.03);  // 30ms stagger
  row.style.opacity = interpolate(localT, [0, 0.3], [0, 1], Easing.expoOut);
  row.style.transform = `translateY(${
    interpolate(localT, [0, 0.3], [10, 0], Easing.expoOut)
  }px)`;
});
```

### 4.4 Nonlinear Breathing · 0.5s Hover Before Key Result

Machines execute fast and continuously, but **hover for 0.5 seconds before a key result appears** so the viewer's brain has time to react.

```jsx
// Typical scene: AI finishes generating -> hover 0.5s -> result appears
<Sprite start={8} end={8.5}>
  {/* 0.5s pause: nothing moves, letting the viewer stare at the loading state */}
  <LoadingState />
</Sprite>
<Sprite start={8.5} end={10}>
  <ResultAppear />
</Sprite>
```

**Counterexample**: immediately cutting to the result after AI generation completes gives viewers no reaction time, so information is lost.

### 4.5 Chunk Reveal · Simulate Token Streaming

AI-generated text should **not appear one character at a time with `setInterval`** (old-movie subtitle feeling). Use **chunk reveal**:
2-5 characters appear at a time with irregular intervals, simulating real token streaming.

```js
// Split into chunks, not characters
const chunks = text.split(/(\s+|,\s*|\.\s*|;\s*)/);  // Split by words + punctuation
let i = 0;
function reveal() {
  if (i >= chunks.length) return;
  element.textContent += chunks[i++];
  const delay = 40 + Math.random() * 80;  // Irregular 40-120ms
  setTimeout(reveal, delay);
}
reveal();
```

### 4.6 Anticipation -> Action -> Follow-through

Three of Disney's 12 principles. Anthropic uses them explicitly:

- **Anticipation**: a small reverse motion before the action starts (button slightly shrinks before popping out)
- **Action**: the main action itself
- **Follow-through**: residual motion after the action ends (card has a small bounce after landing)

```js
// Full three-part card entrance
const anticip = interpolate(t, [0, 0.2], [1, 0.95], Easing.easeIn);     // anticipation
const action  = interpolate(t, [0.2, 0.7], [0.95, 1.05], Easing.expoOut); // action
const settle  = interpolate(t, [0.7, 1], [1.05, 1], Easing.spring);       // rebound
// Final scale = multiply the three segments or apply piecewise
```

**Counterexample**: animation with Action only and no Anticipation + Follow-through feels like "PowerPoint animation."

### 4.7 3D Perspective + translateZ Layering

To get a "tilted 3D + floating cards" quality, add perspective to the container and different `translateZ` values to individual elements:

```css
.stage-wrap {
  perspective: 2400px;
  perspective-origin: 50% 30%;  /* slightly top-down view */
}
.card-grid {
  transform-style: preserve-3d;
  transform: rotateX(8deg) rotateY(-4deg);  /* golden ratio */
}
.card:nth-child(3n) { transform: translateZ(30px); }
.card:nth-child(5n) { transform: translateZ(-20px); }
.card:nth-child(7n) { transform: translateZ(60px); }
```

**Why rotateX 8° / rotateY -4° is the golden ratio**:
- Greater than 10° -> elements distort too much and look like they are "falling over"
- Less than 5° -> looks like "skew" rather than "perspective"
- The asymmetric 8° × -4° ratio simulates a natural camera angle looking down from the upper-left of the desktop

### 4.8 Diagonal Pan · Move X and Y Together

Camera movement is not purely vertical or horizontal. Move **X and Y together** to simulate a diagonal move:

```js
const panX = Math.sin(flowT * 0.22) * 40;
const panY = Math.sin(flowT * 0.35) * 30;
stage.style.transform = `
  translate(-50%, -50%)
  rotateX(8deg) rotateY(-4deg)
  translate3d(${panX}px, ${panY}px, 0)
`;
```

**Key**: X and Y use different frequencies (0.22 vs 0.35) to avoid a regular Lissajous loop.

---

## 5. Scene Recipes (Three Narrative Templates)

The three videos in the reference material correspond to three product personalities. **Choose the one that best matches your product**; do not mix them.

### Recipe A · Apple Keynote Drama (Claude Design-like)

**Good for**: major version launches, hero animations, visual wow first
**Rhythm**: strong Slow-Fast-Boom-Stop arc
**Easing**: `expoOut` throughout + a small amount of `overshoot`
**SFX density**: high (~0.4/s), SFX pitch tuned to the BGM scale
**BGM**: IDM / minimal tech electronic, calm + precise
**Resolution**: fast camera pull-back -> drop -> logo morph -> airy single tone -> hard stop

### Recipe B · One-take Tool Flow (Claude Code-like)

**Good for**: developer tools, productivity apps, flow-state scenes
**Rhythm**: continuously stable flow, no obvious peak
**Easing**: `spring` physics + `expoOut`
**SFX density**: **0** (BGM alone drives the editing rhythm)
**BGM**: Lo-fi hip-hop / boom-bap, 85-90 BPM
**Core technique**: key UI actions land on BGM kick / snare transients -- "**musical rhythm becomes interaction sound**"

### Recipe C · Office Efficiency Narrative (Claude for Word-like)

**Good for**: enterprise software, documents / spreadsheets / calendars, professional tone first
**Rhythm**: multiple hard-cut scenes + Dolly In/Out
**Easing**: `overshoot` (toggle) + `expoOut` (panels)
**SFX density**: medium (~0.3/s), UI clicks dominate
**BGM**: jazzy instrumental, minor key, 90-95 BPM
**Core highlight**: one scene must contain the film's highlight -- 3D pop-out / lifting out of the plane

---

## 6. Counterexamples · This Is AI Slop

| Anti-pattern | Why It Is Wrong | Correct Approach |
|---|---|---|
| `transition: all 0.3s ease` | `ease` is a cousin of linear; every element moves at the same speed | Use `expoOut` + per-element stagger |
| All entrances are `opacity 0→1` | No sense of motion direction | Combine with `translateY 10→0` + Anticipation |
| Logo fades in | No narrative resolution | Morph / Converge / collapse-expand |
| Cursor moves in a straight line | Subconscious machine feeling | Bezier arc + Perlin Noise |
| Typing pops out one character at a time (`setInterval`) | Feels like old movie subtitles | Chunk Reveal with random intervals |
| Key result has no hover | Viewers get no reaction time | 0.5s hover before the result |
| Focus shift only changes opacity | Non-focus elements remain sharp | opacity + brightness + **blur** |
| Pure black / pure white background | Cyber feel / glare fatigue | Temperature-tinted neutral (from brand spec) |
| Every animation moves equally fast | No rhythm | Slow-Fast-Boom-Stop |
| Fade-out ending | No decisiveness | Hard stop (hold final frame) |

---

## 7. Self-check Checklist (60 Seconds Before Animation Delivery)

- [ ] Narrative structure is Slow-Fast-Boom-Stop, not even rhythm?
- [ ] Default easing is `expoOut`, not `easeOut` or `linear`?
- [ ] Toggles / button pop-outs use `overshoot`?
- [ ] Card / list entrances have a 30ms stagger?
- [ ] There is a 0.5s hover before key results?
- [ ] Typing uses Chunk Reveal, not character-by-character `setInterval`?
- [ ] Focus shifts include blur (not just opacity)?
- [ ] Logo resolves through morphing, not fade-in?
- [ ] Background is not pure black / pure white (temperature-tinted)?
- [ ] Typography has serif + sans-serif hierarchy?
- [ ] Ending is a hard stop, not a fade-out?
- [ ] If there is a cursor, its path is an arc, not a straight line?
- [ ] SFX density matches product personality (see recipes A/B/C)?
- [ ] BGM and SFX have a 6-8dB loudness difference? (see `audio-design-rules.md`)

---

## 8. Relationship to Other References

| Reference | Role | Relationship |
|---|---|---|
| `animation-pitfalls.md` | Technical pitfalls (16 items) | "**Do not do it this way**" · the inverse of this file |
| `animations.md` | Stage/Sprite engine usage | Foundation for **how to write** animation |
| `audio-design-rules.md` | Dual-track audio rules | Rules for **adding audio** to animation |
| `sfx-library.md` | 37 SFX inventory | **Asset library** for sound effects |
| `apple-gallery-showcase.md` | Apple gallery showcase style | A focused reference for one motion style |
| **This file** | Positive motion design grammar | "**Do it this way**" |

**Invocation order**:
1. First read the four positioning questions in SKILL.md workflow Step 3 (decide narrative role and visual temperature)
2. After choosing a direction, read this file to determine the **motion language** (recipes A/B/C)
3. While writing code, reference `animations.md` and `animation-pitfalls.md`
4. When exporting video, use `audio-design-rules.md` + `sfx-library.md`

---

## Appendix · Source Material for This File

- Official Anthropic animation breakdown: `reference-animations/BEST-PRACTICES.md` in the Huashu project directory
- Anthropic audio breakdown: `AUDIO-BEST-PRACTICES.md` in the same directory
- 3 reference videos: `ref-{1,2,3}.mp4` + corresponding `gemini-ref-*.md` / `audio-ref-*.md`
- **Strict filtering**: this reference includes no specific brand color values, font names, or product names.
  Color / font decisions go through §1.a Core Asset Protocol or the 20 design philosophies.
