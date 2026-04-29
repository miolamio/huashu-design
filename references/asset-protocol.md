# Core Asset Protocol — Execution Detail

> Detailed execution procedures for §1.a Core Asset Protocol in `SKILL.md`.
> The 5-step ritual, the "Assets Beat Specs" priority order, the 5-10-2-8 quality gate, the "If The Protocol Fails" table, and the "Failure Cases To Remember" list are all in `SKILL.md` — those are the load-bearing rules. **This file is the implementation handbook**: the exact prompts, search paths, download commands, verification grep, and the `brand-spec.md` template.

## Contents

- [Step 1 · Ask For The Full Asset List](#step-1--ask-for-the-full-asset-list) — verbatim prompt template to send the user
- [Step 2 · Search Official Channels By Asset Type](#step-2--search-official-channels-by-asset-type) — search-path table + WebSearch keywords
- [Step 3 · Download Assets With Fallback Paths](#step-3--download-assets-with-fallback-paths) — logo / product image / UI screenshot procedures
- [Step 3.4 — 5-10-2-8 Quality Gate Recording](#step-34--5-10-2-8-quality-gate-recording) — the five quality dimensions to score per candidate
- [Step 4 · Verify And Extract](#step-4--verify-and-extract) — per-asset verification table, color-extraction grep, demo-brand contamination warning
- [Step 5 · Freeze Assets In `brand-spec.md`](#step-5--freeze-assets-in-brand-specmd) — full markdown template + post-spec rules

---

## Step 1 · Ask For The Full Asset List

Do not merely ask "Do you have brand guidelines?" Ask item by item:

```text
For <brand/product>, do you have any of the following? Listed by priority:
1. Logo (SVG / high-res PNG) — required for any brand
2. Product photos / official renders — required for physical products
3. UI screenshots / interface assets — required for digital products
4. Color values (HEX / RGB / brand palette)
5. Font list (display / body)
6. Brand guidelines PDF / Figma design system / brand website link

Send what you have. I will search, capture, or generate what is missing.
```

## Step 2 · Search Official Channels By Asset Type

| Asset | Search Path |
|---|---|
| **Logo** | `<brand>.com/brand`, `<brand>.com/press`, `<brand>.com/press-kit`, `brand.<brand>.com`, inline SVG in the homepage header |
| **Product photo / render** | Product page hero image and gallery, official launch film frames, official press images |
| **UI screenshot** | App Store / Google Play screenshots, website screenshots section, official demo video frames |
| **Colors** | Inline CSS, Tailwind config, brand guidelines PDF |
| **Fonts** | `<link rel="stylesheet">`, Google Fonts traces, brand guidelines |

Fallback `WebSearch` keywords:

- Logo: `<brand> logo download SVG`, `<brand> press kit`
- Product image: `<brand> <product> official renders`, `<brand> <product> product photography`
- UI: `<brand> app screenshots`, `<brand> dashboard UI`

## Step 3 · Download Assets With Fallback Paths

### 3.1 Logo, required for any brand

1. Prefer standalone SVG/PNG files:

   ```bash
   curl -o assets/<brand>-brand/logo.svg https://<brand>.com/logo.svg
   curl -o assets/<brand>-brand/logo-white.svg https://<brand>.com/logo-white.svg
   ```

2. Extract inline SVG from official HTML:

   ```bash
   curl -A "Mozilla/5.0" -L https://<brand>.com -o assets/<brand>-brand/homepage.html
   # Then grep/extract the <svg>...</svg> logo node.
   ```

3. Last resort: official social avatar from GitHub, X/Twitter, or LinkedIn.

### 3.2 Product photos / renders, required for physical products

Priority order:

1. Official product-page hero image, usually 2000px+.
2. Official press kit.
3. Official launch video frames via `yt-dlp` + `ffmpeg`.
4. Wikimedia Commons.
5. AI generation fallback using nano-banana-pro with the real product image as reference. Do not hand-draw with CSS/SVG.

```bash
curl -A "Mozilla/5.0" -L "<hero-image-url>" -o assets/<brand>-brand/product-hero.png
```

### 3.3 UI screenshots, required for digital products

- App Store / Google Play product screenshots. Verify they are real UI, not only marketing mockups.
- Website screenshots section.
- Product demo video frames.
- Official X/Twitter launch screenshots.
- If the user has an account, ask for a real product screenshot.

## Step 3.4 — 5-10-2-8 Quality Gate Recording

The 5-10-2-8 gate (5 search rounds, 10 candidates, 2 picks, ≥8/10 each) is in `SKILL.md` §1.a as the load-bearing rule. **For each candidate**, record the score in `brand-spec.md` against these five dimensions:

1. **Resolution**: at least 2000px; print/large-screen work prefers 3000px+.
2. **Rights clarity**: official source > public domain > free stock > suspicious scrape; suspicious sources score 0.
3. **Brand fit**: aligned with the brand-spec tone keywords.
4. **Lighting / composition / style consistency**: selected assets should work together.
5. **Narrative independence**: each asset can carry a narrative role, not just decoration.

The philosophy is **quality over quantity**. Every visual element adds or subtracts points from the work.

## Step 4 · Verify And Extract

| Asset | Verification |
|---|---|
| **Logo** | File exists, opens, has dark/light variants when possible, transparent background |
| **Product image** | At least one 2000px+ image, clean or transparent background, multiple angles if possible |
| **UI screenshot** | Real resolution, current version, no private user data |
| **Colors** | `grep -hoE '#[0-9A-Fa-f]{6}' assets/<brand>-brand/*.{svg,html,css} \| sort \| uniq -c \| sort -rn \| head -20`, then filter black/white/gray |

**Beware demo-brand contamination**: product screenshots may include a customer's brand colors. Those are not the product's colors.

**Brands also have multiple valid faces**. A marketing site and product UI may use different palettes. Choose the face that matches the deliverable.

## Step 5 · Freeze Assets In `brand-spec.md`

```markdown
# <Brand> · Brand Spec
> Collected: YYYY-MM-DD
> Sources: <download/source list>
> Completeness: <complete / partial / inferred>

## Core Assets

### Logo
- Main: `assets/<brand>-brand/logo.svg`
- Light-background inverse: `assets/<brand>-brand/logo-white.svg`
- Use cases: <intro/outro/corner watermark/global>
- Do not: <stretch/recolor/add outline>

### Product Images (required for physical products)
- Hero: `assets/<brand>-brand/product-hero.png` (2000×1500)
- Details: `assets/<brand>-brand/product-detail-1.png`, `product-detail-2.png`
- Scene: `assets/<brand>-brand/product-scene.png`
- Use cases: <close-up/rotation/comparison>

### UI Screenshots (required for digital products)
- Home: `assets/<brand>-brand/ui-home.png`
- Core feature: `assets/<brand>-brand/ui-feature-<name>.png`
- Use cases: <product reveal/dashboard fade-in/comparison>

## Supporting Assets

### Palette
- Primary: #XXXXXX <source>
- Background: #XXXXXX
- Ink: #XXXXXX
- Accent: #XXXXXX
- Banned colors: <colors the brand clearly does not use>

### Type
- Display: <font stack>
- Body: <font stack>
- Mono, for data/HUD: <font stack>

### Signature Details
- <details worth executing at 120%>

### No-go Zone
- <explicitly forbidden choices>

### Tone Keywords
- <3-5 adjectives>
```

After writing the spec:

- All HTML must reference real asset file paths from `brand-spec.md`.
- Use the logo as an `<img>`, not a redraw.
- Use product images as `<img>`, not CSS silhouettes.
- Inject colors as CSS variables: `:root { --brand-primary: ...; }`.
- If a new color is needed, update the spec first.
