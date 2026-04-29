# Cinematic Patterns · Best Practices for Workflow Demos

> Five key patterns for upgrading from "PPT animation" to "launch-event-grade cinematic."
> Distilled from the two cinematic demos in the 2026-04 "Talking About Skills" deck: Nuwa workflow + Darwin workflow. Field-tested and reproducible.

---

## 0 · What This Document Solves

When you need to create a "workflow demo animation" (typical scenarios: skill workflow, product onboarding, API call flow, agent task execution), there are two common approaches:

| Paradigm | What It Looks Like | Result |
|---|---|---|
| **PPT animation** (bad) | step 1 fade in → step 2 fade in → step 3 fade in, with 4 boxes arranged on screen together | The audience feels "this is just a PPT with fade effects," with no wow moment |
| **Cinematic** (good) | Scene-based; focus on only one thing at a time; scenes transition with dissolve / focus pull / morph | The audience feels "this is a product-launch segment" and wants to screenshot/share it |

The root difference is **not animation technique**; it is **narrative paradigm**. This document explains how to upgrade from the former to the latter.

---

## 1 · Five Core Patterns

### Pattern A · Dashboard + Cinematic Overlay Two-Layer Structure

**Problem**: a purely cinematic page defaults to a black screen plus a ▶ button. If the user reaches this page and does not click, there is nothing to see.

**Solution**:
```
DEFAULT state (always visible): complete static workflow dashboard
  └── Audience immediately understands how this skill / workflow runs

POINT ▶ trigger (overlay rises): 22-second cinematic
  └── Automatically fades back to DEFAULT after finishing

```

**Implementation points**:
- `.dash` is visible by default; `.cinema` defaults to `opacity: 0; pointer-events: none`
- `.play-cta` is a small gold button in the lower-right, not a large central cover
- Click → `cinema.classList.add('show')` + `dash.classList.add('hide')`
- Run once with `requestAnimationFrame`, not a loop; after the end, `endCinematic()` reverses the state

**Anti-pattern**: default state = large central ▶ overlay covering everything, leaving the page blank before it is clicked.

---

### Pattern B · Scene-Based, NOT Step-Based

**Problem**: splitting an animation into "step 1 appears → step 2 appears → ..." is PPT thinking.

**Solution**: split it into 5 scenes. Each scene is an **independent shot**, and the full screen focuses on only one thing:

| Scene Type | Role | Duration |
|---|---|---|
| 1 · Invoke | User input trigger, such as terminal typewriter | 3-4s |
| 2 · Process | Visualization of the core workflow, with a unique visual language | 5-6s |
| 3 · Result/Insight | Key extracted artifact, visualized | 4-5s |
| 4 · Output | Real output display: file / diff / numbers | 3-4s |
| 5 · Hero Reveal | Closing hero moment: large type + value proposition | 4-5s |

**Total duration ≈ 22 seconds**. This is the tested golden length:
- Shorter than 18 seconds: the PM has not entered the scene before it ends
- Longer than 25 seconds: attention fades
- 22 seconds is enough to hook → unfold → resolve → leave an impression

**Implementation points**:
- Use a global timeline such as `T = { DURATION: 22.0, s1_in: [0, 0.7], s2_in: [3.8, 4.6], ... }`
- A single `requestAnimationFrame(render)` calculates opacity / transform for all scenes
- Do not use chained setTimeout; it is fragile and hard to debug
- Easing must use `expoOut` / `easeOut` / cubic-bezier; **linear is forbidden**

---

### Pattern C · Each Demo Must Have Its Own Visual Language

**Problem**: after finishing the first cinematic, the second one lazily reuses the same template: same orbit + pentagon + typewriter + hero large type, with only the copy changed.

**Result**: the audience notices that two skills "look exactly the same," which implies the two skills are not meaningfully different.

**Solution**: each workflow has a different core metaphor, so its visual language must also differ.

**Case comparison**:

