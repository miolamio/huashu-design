# Animations: Timeline Animation Engine

Read this when creating animation / motion design HTML. It covers the core ideas, usage, and common patterns.

## Core Pattern: Stage + Sprite

Our animation system (`assets/animations.jsx`) provides a timeline-driven engine:

- **`<Stage>`**: The container for the entire animation. It automatically provides auto-scale (fit viewport), scrubber, and play / pause / loop controls.
- **`<Sprite start end>`**: A time segment. A Sprite is only visible from `start` to `end`. Inside it, use the `useSprite()` hook to read the local progress `t` (0 -> 1).
- **`useTime()`**: Reads the current global time in seconds.
- **`Easing.easeInOut` / `Easing.easeOut` / ...**: Easing functions.
- **`interpolate(t, from, to, easing?)`**: Interpolates values based on `t`.

This pattern borrows ideas from Remotion / After Effects, but stays lightweight and dependency-free.

## Getting Started

```html
<script type="text/babel" src="animations.jsx"></script>
<script type="text/babel">
  const { Stage, Sprite, useTime, useSprite, Easing, interpolate } = window.Animations;

  function Title() {
    const { t } = useSprite();  // Local progress 0 -> 1
    const opacity = interpolate(t, [0, 1], [0, 1], Easing.easeOut);
    const y = interpolate(t, [0, 1], [40, 0], Easing.easeOut);
    return (
      <h1 style={{ 
        opacity, 
        transform: `translateY(${y}px)`,
        fontSize: 120,
        fontWeight: 900,
      }}>
        Hello.
      </h1>
    );
  }

  function Scene() {
    return (
      <Stage duration={10}>  {/* 10-second animation */}
        <Sprite start={0} end={3}>
          <Title />
        </Sprite>
        <Sprite start={2} end={5}>
          <SubTitle />
        </Sprite>
        {/* ... */}
      </Stage>
    );
  }

  const root = ReactDOM.createRoot(document.getElementById('root'));
  root.render(<Scene />);
</script>
```

## Common Animation Patterns

### 1. Fade In / Fade Out

```jsx
function FadeIn({ children }) {
  const { t } = useSprite();
  const opacity = interpolate(t, [0, 0.3], [0, 1], Easing.easeOut);
  return <div style={{ opacity }}>{children}</div>;
}
```

**Note the range**: `[0, 0.3]` means the fade-in completes during the first 30% of the sprite duration, then keeps `opacity=1`.

### 2. Slide In

```jsx
function SlideIn({ children, from = 'left' }) {
  const { t } = useSprite();
  const progress = interpolate(t, [0, 0.4], [0, 1], Easing.easeOut);
  const offset = (1 - progress) * 100;
  const directions = {
    left: `translateX(-${offset}px)`,
    right: `translateX(${offset}px)`,
    top: `translateY(-${offset}px)`,
    bottom: `translateY(${offset}px)`,
  };
  return (
    <div style={{
      transform: directions[from],
      opacity: progress,
    }}>
      {children}
    </div>
  );
}
```

### 3. Character-by-Character Typewriter

```jsx
function Typewriter({ text }) {
  const { t } = useSprite();
  const charCount = Math.floor(text.length * Math.min(t * 2, 1));
  return <span>{text.slice(0, charCount)}</span>;
}
```

### 4. Number Count-up

```jsx
function CountUp({ from = 0, to = 100, duration = 0.6 }) {
  const { t } = useSprite();
  const progress = interpolate(t, [0, duration], [0, 1], Easing.easeOut);
  const value = Math.floor(from + (to - from) * progress);
  return <span>{value.toLocaleString()}</span>;
}
```

### 5. Segmented Explanation (Typical Teaching Animation)

```jsx
function Scene() {
  return (
    <Stage duration={20}>
      {/* Phase 1: Show the problem */}
      <Sprite start={0} end={4}>
        <Problem />
      </Sprite>

      {/* Phase 2: Show the approach */}
      <Sprite start={4} end={10}>
        <Approach />
      </Sprite>

      {/* Phase 3: Show the result */}
      <Sprite start={10} end={16}>
        <Result />
      </Sprite>

      {/* Caption shown throughout */}
      <Sprite start={0} end={20}>
        <Caption />
      </Sprite>
    </Stage>
  );
}
```

## Easing Functions

