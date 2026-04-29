# Audio Design Rules · huashu-design

> Audio application recipes for all animation demos. Use alongside `sfx-library.md` (the asset inventory).
> Battle-tested through huashu-design launch hero v1-v9 iterations, Gemini deep breakdowns of three official Anthropic videos, and 8000+ A/B comparisons.

---

## Core Principle · Dual-track Audio (Hard Rule)

Animation audio **must be designed as two independent layers**. Do not use only one layer:

| Layer | Role | Time Scale | Relationship to Visuals | Frequency Range |
|---|---|---|---|---|
| **SFX (beat layer)** | Marks every visual beat | Short, 0.2-2s | **Strong sync** (frame-aligned) | **High frequencies, 800Hz+** |
| **BGM (atmosphere bed)** | Emotional bed and sound field | Continuous, 20-60s | Weak sync (section-level) | **Low-mid frequencies, <4kHz** |

**An animation with only BGM is incomplete**. Viewers subconsciously feel that "the image is moving but the sound is not responding," which is the root of the cheap feeling.

---

## Gold Standard · Golden Ratios

These values are **hard engineering parameters** derived from testing the three official Anthropic videos and our own final v9 version. Use them directly:

### Volume
- **BGM volume**: `0.40-0.50` (relative to full scale 1.0)
- **SFX volume**: `1.00`
- **Loudness difference**: BGM peak should be **-6 to -8 dB lower** than SFX (SFX stands out through loudness separation, not absolute SFX loudness)
- **`amix` parameter**: `normalize=0` (never use `normalize=1`; it flattens dynamic range)

### Frequency Separation (P1 Hard Optimization)

Anthropic's secret is not "louder SFX"; it is **frequency layering**:

```bash
[bgm_raw]lowpass=f=4000[bgm]      # Limit BGM to low-mid frequencies below 4kHz
[sfx_raw]highpass=f=800[sfx]      # Push SFX into mid-high frequencies above 800Hz
[bgm][sfx]amix=inputs=2:duration=first:normalize=0[a]
```

Why: the human ear is most sensitive in the 2-5kHz range (the "presence" range). If SFX are all in this range and the BGM also covers the full spectrum, **SFX are masked by the high-frequency part of the BGM**. Highpass the SFX and lowpass the BGM so each owns a different part of the spectrum. SFX clarity improves immediately.

### Fade
- BGM in: `afade=in:st=0:d=0.3` (0.3s, avoids hard cuts)
- BGM out: `afade=out:st=N-1.5:d=1.5` (1.5s tail for a resolved ending)
- SFX already include envelopes and do not need additional fade

---

## SFX Cue Design Rules

### Density (SFX Count per 10 Seconds)

Testing across the three Anthropic videos shows three SFX density tiers:

| Video | SFX per 10s | Product Personality | Scene |
|---|---|---|---|
| Artifacts (ref-1) | **~9 per 10s** | Feature-dense, information-rich | Complex tool demo |
| Code Desktop (ref-2) | **0** | Pure atmosphere, meditative | Focused developer-tool state |
| Word (ref-3) | **~4 per 10s** | Balanced, office rhythm | Productivity tool |

**Heuristic**:
- Calm / focused product personality -> low SFX density (0-3 per 10s), BGM leads
- Lively / information-rich product personality -> high SFX density (6-9 per 10s), SFX drives the rhythm
- **Do not fill every visual beat**. Silence feels more premium than density. **Deleting 30-50% of cues makes the remaining ones more dramatic**.

### Cue Selection Priority

Not every visual beat needs SFX. Choose by this priority:

**P0 Required** (omitting it feels wrong):
- Typing (terminal / input)
- Click / selection (user decision moment)
- Focus switch (visual protagonist changes)
- Logo reveal (brand resolution)

**P1 Recommended**:
- Element entrance / exit (modal / card)
- Completion / success feedback
- AI generation start / end
- Major transition (scene switch)

**P2 Optional** (too many becomes noisy):
- Hover / focus-in
- Progress tick
- Decorative ambient

### Timestamp Alignment Precision