| Dimension | Nuwa (persona distillation) | Darwin (skill optimization) |
|---|---|---|
| Core metaphor | Collect → distill → write | Loop → evaluate → ratchet |
| Visual motion | Floating / radiating / pentagon | Looping / rising / comparison |
| Scene 2 | 3D Orbit · 8 archive cards floating on a perspective ellipse | Spin Loop · tokens run 5 laps around a 6-node ring |
| Scene 3 | Pentagon · 5 tokens radiate from center | v1 vs v5 · side-by-side diff, red version vs gold version |
| Scene 4 | SKILL.md typewriter | Hill-Climb · full-screen curve drawing |
| Scene 5 hero | "21 minutes" in large serif italic | Rotating gear ⚙ + gold "KEPT +1.1" tag |

**Test**: cover the copy and look only at the visuals. Can you tell which demo it is? If not, the design is lazy.

---

### Pattern D · Use Real AI-Generated Assets, Not Emoji or Hand-Drawn SVG

**Problem**: a 3D orbit / gallery needs asset fragments floating around. Emoji such as 📚🎤 are ugly and unbranded, while hand-drawn SVG book spines never look like real books.

**Solution**: use `huashu-gpt-image` to generate one 4×2 grid image: 8 theme-relevant objects, white background, 60px breathing space, unified style. Then use `extract_grid.py --mode bbox` to cut it into 8 independent transparent PNGs.

**Prompt points** (for detailed prompt patterns, see the `huashu-gpt-image` skill):
- IP anchor, such as "1960s Caltech archive aesthetic" / "Hearthstone-style consistent treatment"
- White background, because it is easier to cut out; gray backgrounds look atmospheric but make transparent extraction difficult
- Use 4×2, not 5×5, to avoid last-row compression bugs
- Persona finishing, such as "You are a Wired magazine curator preparing an exhibition photo"

**Anti-pattern**: using emoji as icons, or replacing product images with CSS silhouettes.

---

### Pattern E · BGM + SFX Dual-Track System

**Problem**: animation without sound subconsciously feels like a poor demo.

**Solution**: long-form BGM plus 11 SFX cues.

**General SFX cue recipe** (suitable for workflow demos):

| Time | SFX | Trigger Scene |
|---|---|---|
| 0.10s | whoosh | Terminal rises from below |
| 3.0s | enter | Typewriter completes; enter is pressed |
| 4.0s | slide-in | Scene 2 elements enter |
| 5-9s × 5 times | sparkle | Key process nodes, such as each generation / token / data point |
| 14s | click | Switch to output scene |
| 17.8s | logo-reveal | Hero reveal moment |
| typewriter | type | Trigger once every 2 characters; keep density controlled |

**Frequency separation**: BGM volume 0.32 for low-frequency bed, SFX volume 0.55 for mid/high punch, sparkle 0.7 for salience, logo-reveal 0.85 for the strongest hero moment.

**User controls**:
- Must include a ▶ start overlay because browsers restrict autoplay
- Add a small mute button in the upper-right so the user can switch sound off at any time
- Do not force audio to play just because the user reached the slide

---

## 2 · Static Dashboard Design Points

The dashboard is Layer 1 of the two-layer structure. A PM should understand the skill even without clicking ▶.

**Layout**: 3-column grid, or 1 large + 2 small panels. Each panel answers a different question:

| Panel Type | Question It Answers | Example |
|---|---|---|
| **Pipeline / Flow Diagram** | "What is this skill's workflow?" | Nuwa 4-stage pipeline · Darwin autoresearch loop |
| **Snapshot / State** | "What does the real output data look like?" | Darwin 8-dimensional rubric snapshot |
| **Trajectory / Evolution** | "How does it change after multiple runs?" | Darwin 5-generation hill-climb curve |
| **Examples / Gallery** | "What has it already produced?" | Nuwa 21 personas gallery |
| **Strip · Example I/O** | "What input becomes what output?" | Nuwa example strip: `› nuwa distill Feynman → feynman.skill (21 min)` |

**Key constraints**:
- Information density must be sufficient; every panel should carry differentiated information
- But do not stuff it with data slop; every number must be meaningful
- The palette must match the cinematic layer so the transition is not jarring

---

## 3 · Debugging and Development Tools

Every long animation must include these three dev tools; otherwise debugging becomes painful.

### Tool 1 · `?seek=N` Freeze at Second N

