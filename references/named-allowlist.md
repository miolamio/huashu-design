# Named Allowlist — Specific Recommendations by Archetype

> The anti-AI-slop section in `SKILL.md` is a **blocklist** ("don't use Inter / purple gradients / SVG illustration"). This file is the matching **allowlist**: when you commit to a tone, here are the specific fonts, color directions, and layout grids that work, named explicitly.
>
> **Why named?** Anthropic's `frontend-design` skill ships verbatim banned fonts ("Inter, Roboto, Arial, Open Sans, Lato, system fonts, Space Grotesk") AND verbatim recommended fonts by archetype. The 2025-2026 anti-slop literature converges on the same finding: **vague guidance ("be distinctive") fails because models converge to the average; specific names succeed because they remove the most-common attractor states.**
>
> **Use after** the design POV is set (per SKILL.md §POV-First Gate) or after Design Direction Advisor recommends a school. **This file is for execution, not direction-setting** — direction-setting lives in `references/design-styles.md` (the 20-style library).

## Contents

- [How to Use This File](#how-to-use-this-file)
- [Archetype 1 · Editorial / Magazine / Longform](#archetype-1--editorial--magazine--longform)
- [Archetype 2 · Startup / Modern SaaS](#archetype-2--startup--modern-saas)
- [Archetype 3 · Technical / Developer Tools](#archetype-3--technical--developer-tools)
- [Archetype 4 · Code / Terminal UI / Mono-Forward](#archetype-4--code--terminal-ui--mono-forward)
- [Archetype 5 · Industrial / Brutalist / Raw](#archetype-5--industrial--brutalist--raw)
- [Archetype 6 · Eastern Minimal (Hara / Takram)](#archetype-6--eastern-minimal-hara--takram)
- [Archetype 7 · Playful / Memphis / Joyful](#archetype-7--playful--memphis--joyful)
- [Cross-Archetype Patterns](#cross-archetype-patterns)
- [Font Licensing Notes](#font-licensing-notes)

---

## How to Use This File

1. The agent has committed to a POV or a design direction (per SKILL.md §POV-First Gate or Design Direction Advisor).
2. Pick the archetype that matches. **Do not mix archetypes** — Editorial + Brutalist is not a thing.
3. Use the named fonts, color directions, and layout grids verbatim. Do not invent.
4. If the brand spec from §1.a Core Asset Protocol overrides any item here, the brand spec wins. The allowlist is a *fallback* when no brand exists.

---

## Archetype 1 · Editorial / Magazine / Longform

**When**: book-style content, op-eds, longform essays, brand journals, museum sites, gallery work.

| Dimension | Recommended (named) |
|---|---|
| **Display** | Fraunces · Crimson Pro · Playfair Display · Cormorant Garamond · GT Sectra |
| **Body** | Source Serif 4 · Crimson Pro · Lora · Newsreader |
| **Mono (rare; for captions/datelines)** | JetBrains Mono Light |
| **Color direction** | Warm cream (`#FAF7F0`, `#F5EFE6`) ground + ink-black (`#1A1A1A`) text + one rust/oxblood accent (`oklch(0.45 0.18 30)`) |
| **Layout grid** | Asymmetric 2-column or 3-column with deep leading (`line-height: 1.7-1.8`); generous outer margins (≥10% of viewport); pull-quotes break the grid |
| **Signature details** | `text-wrap: balance` on H1; `hanging-punctuation: first`; drop-cap on opening paragraph (`::first-letter`) |

**Don't pair with**: Inter body. Sans-only stacks. Rainbow gradients.

## Archetype 2 · Startup / Modern SaaS

**When**: marketing landing page, product launch site, pricing page, B2B SaaS dashboard wrapper.

| Dimension | Recommended (named) |
|---|---|
| **Display** | Clash Display · Satoshi · Cabinet Grotesk · Chillax · Bricolage Grotesque |
| **Body** | Geist Sans · Switzer · General Sans · Inter Tight (note: prefer Switzer over Inter; if user demands Inter, use Inter Tight not Inter) |
| **Mono** | Geist Mono · JetBrains Mono · Berkeley Mono (paid; flag) |
| **Color direction** | Dark canvas (`#0A0A0A` to `#111111`) + restrained single accent (`oklch(0.65 0.15 240)` cool electric blue OR `oklch(0.7 0.18 145)` muted electric green) + 3-4 grays at `oklch(0.18..0.78)` |
| **Layout grid** | 12-column with thin `0.5px` rules (Vercel-style), large hero with `clamp()` type sizes (`clamp(48px, 8vw, 128px)`), feature sections that escape the grid |
| **Signature details** | Subtle dot-grid or line-grid background; `backdrop-filter: blur(20px)` on sticky nav; `text-spacing-trim: space-all` for tight Asian-aware kerning |

**Don't pair with**: serif-display. Bento grids with uniform shadows. Centered hero with floating cards.

## Archetype 3 · Technical / Developer Tools

**When**: API docs, CLI documentation, internal tools, monitoring dashboards, devtool marketing.

| Dimension | Recommended (named) |
|---|---|
| **Display** | IBM Plex Sans Condensed · Söhne (paid; flag) · Inter Tight (acceptable here only) |
| **Body** | IBM Plex Sans · Source Sans 3 · Public Sans |
| **Mono (load-bearing)** | IBM Plex Mono · JetBrains Mono · Fira Code (with ligatures) · Cascadia Code |
| **Color direction** | Solarized-derived (`#002b36` / `#073642` ground; `#268bd2` / `#cb4b16` accents) OR muted neutral (`#0F1419` ground, `#5A6470` ink, `#FF9C00` warning, `#3FB950` success) |
| **Layout grid** | Sidebar + main with monospaced gutter; tables as first-class citizens (not styled-up divs); inline code rendered with `<code>` not `<span>` |
| **Signature details** | Real syntax-highlighted code blocks (Shiki / highlight.js); CLI session demos with `▮` cursor; data-density acceptance — don't pad with whitespace if the user is technical |

**Don't pair with**: emoji icons. Glassmorphism. Marketing-speak microcopy.

## Archetype 4 · Code / Terminal UI / Mono-Forward

**When**: terminal mockups, code-editor screenshots, AI-coding-tool marketing, programming tutorials.

| Dimension | Recommended (named) |
|---|---|
| **Display** | JetBrains Mono Bold · Berkeley Mono Bold (paid; flag) · IBM Plex Mono Bold |
| **Body** | JetBrains Mono · IBM Plex Mono · Fira Code |
| **Mono is the body** | (Same as above) |
| **Color direction** | Editor theme palettes used coherently: Dracula (`#282a36` / `#bd93f9`), Catppuccin Mocha (`#1e1e2e` / `#cba6f7`), GitHub Dark Default (`#0d1117` / `#58a6ff`). Pick one theme; do not blend. |
| **Layout grid** | Single-column or sidebar+main with hard `0px` border-radius (no rounded corners); fixed-width 80-char measure; gutter for line numbers |
| **Signature details** | Cursor blink animation (`opacity: 0 / 1` keyframes at `1s steps(1)`); typed-in-real-time animation; shell-prompt prefix (`$` or `>`) with semantic color |

**Don't pair with**: serif anywhere. Soft pastels. Gradients (except subtle vignette). Lucide icons (use Tabler or feather instead, less rounded).

## Archetype 5 · Industrial / Brutalist / Raw

**When**: art-school portfolios, music labels, design studio sites, architecture firm pages, manifesto pages.

| Dimension | Recommended (named) |
|---|---|
| **Display** | Druk Wide / Druk Cond (paid; flag) · Bebas Neue · Anton · PP Neue Machina (paid; flag) |
| **Body** | Helvetica Neue (system; permitted in this archetype only) · Arial (system; permitted in this archetype only) · Akzidenz-Grotesk (paid; flag) — note: yes these are normally banned, but Brutalism is the one place they read as intentional |
| **Mono** | Courier · Courier Prime |
| **Color direction** | Pure black (`#000000`), pure white (`#FFFFFF`), one screaming accent (Klein blue `#002FA7`, Yves Saint Laurent yellow `#F1B400`, hot pink `#FF1493`). No grays. |
| **Layout grid** | Broken grid (intentionally misaligned), oversized type that breaks the viewport (`clamp(120px, 18vw, 320px)`), text on top of images without overlay opacity |
| **Signature details** | Visible grid (CSS Grid with `outline: 1px solid red` left in production deliberately); raw `<details>` elements not styled; hard line-height (`1.0` or `0.9` for display) |

**Don't pair with**: rounded corners. Drop shadows. Smooth easing curves on motion (use `cubic-bezier(0.85, 0, 0.15, 1)` step-like ease).

## Archetype 6 · Eastern Minimal (Hara / Takram)

**When**: Japanese / East Asian product brands, ceremony / tea / paper / craft products, contemplative content, "ma" (negative space)-driven work.

| Dimension | Recommended (named) |
|---|---|
| **Display (Latin)** | Cormorant Garamond Light · Source Serif 4 Light · Newsreader Light |
| **Display (CJK)** | Noto Serif JP / KR / SC · Source Han Serif · LXGW WenKai (free, beautiful for SC) · Tsukushi Mincho (paid; flag) |
| **Body (CJK)** | Noto Sans JP / KR / SC · LXGW WenKai for hand-written warmth |
| **Color direction** | Off-white paper (`#F8F6F0`, `#FAF7F2`) + sumi-ink (`#1F1A17`) + one earth accent (indigo `#1E3A5F`, persimmon `#D2691E`, moss `#5A7247`). NEVER pure white. |
| **Layout grid** | 60-70% whitespace by visual weight; one dominant element per screen; vertical rhythm with `:lang(ja)` aware line-height (`1.8-2.0`); `text-spacing-trim: space-all` for proper CJK punctuation |
| **Signature details** | Hairline rules (`0.5px solid rgba(31, 26, 23, 0.15)`); seal-script red square stamp accent; horizontal-vertical layout switching; `writing-mode: vertical-rl` for poetic captions |

**Don't pair with**: SaaS dark mode. Lucide icons. CTA buttons (replace with quiet underlines).

## Archetype 7 · Playful / Memphis / Joyful

**When**: kids' products, art toys, ice-cream brands, festivals, children's books, anything Sagmeister-influenced.

| Dimension | Recommended (named) |
|---|---|
| **Display** | Boldonse · Mona Sans · Fraunces (rounded warmth via the SOFT axis) · Recoleta (paid; flag) · Sentinel (paid; flag) · GT Maru (paid; flag) |
| **Body** | DM Sans · Mulish · Manrope (these are normally banned-adjacent — only here because the playful archetype rewards their friendliness) |
| **Mono** | Cartograph CF (paid) · DM Mono |
| **Color direction** | Memphis palette: pastel pink `oklch(0.85 0.08 10)` + cyan `oklch(0.78 0.10 200)` + yellow `oklch(0.90 0.12 90)` + black `#000`. 4-5 colors used at high saturation. |
| **Layout grid** | Asymmetric, with intentional "wrong" placements; geometric decorative shapes (squiggle, dot grid, triangle, half-circle) layered behind type; rotate select elements 2-7° |
| **Signature details** | Hand-drawn squiggles via SVG; shadow as offset solid color (not blur), e.g., `box-shadow: 8px 8px 0 #000`; Comic Neue or Comic Mono for explicitly-childlike contexts only |

**Don't pair with**: corporate stock photography. Centered layouts. Glassmorphism. Restraint.

---

## Cross-Archetype Patterns

These work across multiple archetypes when used with care:

- **Color spaces**: prefer `oklch()` over `hsl()`. Hue stable across lightness changes. Define `--primary`, `--primary-light`, `--primary-dark` as same hue, varying lightness.
- **Typography**: pair display + body where contrast is **structural, not weight-based**. Display Serif + body sans (Editorial), Display Sans + body Mono (Technical), Display Mono + body Mono (Code). Avoid 400-vs-600 weight pairs — the contrast is too quiet.
- **Spacing**: use a single base unit (4px or 8px) and stack at multiples (4, 8, 12, 16, 24, 32, 48, 64, 96, 128). Don't invent 18px or 22px gaps.
- **Motion**: `cubic-bezier(0.34, 1.56, 0.64, 1)` for spring-like overshoot in playful contexts. `cubic-bezier(0.4, 0, 0.2, 1)` for technical / SaaS. `cubic-bezier(0.85, 0, 0.15, 1)` for brutalist / step-like.

## Font Licensing Notes

All fonts in this allowlist marked without "(paid; flag)" are **freely usable for commercial work**:

- **Google Fonts** (SIL OFL): Fraunces, Crimson Pro, Playfair Display, Cormorant Garamond, Source Serif 4, Lora, Newsreader, JetBrains Mono, Geist Sans, Geist Mono, IBM Plex (all weights), Source Sans 3, Public Sans, Bebas Neue, Anton, DM Sans, Mulish, Manrope, DM Mono, Comic Neue, Noto Serif JP/KR/SC, Noto Sans JP/KR/SC, Source Han Serif.
- **Fontshare** (free for commercial use): Switzer, General Sans, Clash Display, Satoshi, Chillax, Boldonse, Mona Sans. **Cabinet Grotesk** (Fontshare; ITF Free Font License). **Bricolage Grotesque** is on Google Fonts (SIL OFL).
- **Other free**: LXGW WenKai (open source, beautiful Simplified Chinese serif).
- **Paid (always flag the user before recommending)**: Söhne (Klim), GT Sectra (Grilli), GT Maru (Grilli), Druk family (Commercial Type), PP Neue Machina (Pangram Pangram premium tier), Akzidenz-Grotesk (Berthold), Berkeley Mono (US Graphics), Tsukushi Mincho (Fontworks), Sentinel (H&Co), Cartograph CF (Connary Fagen), **Recoleta (Latinotype — the freely circulating version is personal-use demo only; commercial license required)**.

When the brand-spec.md from §1.a Core Asset Protocol specifies a paid font that the user already owns, that wins regardless of this allowlist. When proposing a paid font for a new design, **always tell the user the font is paid and offer a free substitute** from the same archetype.