- **Same-frame alignment** (0ms error): click / focus switch / logo landing
- **Lead by 1-2 frames** (-33ms): fast whoosh (sets psychological expectation)
- **Lag by 1-2 frames** (+33ms): object landing / impact (matches real physics)

---

## BGM Selection Decision Tree

The huashu-design skill includes 6 BGM tracks (`assets/bgm-*.mp3`):

```
What is the animation's personality?
├─ Product launch / technical demo → bgm-tech.mp3 (minimal synth + piano)
├─ Tutorial explanation / tool usage → bgm-tutorial.mp3 (warm, instructional)
├─ Educational learning / principle explanation → bgm-educational.mp3 (curious, thoughtful)
├─ Marketing ad / brand promo → bgm-ad.mp3 (upbeat, promotional)
└─ Need a variant in the same style → bgm-*-alt.mp3 (alternate versions)
```

### No-BGM Scenarios (Worth Considering)

Reference Anthropic Code Desktop (ref-2): **0 SFX + pure lo-fi BGM** can also feel very refined.

**When to choose no BGM**:
- Animation duration <10s (BGM cannot establish itself)
- Product personality is "focused / meditative"
- The scene itself has ambient sound / narration
- SFX density is very high (avoid auditory overload)

---

## Scene Recipes (Ready to Use)

### Recipe A · Product Launch Hero (Same Pattern as huashu-design v9)
```
Duration: 25 seconds
BGM: bgm-tech.mp3 · 45% · frequency range <4kHz
SFX density: ~6 per 10s

cues:
  terminal typing → type × 4 (0.6s apart)
  enter           → enter
  card converge   → card × 4 (staggered by 0.2s)
  selected        → click
  Ripple          → whoosh
  4 focus shifts  → focus × 4
  Logo            → thud (1.5s)

Volume: BGM 0.45 / SFX 1.0 · amix normalize=0
```

### Recipe B · Tool Feature Demo (Reference: Anthropic Code Desktop)
```
Duration: 30-45 seconds
BGM: bgm-tutorial.mp3 · 50%
SFX density: 0-2 per 10s (very sparse)

Strategy: Let BGM + explanatory voiceover drive the piece. Use SFX only at decisive moments
(file saved / command execution complete).
```

### Recipe C · AI Generation Demo
```
Duration: 15-20 seconds
BGM: bgm-tech.mp3 or no BGM
SFX density: ~8 per 10s (high density)

cues:
  user input → type + enter
  AI starts processing → magic/ai-process (1.2s loop)
  generation complete → feedback/complete-done
  result appears → magic/sparkle
  
Highlight: ai-process can loop 2-3 times through the full generation process.
```

### Recipe D · Pure Atmospheric Long Take (Reference: Artifacts)
```
Duration: 10-15 seconds
BGM: none
SFX: use 3-5 carefully designed cues on their own

Strategy: Every SFX is the protagonist, with no BGM "smearing" them together.
Good for: slow single-product shots, close-up showcases.
```

---

## ffmpeg Composition Templates

### Template 1 · Overlay a Single SFX onto Video
```bash
ffmpeg -y -i video.mp4 -itsoffset 2.5 -i sfx.mp3 \
  -filter_complex "[0:a][1:a]amix=inputs=2:normalize=0[a]" \
  -map 0:v -map "[a]" output.mp4
```

### Template 2 · Multi-SFX Timeline Composition (Aligned by Cue Time)
```bash
ffmpeg -y \
  -i sfx-type.mp3 -i sfx-enter.mp3 -i sfx-click.mp3 -i sfx-thud.mp3 \
  -filter_complex "\
[0:a]adelay=1100|1100[a0];\
[1:a]adelay=3200|3200[a1];\
[2:a]adelay=7000|7000[a2];\
[3:a]adelay=21800|21800[a3];\
[a0][a1][a2][a3]amix=inputs=4:duration=longest:normalize=0[mixed]" \
  -map "[mixed]" -t 25 sfx-track.mp3
```
**Key parameters**:
- `adelay=N|N`: the first value is left-channel delay in ms, the second is right-channel delay; write both to ensure stereo alignment
- `normalize=0`: preserves dynamic range; critical
- `-t 25`: truncates to the specified duration