```js
const seek = parseFloat(params.get('seek'));
if (!isNaN(seek)) {
  started = true; muted = true;
  frozenT = seek;  // render() uses this t instead of elapsed
  cinema.classList.add('show'); dash.classList.add('hide');
}

// inside render():
let t = frozenT !== null ? frozenT : (elapsed % T.DURATION);
```

Usage: `http://.../slide.html?seek=12` directly shows the frame at second 12, with no waiting.

### Tool 2 · `?autoplay=1` Skip the ▶ Overlay

Useful for automated Playwright screenshot tests and for forced startup when embedded in an iframe.

### Tool 3 · Manual REPLAY Button

A small upper-right button lets users and developers replay as many times as needed. CSS:

```css
.replay{position:absolute;top:18px;right:18px;background:rgba(212,165,116,0.1);
  border:1px solid rgba(212,165,116,0.3);color:#D4A574;
  font-family:monospace;font-size:10px;letter-spacing:.28em;text-transform:uppercase;
  padding:6px 12px;border-radius:1px;cursor:pointer;backdrop-filter:blur(6px);z-index:6}
```

---

## 4 · iframe Embedding Pitfalls (If the Cinematic Is Embedded in a Deck)

### Pitfall 1 · Parent Window Click Zones Intercept Buttons Inside iframe

If deck `index.html` adds transparent left/right 22vw click zones for slide navigation, those zones can **cover the ▶ play button inside the iframe**. The user's click is swallowed as "next slide."

**Fix**: add `top: 12vh; bottom: 25vh` to the click zones, leaving the top and bottom 25% unblocked so the iframe's central ▶ and lower-right ▶ remain clickable.

### Pitfall 2 · Keyboard Events Are Lost After iframe Takes Focus

After the user clicks inside the iframe, focus stays inside the iframe, and the parent window no longer receives ←/→ keyboard events.

**Fix**:
```js
iframe.addEventListener('load', () => {
  // Inject a keyboard forwarder.
  const doc = iframe.contentDocument;
  doc.addEventListener('keydown', (e) => {
    window.dispatchEvent(new KeyboardEvent('keydown', { key: e.key, ... }));
  });
  // Pull focus back to the parent window after clicks.
  doc.addEventListener('click', () => setTimeout(() => window.focus(), 0));
});
```

### Pitfall 3 · Behavior Differences Between file:// and https://

A cinematic that works locally under file:// may break after deployment because:
- Under file://, iframe contentDocument is same-origin
- Under https://, it is also same-origin if it shares the same host, but audio autoplay restrictions are stricter

**Fix**:
- Before deployment, start local HTTP with `python3 -m http.server` and test again
- BGM must wait until the user clicks ▶ before calling `bgm.play()`; do not play on page load

---

## 5 · Anti-Pattern Quick Reference

| ❌ Anti-Pattern | ✅ Correct Pattern |
|---|---|
| Default = black-screen ▶ overlay | Default = static dashboard, with ▶ as secondary action |
| 4 steps side by side fading in | 5 full-screen scenes; each scene focuses on one thing |
| Reuse template and swap copy for different demos | Independent visual language for each demo; distinguishable with copy covered |
| Emoji / hand-drawn SVG as assets | gpt-image-2 large image + extract_grid cutouts |
| No BGM or SFX | BGM + 11 SFX cues as a dual-track system |
| Schedule with chained setTimeout | requestAnimationFrame + global timeline object T |
| Linear animation | Expo / cubic-bezier easing |
| No dev tools | `?seek=N` + `?autoplay=1` + REPLAY button |
| Parent click zone swallows iframe buttons | Add top/bottom margins to click zones so buttons remain accessible |

---

## 6 · Time Budget

Using this pattern, one complete cinematic demo, including dashboard, takes:

| Task | Time |
|---|---|
| Design 5-scene narrative + visual language | 30 minutes; think carefully because this determines distinctiveness |
| Static dashboard layout + content | 1 hour |
| Implement 5 cinematic scenes | 1.5 hours |
| Time audio cues + replay button | 30 minutes |
| Validate 5 key moments with Playwright screenshots | 15 minutes |
| **Single demo total** | **3-4 hours** |

The second demo can reuse the framework, but **its visual language must still be independent**. Expected time: about 2-3 hours.
