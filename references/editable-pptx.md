# Editable PPTX Export: HTML Hard Constraints + Size Decisions + Common Errors

This document covers the path for **using `scripts/html2pptx.js` + `pptxgenjs` to translate HTML element by element into real editable PowerPoint text boxes**. It is also the only path supported by `export_deck_pptx.mjs`.

> **Core premise**: to use this path, the HTML must be written according to the following four constraints from the first line. **Do not write freely first and convert later**; retrofitting afterward can trigger 2-3 hours of rework (confirmed during the 2026-04-20 options private board project).
>
> For cases that prioritize visual freedom (animation / web components / CSS gradients / complex SVG), use the PDF path instead (`export_deck_pdf.mjs` / `export_deck_stage_pdf.mjs`). **Do not** expect PPTX export to provide both visual fidelity and editability; this is a physical constraint of the PPTX file format itself (see "Why the Four Constraints Are Physical Constraints, Not Bugs" at the end).

---

## Canvas Size: Use 960×540pt (`LAYOUT_WIDE`)

PPTX units are **inches** (physical size), not px. Decision principle: the body's computedStyle size must **match the presentation layout's inch size** (±0.1", enforced by `html2pptx.js` through `validateDimensions`).

### Three Candidate Sizes

| HTML body | Physical size | Matching PPT layout | When to choose |
|---|---|---|---|
| **`960pt × 540pt`** | **13.333″ × 7.5″** | **pptxgenjs `LAYOUT_WIDE`** | ✅ **Default recommendation** (modern PowerPoint 16:9 standard) |
| `720pt × 405pt` | 10″ × 5.625″ | Custom | Only when the user specifies an "old PowerPoint Widescreen" template |
| `1920px × 1080px` | 20″ × 11.25″ | Custom | ❌ Non-standard size; fonts look abnormally small after projection |

**Do not think of the HTML size as resolution.** PPTX is a vector document; the body size determines the **physical size**, not sharpness. An oversized body (20″×11.25″) will not make text clearer; it only makes font sizes smaller relative to the canvas, which looks worse when projected or printed.

### Three Equivalent Ways to Write the Body Size

```css
body { width: 960pt;  height: 540pt; }    /* Clearest, recommended */
body { width: 1280px; height: 720px; }    /* Equivalent, px habit */
body { width: 13.333in; height: 7.5in; }  /* Equivalent, inch intuition */
```

Matching pptxgenjs code:

```js
const pptx = new pptxgen();
pptx.layout = 'LAYOUT_WIDE';  // 13.333 × 7.5 inch, no custom layout needed
```

---

## Four Hard Constraints (Violations Fail Directly)

`html2pptx.js` translates the HTML DOM into PowerPoint objects element by element. PowerPoint formatting constraints projected onto HTML = the following four rules.

### Rule 1: Do Not Put Text Directly Inside DIVs; Wrap It in `<p>` or `<h1>`-`<h6>`

```html
<!-- ❌ Wrong: text directly inside div -->
<div class="title">Q3 revenue grew 23%</div>

<!-- ✅ Correct: text inside <p> or <h1>-<h6> -->
<div class="title"><h1>Q3 revenue grew 23%</h1></div>
<div class="body"><p>New users are the primary driver</p></div>
```

**Why**: PowerPoint text must live in a text frame, and text frames correspond to paragraph-level HTML elements (p/h*/li). A bare `<div>` has no matching text container in PPTX.

**Do not use `<span>` to carry primary text either**; span is an inline element and cannot be independently aligned as a text box. span can only be **inside p/h\*** for local styling (bold, color changes).

### Rule 2: CSS Gradients Are Not Supported; Use Solid Colors Only

```css
/* ❌ Wrong */
background: linear-gradient(to right, #FF6B6B, #4ECDC4);

/* ✅ Correct: solid color */
background: #FF6B6B;

/* ✅ If you must create multicolor stripes, use flex children with their own solid colors */
.stripe-bar { display: flex; }
.stripe-bar div { flex: 1; }
.red   { background: #FF6B6B; }
.teal  { background: #4ECDC4; }
```

**Why**: PowerPoint shape fills support solid/gradient-fill types, but pptxgenjs `fill: { color: ... }` only maps to solid fills. Native PowerPoint gradients require separate structure, which the current toolchain does not support.

### Rule 3: Backgrounds/Borders/Shadows Belong on DIVs, Not Text Tags

```html
<!-- ❌ Wrong: <p> has a background color -->
<p style="background: #FFD700; border-radius: 4px;">Key content</p>

<!-- ✅ Correct: outer div carries background/border; <p> only carries text -->
<div style="background: #FFD700; border-radius: 4px; padding: 8pt 12pt;">
  <p>Key content</p>
</div>
```

**Why**: In PowerPoint, a shape (rectangle/rounded rectangle) and a text frame are two objects. HTML `<p>` only translates to a text frame; backgrounds/borders/shadows belong to shapes and must be written on the **div that wraps the text**.

### Rule 4: DIVs Cannot Use `background-image`; Use `<img>` Tags

```html
<!-- ❌ Wrong -->
<div style="background-image: url('chart.png')"></div>

<!-- ✅ Correct -->
<img src="chart.png" style="position: absolute; left: 50%; top: 20%; width: 300pt; height: 200pt;" />
```

**Why**: `html2pptx.js` extracts image paths only from `<img>` elements. It does not parse CSS `background-image` URLs.

---

## Path A HTML Template Skeleton

Use one independent HTML file per slide, with isolated scopes (avoids CSS pollution from single-file decks).

```html
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<style>
  * { margin: 0; padding: 0; box-sizing: border-box; }
  body {
    width: 960pt; height: 540pt;           /* ⚠️ Matches LAYOUT_WIDE */
    font-family: system-ui, -apple-system, "PingFang SC", sans-serif;
    background: #FEFEF9;                    /* Solid color, no gradients */
    overflow: hidden;
  }
  /* DIV handles layout/background/borders */
  .card {
    position: absolute;
    background: #1A4A8A;                    /* Background on DIV */
    border-radius: 4pt;
    padding: 12pt 16pt;
  }
  /* Text tags only handle typography; no background/border */
  .card h2 { font-size: 24pt; color: #FFFFFF; font-weight: 700; }
  .card p  { font-size: 14pt; color: rgba(255,255,255,0.85); }
</style>
</head>
<body>

  <!-- Title area: outer div positions, inner text tags carry text -->
  <div style="position: absolute; top: 40pt; left: 60pt; right: 60pt;">
    <h1 style="font-size: 36pt; color: #1A1A1A; font-weight: 700;">Use an assertion sentence for the title, not a topic label</h1>
    <p style="font-size: 16pt; color: #555555; margin-top: 10pt;">Subtitle adds supporting context</p>
  </div>

  <!-- Content card: div handles background, h2/p handle text -->
  <div class="card" style="top: 130pt; left: 60pt; width: 240pt; height: 160pt;">
    <h2>Key Point One</h2>
    <p>Short supporting copy</p>
  </div>

  <!-- Lists: use ul/li, not manually typed • bullets -->
  <div style="position: absolute; top: 320pt; left: 60pt; width: 540pt;">
    <ul style="font-size: 16pt; color: #1A1A1A; padding-left: 24pt; list-style: disc;">
      <li>First key point</li>
      <li>Second key point</li>
      <li>Third key point</li>
    </ul>
  </div>

  <!-- Illustration: use <img>, not background-image -->
  <img src="illustration.png" style="position: absolute; right: 60pt; top: 110pt; width: 320pt; height: 240pt;" />

</body>
</html>
```

---

## Common Error Lookup

| Error message | Cause | Fix |
|---------|------|---------|
| `DIV element contains unwrapped text "XXX"` | Bare text inside div | Wrap text in `<p>` or `<h1>`-`<h6>` |
| `CSS gradients are not supported` | Used linear/radial-gradient | Change to a solid color, or use segmented flex children |
| `Text element <p> has background` | Background color on a `<p>` tag | Wrap it in a `<div>` that carries the background; keep `<p>` text-only |
| `Background images on DIV elements are not supported` | div uses background-image | Change to an `<img>` tag |
| `HTML content overflows body by Xpt vertically` | Content exceeds 540pt | Reduce content, shrink font size, or clip with `overflow: hidden` |
| `HTML dimensions don't match presentation layout` | Body size does not match presentation layout | Use `960pt × 540pt` body with `LAYOUT_WIDE`, or define a custom size with defineLayout |
| `Text box "XXX" ends too close to bottom edge` | Large `<p>` text is < 0.5 inch from the body bottom edge | Move it upward and leave enough bottom margin; PowerPoint itself obscures part of the bottom |

---

## Basic Workflow (PPTX in 3 Steps)

### Step 1: Write Each Page as Independent HTML Under the Constraints

```
MyDeck/
├── slides/
│   ├── 01-cover.html    # Each file is complete 960×540pt HTML
│   ├── 02-agenda.html
│   └── ...
└── illustration/        # All images referenced by <img>
    ├── chart1.png
    └── ...
```

### Step 2: Write `build.js` to Call `html2pptx.js`

```js
const pptxgen = require('pptxgenjs');
const html2pptx = require('../scripts/html2pptx.js');  // This skill's script

(async () => {
  const pres = new pptxgen();
  pres.layout = 'LAYOUT_WIDE';  // 13.333 × 7.5 inch, matches the HTML's 960×540pt

  const slides = ['01-cover.html', '02-agenda.html', '03-content.html'];
  for (const file of slides) {
    await html2pptx(`./slides/${file}`, pres);
  }

  await pres.writeFile({ fileName: 'deck.pptx' });
})();
```

### Step 3: Open and Check

- Open the exported PPTX in PowerPoint/Keynote
- Double-click any text; it should be directly editable (if it is an image, Rule 1 was violated)
- Verify overflow: every page should stay within the body bounds, with no clipping

---

## This Path vs. Other Options (What to Choose When)

| Need | Choose |
|------|------|
| Colleagues will edit PPTX text / send it to non-technical people for continued editing | **This path** (editable; requires writing HTML under the four constraints from the start) |
| Presentation only / archive delivery, no later editing | `export_deck_pdf.mjs` (multi-file) or `export_deck_stage_pdf.mjs` (single-file deck-stage), exports vector PDF |
| Visual freedom comes first (animation, web components, CSS gradients, complex SVG), and non-editability is acceptable | **PDF** (same as above); PDF is both faithful and cross-platform, more appropriate than an "image PPTX" |

**Never force visually free HTML through html2pptx**. In real tests, visually driven HTML had a pass rate below 30%; retrofitting the rest page by page is slower than rewriting. In this scenario, export PDF instead of squeezing it into PPTX.

---

## Fallback: A Visual Draft Already Exists, but the User Insists on Editable PPTX

This case comes up occasionally: you/the user already wrote a visually driven HTML draft (with gradients, web components, complex SVG, etc.). PDF would be the right output, but the user explicitly says, "No, it must be an editable PPTX."

**Do not run `html2pptx` and hope it passes**. In real tests, visually driven HTML passes html2pptx less than 30% of the time; the remaining 70% either fails or becomes visually distorted. The correct fallback is:

### Step 1 · Explain the Limits First (Transparent Communication)

Tell the user these three things in one sentence:

> "Your current HTML uses [list specifics: gradients / web components / complex SVG / ...], so direct conversion to editable PPTX will fail. I have two options:
> - A. **Export PDF** (recommended): preserves the visual design 100%; recipients can view and print it but cannot edit text
> - B. **Rewrite an editable HTML version using the visual draft as the blueprint**: preserve the design decisions around color/layout/copy, but reorganize the HTML under the four hard constraints, **sacrificing** visual capabilities such as gradients, web components, and complex SVG → then export editable PPTX
>
> Which do you choose?"

Do not make Option B sound trivial; clearly state **what will be lost**. Let the user make the tradeoff.

### Step 2 · If the User Chooses B: The AI Rewrites Proactively

The doctrine here is: **the user provides design intent; you translate it into a compliant implementation**. Do not ask the user to learn the four hard constraints and rewrite it themselves.

Principles for rewriting:
- **Preserve**: color system (primary/secondary/neutral colors), information hierarchy (title/subtitle/body/notes), core copy, layout skeleton (top/middle/bottom / left-right columns / grid), page rhythm
- **Degrade**: CSS gradients → solid colors or flex segments, web components → paragraph-level HTML, complex SVG → simplified `<img>` or solid-color geometry, shadows → remove or reduce to very weak, custom fonts → move toward system fonts
- **Rewrite**: bare text → wrap in `<p>` / `<h*>`, `background-image` → `<img>` tag, backgrounds/borders on `<p>` → outer div carries them

### Step 3 · Deliver a Comparison Checklist (Transparent Handoff)

After rewriting, give the user a before/after comparison so they know which visual details were simplified:

```
Original design → editable version adjustment
- Purple gradient in title area → solid primary color background #5B3DE8
- Data card shadows → removed (replaced with 2pt stroke separation)
- Complex SVG line chart → simplified into <img> PNG (generated from HTML screenshot)
- Hero web component animation → static first frame (web components cannot be translated)
```

### Step 4 · Export and Deliver Both Formats

- `editable` HTML version → run `scripts/export_deck_pptx.mjs` to create an editable PPTX
- **Recommend also keeping** the original visual draft → run `scripts/export_deck_pdf.mjs` to create a high-fidelity PDF
- Deliver both formats to the user: PDF from the visual draft + editable PPTX, each serving its role

### When to Refuse Option B Directly

In a few cases, rewriting is too expensive and you should steer the user away from editable PPTX:
- The HTML's core value is animation or interaction (after rewriting, only a static first frame remains, losing 50%+ of the information)
- More than 30 pages, with rewrite cost exceeding 2 hours
- The visual design deeply depends on precise SVG / custom filters (after rewriting, it would barely relate to the original)

In that case, tell the user: "This deck is too expensive to rewrite; I recommend exporting PDF instead of PPTX. If the recipient truly requires pptx format, accept that the visual design will become much plainer. Should we switch to PDF?"

---

## Why the Four Constraints Are Physical Constraints, Not Bugs

These four rules are not `html2pptx.js` laziness; they are the result of **PowerPoint file format (OOXML) constraints** projected onto HTML:

- Text in PPTX must live in a text frame (`<a:txBody>`), which corresponds to paragraph-level HTML elements
- PPTX shapes and text frames are two objects; one element cannot simultaneously draw a background and write text
- PPTX shape fills have limited gradient support (only certain preset gradients, not arbitrary CSS angled gradients)
- PPTX picture objects must reference real image files, not CSS properties

Once you understand this, **do not expect the tool to get smarter**. The HTML implementation must adapt to the PPTX format, not the other way around.