### Template 3 · Video + SFX Track + BGM (With Frequency Separation)
```bash
ffmpeg -y -i video.mp4 -i sfx-track.mp3 -i bgm.mp3 \
  -filter_complex "\
[2:a]atrim=0:25,afade=in:st=0:d=0.3,afade=out:st=23.5:d=1.5,\
     lowpass=f=4000,volume=0.45[bgm];\
[1:a]highpass=f=800,volume=1.0[sfx];\
[bgm][sfx]amix=inputs=2:duration=first:normalize=0[a]" \
  -map 0:v -map "[a]" -c:v copy -c:a aac -b:a 192k final.mp4
```

---

## Failure Mode Quick Reference

| Symptom | Root Cause | Fix |
|---|---|---|
| SFX cannot be heard | High-frequency BGM masks it | Add `lowpass=f=4000` to BGM + `highpass=f=800` to SFX |
| Sound effects are too loud / harsh | Absolute SFX volume is too high | Lower SFX volume to 0.7, also lower BGM to 0.3, preserving the gap |
| BGM and SFX rhythms conflict | Wrong BGM choice (used music with a strong beat) | Switch to ambient / minimal synth BGM |
| BGM cuts off abruptly at the end | No fade out | `afade=out:st=N-1.5:d=1.5` |
| SFX overlap into a smear | Cues are too dense + each SFX is too long | Keep SFX under 0.5s and cue spacing >= 0.2s |
| WeChat article MP4 has no sound | WeChat articles sometimes mute autoplay | Do not worry; sound plays when users open it. GIFs are silent by nature |

---

## Linking Audio and Visuals (Advanced)

### SFX Timbre Should Match the Visual Style

- Warm beige / paper-like visuals -> use **wooden / soft** SFX timbres (Morse, paper snap, soft click)
- Cold dark tech visuals -> use **metallic / digital** SFX timbres (beep, pulse, glitch)
- Hand-drawn / playful visuals -> use **cartoon / exaggerated** SFX timbres (boing, pop, zap)

The warm beige background in our current `apple-gallery-showcase.md` pairs with `keyboard/type.mp3` (mechanical) + `container/card-snap.mp3` (soft) + `impact/logo-reveal-v2.mp3` (cinematic bass).

### SFX Can Guide Visual Rhythm

Advanced technique: **design the SFX timeline first, then adjust the visual animation to align with the SFX** (not the other way around).
Every SFX cue is a "clock tick." When the visuals adapt to the SFX rhythm, the result feels very stable. When SFX chase the visuals, even a ±1-frame mismatch often feels wrong.

---

## Quality Checklist (Pre-release Self-check)

- [ ] Loudness difference: SFX peak - BGM peak = -6 to -8 dB?
- [ ] Frequency ranges: BGM lowpass 4kHz + SFX highpass 800Hz?
- [ ] `amix normalize=0` (preserve dynamic range)?
- [ ] BGM fade-in 0.3s + fade-out 1.5s?
- [ ] Is the SFX count appropriate (choose density by scene personality)?
- [ ] Is every SFX aligned to the visual beat on the same frame (within ±1 frame)?
- [ ] Is the logo reveal sound effect long enough (recommended 1.5s)?
- [ ] Listen once with BGM muted: does SFX alone have enough rhythm?
- [ ] Listen once with SFX muted: does BGM alone have emotional movement?

Each layer should work on its own. If it only sounds good when the two layers are combined, the design is not done.

---

## References

- SFX asset inventory: `sfx-library.md`
- Visual style reference: `apple-gallery-showcase.md`
- Deep audio analysis of the three Anthropic videos: `/Users/alchain/Documents/writing/01-wechat-writing/projects/2026.04-huashu-design-release/reference-animations/AUDIO-BEST-PRACTICES.md`
- huashu-design v9 real-world case: `/Users/alchain/Documents/writing/01-wechat-writing/projects/2026.04-huashu-design-release/images/hero-animation-v9-final.mp4`