Preset easing curves:

| Easing | Character | Use For |
|--------|-----------|---------|
| `linear` | Constant speed | Scrolling captions, continuous animations |
| `easeIn` | Slow -> fast | Exits and disappearances |
| `easeOut` | Fast -> slow | Entrances and appearances |
| `easeInOut` | Slow -> fast -> slow | Position changes |
| **`expoOut`** ⭐ | **Exponential ease-out** | **Anthropic-level primary easing** (physical weight) |
| **`overshoot`** ⭐ | **Elastic rebound** | **Toggles / button pop-outs / emphasized interactions** |
| `spring` | Spring | Interaction feedback, geometry settling back into place |
| `anticipation` | Reverse first, then forward | Emphasized actions |

**Use `expoOut` as the default primary easing** (not `easeOut`) -- see `animation-best-practices.md` §2.
Use `expoOut` for entrances, `easeIn` for exits, and `overshoot` for toggles. These are the base rules for Anthropic-level animation.

## Rhythm and Duration Guide

### Micro-interactions (0.1-0.3s)
- Button hover
- Card expand
- Tooltip appearance

### UI Transitions (0.3-0.8s)
- Page changes
- Modal appearance
- List item insertion

### Narrative Animation (2-10s per segment)
- One phase of a concept explanation
- Data chart reveal
- Scene transition

### A Single Narrative Segment Should Not Exceed 10 Seconds

Human attention is limited. Use 10 seconds to explain one thing, then move to the next.

## How to Think When Designing Animation

### 1. Content / Story First, Animation Second

**Wrong**: Start by wanting a fancy animation, then stuff content into it.
**Right**: First decide what information you need to communicate, then use animation to serve that information.

Animation is a **signal**, not **decoration**. A fade-in emphasizes "this is important, look here." If everything fades in, the signal stops working.

### 2. Write the Timeline by Scene

```
0:00 - 0:03   Problem appears (fade in)
0:03 - 0:06   Problem enlarges / unfolds (zoom + pan)
0:06 - 0:09   Solution appears (slide in from right)
0:09 - 0:12   Solution explanation unfolds (typewriter)
0:12 - 0:15   Result demo (counter up + chart reveal)
0:15 - 0:18   One-sentence summary (static, read for 3 seconds)
0:18 - 0:20   CTA or fade out
```

Write the timeline first, then write the components.

### 3. Assets First

Prepare the images / icons / fonts used by the animation **before** building it. Do not stop halfway through to hunt for assets; it breaks the rhythm.

## Common Issues

**Animation stutters**
-> Usually caused by layout thrashing. Use `transform` and `opacity`; avoid animating `top` / `left` / `width` / `height` / `margin`. Browsers GPU-accelerate `transform`.

**Animation is too fast to follow**
-> Reading one Chinese character takes about 100-150ms; reading one word takes 300-500ms. If text is carrying the story, keep each sentence on screen for at least 3 seconds.

**Animation is too slow and viewers get bored**
-> Interesting visual changes need density. Static shots longer than 5 seconds feel dull.

**Multiple animations interfere with each other**
-> Use CSS `will-change: transform` to tell the browser ahead of time that the element will move, reducing reflow.

**Recording as video**
-> Use the skill's built-in toolchain (one command produces three formats): see `video-export.md`
- `scripts/render-video.js` -- HTML -> 25fps MP4 (Playwright + ffmpeg)
- `scripts/convert-formats.sh` -- 25fps MP4 -> 60fps MP4 + optimized GIF
- Need more precise frame rendering? Make `render(t)` a pure function; see item 5 in `animation-pitfalls.md`

## Working with Video Tools

This skill creates **HTML animations** that run in the browser. If the final output needs to be video material:

- **Short animation / concept demo**: Build the HTML animation with this method -> screen record it.
- **Long video / narrative**: This skill focuses on HTML animation. Use an AI video-generation skill or professional video software for long-form video.
- **Motion graphics**: Professional tools such as After Effects / Motion Canvas are more appropriate.

## About Libraries Such as Popmotion

If you truly need physical animation (spring, decay, keyframes with precise timing) and our engine is not enough, you can fall back to Popmotion:

```html
<script src="https://unpkg.com/popmotion@11.0.5/dist/popmotion.min.js"></script>
```

But **try our engine first**. It is enough for 90% of cases.
