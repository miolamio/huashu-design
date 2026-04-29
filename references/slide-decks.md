# Slide Decks: HTML Slide Production Guidelines

Slide decks are a frequent design task. This document explains how to make strong HTML slide decks, from architecture choice and single-slide design to complete PDF/PPTX export paths.

**This skill covers**:
- **HTML presentation version (the base artifact, always required by default)** → independent HTML per slide + `assets/deck_index.html` aggregation, with keyboard navigation and full-screen presenting in the browser
- HTML → PDF export → `scripts/export_deck_pdf.mjs` / `scripts/export_deck_stage_pdf.mjs`
- HTML → editable PPTX export → `references/editable-pptx.md` + `scripts/html2pptx.js` + `scripts/export_deck_pptx.mjs` (requires writing HTML under 4 hard constraints)

> **⚠️ HTML is the foundation; PDF/PPTX are derivatives.** No matter what final format is delivered, you **must** first create an aggregated HTML presentation version (`index.html` + `slides/*.html`). It is the source of the slide work. PDF/PPTX are snapshots exported from HTML with a single command.
>
> **Why HTML first**:
> - Best for live talks and demos: full-screen directly through a projector or screen share, keyboard navigation, no dependency on Keynote/PPT software
> - During development, each slide can be opened and checked independently without rerunning export every time
> - It is the single upstream source for PDF/PPTX export, avoiding the loop of discovering changes only after export and then having to regenerate
> - The deliverable can be a pair such as "HTML + PDF" or "HTML + PPTX", letting the recipient use whichever they prefer
>
> 2026-04-22 moxt brochure field test: after completing 13 HTML slides plus an aggregated `index.html`, `export_deck_pdf.mjs` exported the PDF with one command and zero edits. The HTML version itself was already a browser-ready presentation deliverable.

---

## Contents

- [🛑 Confirm the Delivery Format Before Starting](#-confirm-the-delivery-format-before-starting-hardest-checkpoint) — HTML-first decision tree, PPTX 4-constraints rationale, mixed delivery, emergency recovery
- [🛑 Before Batch Production: 2-Page Showcase](#-before-batch-production-make-a-2-page-showcase-to-set-the-grammar) — set grammar before producing N slides
- [📐 Publication Grammar Template](#-publication-grammar-template-reusable-from-moxt-field-test) — per-slide skeleton, style conventions, visual-lead variation
- [⚠️ Common Pitfalls (moxt Field Notes)](#️-common-pitfalls-moxt-field-notes) — emoji, Playwright, fonts, density
- [🛑 Set the Architecture First: Single File or Multi-File?](#-set-the-architecture-first-single-file-or-multi-file) — decision tree, incident record
- [Path A (Default): Multi-File Architecture](#path-a-default-multi-file-architecture) — directory, template, aggregator, single-slide verification, parallel dev, shared tokens
- [Path B (Small Deck): Single File + deck_stage.js](#path-b-small-deck-single-file--deck_stagejs) — basic usage, 🛑 script-position constraint, CSS trap, starter CSS, custom size
- [Slide Labels](#slide-labels)
- [Speaker Notes](#speaker-notes) — format + writing guidelines
- [Slide Design Patterns](#slide-design-patterns) — establish a system, common layouts, scale, visual rhythm, breathing space
- [Print to PDF](#print-to-pdf)
- [Export to PPTX / PDF (Self-Serve Scripts)](#export-to-pptx--pdf-self-serve-scripts) — `export_deck_pdf.mjs`, `export_deck_stage_pdf.mjs`, `export_deck_pptx.mjs`, format-choice guidance
- [Deep Path for Exporting Editable PPTX](#deep-path-for-exporting-editable-pptx-long-term-projects-only) — long-term-only
- [FAQ](#faq)
- [Validation Checklist](#validation-checklist-required-after-finishing-a-deck) — required after every deck

---

## 🛑 Confirm the Delivery Format Before Starting (Hardest Checkpoint)

**This decision comes before "single file or multiple files."** 2026-04-20 options private-board project field test: **failing to confirm delivery format before implementation caused 2-3 hours of rework.**

### Decision Tree (HTML-First Architecture)

All deliverables start from the same aggregated HTML deck (`index.html` + `slides/*.html`). The delivery format only determines **HTML authoring constraints** and the **export command**:

```
[Always default · required] Aggregated HTML presentation (index.html + slides/*.html)
   │
   ├── Browser presentation only / local HTML archive   → Done here; HTML has maximum visual freedom
   │
   ├── Also need PDF (print / share / archive)          → Run export_deck_pdf.mjs once
   │                                                       HTML remains free-form, no visual constraints
   │
   └── Also need editable PPTX (teammates edit text)    → Write HTML from line one under the 4 hard constraints
                                                           Run export_deck_pptx.mjs once
                                                           Sacrifice gradients / web components / complex SVG
```

### Starting Script (Copy and Use)

> Whether the final delivery is HTML, PDF, or PPTX, I will first make an aggregated HTML presentation that can be navigated and presented in the browser (`index.html` with keyboard navigation). This is always the default base artifact. On top of that, I can export PDF / PPTX snapshots if you need them.
>
> Which export format do you need?
> - **HTML only** (presentation/archive) → complete visual freedom
> - **Also PDF** → same freedom, plus one export command
> - **Also editable PPTX** (teammates will edit text in PowerPoint) → I must write the HTML from the first line under 4 hard constraints, which will sacrifice some visual capability: no gradients, no web components, no complex SVG.

### Why PPTX Requires the 4 Hard Constraints from the Start

Editable PPTX depends on `html2pptx.js` being able to translate DOM elements into PowerPoint objects one by one. It needs **4 hard constraints**:

1. Fixed body size: 960pt × 540pt, matching `LAYOUT_WIDE` (13.333″ × 7.5″), not 1920×1080px
2. All text wrapped in `<p>`/`<h1>`-`<h6>` (no text directly inside divs, and no `<span>` carrying primary text)
3. `<p>`/`<h*>` elements themselves must not have background/border/shadow; put those on an outer div
4. `<div>` must not use `background-image`; use an `<img>` tag
5. Do not use CSS gradients, web components, or complex SVG decoration

**This skill's default HTML has high visual freedom**: many spans, nested flex, complex SVG, web components such as `<deck-stage>`, and CSS gradients. **Almost none of that naturally passes html2pptx constraints**. In field tests, visually driven HTML sent directly to html2pptx had a pass rate below 30%.

### Cost Comparison of Two Real Paths (2026-04-20 Field Failure)

| Path | Approach | Result | Cost |
|------|------|------|------|
| ❌ **Write free-form HTML first, patch PPTX afterward** | Single-file deck-stage + many SVG/span decorations | Editable PPTX leaves only two paths:<br>A. Hand-write hundreds of pptxgenjs coordinate lines<br>B. Rewrite 17 HTML slides into Path A format | 2-3 hours of rework, and the hand-written version has **permanent maintenance cost**: if HTML changes one word, PPTX must be manually synced again |
| ✅ **Write under Path A constraints from the start** | Independent HTML per slide + 4 hard constraints + 960×540pt | One command exports a 100% editable PPTX, while still supporting full-screen browser presentation. Path A HTML is standard browser-playable HTML | Spend 5 extra minutes while writing HTML thinking "how does this text fit into `<p>`"; zero rework |

### What About Mixed Delivery?

If the user says "I need HTML presentation **and** editable PPTX", **that is not mixed**; the PPTX requirement overrides the HTML requirement. HTML written under Path A can still be presented full-screen in the browser. Add a `deck_index.html` aggregator and it works. **No extra cost.**

If the user says "I need PPTX **and** animation / web components", **that is a real conflict**. Tell the user that editable PPTX requires sacrificing those visual capabilities. Ask them to choose; do not secretly implement a hand-written pptxgenjs path, because it becomes permanent maintenance debt.

### What If PPTX Is Requested After HTML Is Already Done? (Emergency Recovery)

Rare case: HTML is complete, then PPTX is requested. Use the recommended **fallback flow**. Full details are at the end of `references/editable-pptx.md` under "Fallback: existing visual draft but user insists on editable PPTX":

1. **First choice: export PDF**. It preserves the visuals 100%, is cross-platform, and the recipient can view and print it. If the real need is "presentation/archive", PDF is the best deliverable.
2. **Second choice: use the visual draft as a model and have AI rewrite an editable HTML version** → export editable PPTX. Preserve the design decisions around color, layout, and copy, while sacrificing gradients, web components, complex SVG, and similar visual capabilities.
3. **Not recommended: hand-rebuild with pptxgenjs**. Positioning, fonts, and alignment all require manual tuning; maintenance cost is high, and every later HTML text edit must be manually synced again.

Always tell the user the choices and let them decide. **Never make hand-written pptxgenjs the first reaction**; it is the last fallback.

---

## 🛑 Before Batch Production: Make a 2-Page Showcase to Set the Grammar

**For any deck with 5 or more slides, never start at page 1 and write straight through to the end.** The correct sequence was validated in the 2026-04-22 moxt brochure project:

1. Pick **2 page types with the biggest visual difference** for the showcase first, such as "cover" + "mood/quote page", or "cover" + "product showcase page"
2. Send screenshots to the user to confirm the grammar: masthead, type, color, spacing, structure, and Chinese/English bilingual ratio if applicable
3. Once the direction is approved, batch-produce the remaining N-2 pages, reusing the established grammar on every slide
4. After all slides are complete, assemble the aggregated HTML plus PDF / PPTX derivatives

**Why**: writing all 13 pages first and then hearing "wrong direction" means reworking 13 pages. Making a 2-page showcase first means a wrong direction costs 2 pages. Once the visual grammar is established, the decision space for the remaining N pages narrows sharply; the only remaining question is "how does this content fit?"

**How to choose showcase pages**: pick the two pages with the most different visual structures. If those two pass, the intermediate states will pass too.

| Deck Type | Recommended Showcase Pair |
|-----------|---------------------|
| B2B brochure / product promotion | Cover + content page (concept/emotion page) |
| Brand launch | Cover + product feature page |
| Data report | Large data-visual page + analytical conclusion page |
| Tutorial/courseware | Chapter opener + concrete knowledge-point page |

---

## 📐 Publication Grammar Template (Reusable from moxt Field Test)

Suitable for B2B brochures, product promotion, and long report decks. Reusing this structure on every page produced 13 visually consistent pages with zero rework.

### Per-Slide Skeleton

```
┌─ masthead (top strip + rule) ────────────┐
│  [logo 22-28px] · A Product Brochure                Issue · Date · URL │
├──────────────────────────────────────────┤
│                                          │
│  ── kicker (short green rule + uppercase label) │
│  CHAPTER XX · SECTION NAME                 │
│                                          │
│  H1 (Noto Serif SC 900 for Chinese)       │
│  Important words alone use the primary brand color │
│                                          │
│  English subtitle (Lora italic, subtitle) │
│  ─────────── divider ──────────           │
│                                          │
│  [specific content: two columns 60/40 / 2x2 grid / list] │
│                                          │
├──────────────────────────────────────────┤
│ section name                     XX / total │
└──────────────────────────────────────────┘
```

### Style Conventions (Copy Directly)

- **H1**: Noto Serif SC 900 for Chinese, 80-140px depending on information volume. Put only the key word in the primary brand color; do not flood the whole heading with color.
- **English subtitle**: Lora italic 26-46px; signature brand words such as "AI team" can be bold plus primary-color italic
- **Body**: Noto Serif SC 17-21px, line-height 1.75-1.85
- **Accent highlight**: use primary-color bold for key terms inside body copy; no more than 3 per slide, or the anchor effect disappears
- **Background**: warm off-white #FAFAFA plus extremely subtle radial-gradient noise (`rgba(33,33,33,0.015)`) for paper texture

### The Visual Lead Must Vary

If all 13 pages are "text + one screenshot", the deck becomes monotonous. **Rotate the type of visual lead on each page**:

| Visual Type | Suitable Section |
|---------|---------------|
| Cover typography (large type + masthead + pillar) | First page / chapter opener |
| Single-character portrait, such as one oversized momo | Introducing one concept/character |
| Multi-character group / avatar cards side by side | Team / user cases |
| Timeline cards in sequence | Showing long-term relationship or evolution |
| Knowledge graph / connected node diagram | Showing collaboration or flow |
| Before/After comparison cards + middle arrow | Showing change or difference |
| Product UI screenshot + outlined device frame | Concrete feature showcase |
| Big-quote (large type across half the slide) | Emotion page / problem page / quote page |
| Real-person avatar + quote cards (2×2 or 1×4) | Testimonials / usage scenarios |
| Large-type back cover + oval URL button | CTA / ending |

---

## ⚠️ Common Pitfalls (moxt Field Notes)

### 1. Emoji Do Not Render in Chromium / Playwright Export

Chromium does not include color emoji fonts by default, so emoji appear as empty boxes in `page.pdf()` or `page.screenshot()`.

**Fix**: use Unicode text symbols (`✦` `✓` `✕` `→` `·` `—`) instead, or switch to plain text, such as "Email · 23" instead of "📧 23 emails".

### 2. `export_deck_pdf.mjs` Fails with `Cannot find package 'playwright'`

Cause: ESM module resolution searches upward from the script location. If the script is in `~/.claude/skills/huashu-design/scripts/`, dependencies are not installed there.

**Fix**: copy the script into the deck project directory, such as `brochure/build-pdf.mjs`, run `npm install playwright pdf-lib` from the project root, then run `node build-pdf.mjs --slides slides --out output/deck.pdf`.

### 3. Google Fonts Not Loaded Before Screenshot → Chinese Renders as Default System Font

Before Playwright screenshot/PDF, wait at least `wait-for-timeout=3500` so webfonts can download and paint. Alternatively, self-host fonts in `shared/fonts/` to reduce network dependency.

### 4. Unbalanced Information Density: Content Slide Is Too Packed

The first version of the moxt philosophy page used 2×2 = 4 paragraphs plus 3 bottom principles, for 7 content blocks total. It felt crowded and repetitive. Switching to 1×3 = 3 paragraphs immediately restored breathing room.

**Fix**: keep each slide to "1 core message + 3-4 supporting points + 1 visual lead." If there is more, split it into a new slide. **Less is more**: the audience sees a slide for about 10 seconds, and one memory point is easier to retain than four.

---

## 🛑 Set the Architecture First: Single File or Multi-File?

**This choice is the first step in building a slide deck. Getting it wrong causes repeated failures. Read this section before implementation.**

### Two Architecture Options

| Dimension | Single File + `deck_stage.js` | **Multi-File + `deck_index.html` Aggregator** |
|------|--------------------------|--------------------------------------|
| Code structure | One HTML file, all slides are `<section>` | One standalone HTML file per slide, `index.html` aggregates through iframes |
| CSS scope | ❌ Global; one slide's styles may affect all slides | ✅ Naturally isolated; each iframe has its own world |
| Verification granularity | ❌ Need JS goTo to switch to a specific slide | ✅ Double-click a single slide file to view it in the browser |
| Parallel development | ❌ One file; multiple agents editing it will conflict | ✅ Multiple agents can work on different slides in parallel with conflict-free merges |
| Debugging difficulty | ❌ One CSS error can wreck the whole deck | ✅ One broken slide only affects itself |
| Embedded interaction | ✅ Shared cross-slide state is easy | 🟡 Requires postMessage between iframes |
| PDF printing | ✅ Built in | ✅ Aggregator walks iframes before print |
| Keyboard navigation | ✅ Built in | ✅ Built into the aggregator |

### Which One? (Decision Tree)

```
│ Question: roughly how many slides will the deck have?
├── ≤10 slides, needs in-deck animation or cross-slide interaction, pitch deck → single file
└── ≥10 slides, academic talk, courseware, long deck, multi-agent parallel work → multi-file (recommended)
```

**Default to the multi-file path.** It is not an alternative; it is the main path for long decks and team collaboration. Reason: every single-file advantage (keyboard navigation, print, scale) is also available in the multi-file approach, while multi-file scope isolation and verifiability cannot be patched back into single-file architecture.

### Why Is This Rule So Strict? (Incident Record)

Single-file architecture once hit four failures in a row during production of an AI psychology lecture deck:

1. **CSS specificity override**: `.emotion-slide { display: grid }` (specificity 10) overrode `deck-stage > section { display: none }` (specificity 2), causing all slides to render and overlap at once.
2. **Shadow DOM slot rules suppressed by outer CSS**: `::slotted(section) { display: none }` could not withstand an outer rule override, so sections would not hide.
3. **localStorage + hash navigation race**: after refresh, the deck did not jump to the hash target; it stayed at the old position recorded in localStorage.
4. **High verification cost**: capturing a specific page required `page.evaluate(d => d.goTo(n))`, which was twice as slow as directly opening `file://.../slides/05-X.html` and often errored.

The root cause was a **single global namespace**. Multi-file architecture removes these issues at the physical level.

---

## Path A (Default): Multi-File Architecture

### Directory Structure

```
MyDeck/
├── index.html              # copied from assets/deck_index.html, with MANIFEST edited
├── shared/
│   ├── tokens.css          # shared design tokens: palette, type scale, common chrome
│   └── fonts.html          # <link> imports for Google Fonts, included per slide
└── slides/
    ├── 01-cover.html       # each file is complete 1920×1080 HTML
    ├── 02-agenda.html
    ├── 03-problem.html
    └── ...
```

### Template Skeleton for Each Slide

```html
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<title>P05 · Chapter Title</title>
<link href="https://fonts.googleapis.com/css2?family=..." rel="stylesheet">
<link rel="stylesheet" href="../shared/tokens.css">
<style>
  /* Styles unique to this slide. Any class name is safe and will not pollute other slides. */
  body { padding: 120px; }
  .my-thing { ... }
</style>
</head>
<body>
  <!-- 1920×1080 content, with body width/height locked in tokens.css -->
  <div class="page-header">...</div>
  <div>...</div>
  <div class="page-footer">...</div>
</body>
</html>
```

**Key Constraints**:
- `<body>` is the canvas; lay out directly on it. Do not wrap with `<section>` or another wrapper.
- `width: 1920px; height: 1080px` is locked by the `body` rule in `shared/tokens.css`.
- Import `shared/tokens.css` for shared design tokens such as palette, type scale, and page-header/footer.
- Write font `<link>` tags in each slide. Per-slide font imports are cheap and ensure each slide can open independently.

### Aggregator: `deck_index.html`

**Copy directly from `assets/deck_index.html`**. You only need to edit one place: the `window.DECK_MANIFEST` array, listing all slide filenames and human-readable labels in order:

```js
window.DECK_MANIFEST = [
  { file: "slides/01-cover.html",    label: "Cover" },
  { file: "slides/02-agenda.html",   label: "Agenda" },
  { file: "slides/03-problem.html",  label: "Problem Statement" },
  // ...
];
```

The aggregator already includes keyboard navigation (←/→/Home/End/number keys/P to print), scale + letterbox, bottom-right counter, localStorage memory, hash slide jumps, and print mode that walks iframes and outputs pages to PDF.

### Single-Slide Verification (The Killer Advantage of Multi-File Architecture)

Every slide is standalone HTML. **After finishing one slide, double-click it in the browser and inspect it immediately**:

```bash
open slides/05-personas.html
```

Playwright screenshots can also go directly to `goto(file://.../slides/05-personas.html)`, with no JS slide jump and no interference from other slides' CSS. This makes the "change a little, verify a little" workflow nearly free.

### Parallel Development

Split the task by slide and hand different slides to different agents at the same time. HTML files are independent, so merges have no conflicts. For long decks, this parallel workflow can compress production time to roughly 1/N.

### What Belongs in `shared/tokens.css`

Only put things that are **truly shared across slides**:

- CSS variables: palette, type scale, spacing scale
- Canvas lock such as `body { width: 1920px; height: 1080px; }`
- Chrome used identically on every slide, such as `.page-header` / `.page-footer`

**Do not** put single-slide layout classes in here. That degrades back into the global-pollution problem of single-file architecture.

---

## Path B (Small Deck): Single File + `deck_stage.js`

Suitable for decks with 10 or fewer slides, decks that need cross-slide shared state (for example, a React tweaks panel controlling all slides), or compact pitch-deck demos.

### Basic Usage

1. Read content from `assets/deck_stage.js` and embed it into the HTML `<script>` tag, or use `<script src="deck_stage.js">`
2. Wrap slides in `<deck-stage>` inside body
3. 🛑 **The script tag must come after `</deck-stage>`**. See the hard constraint below.

```html
<body>

  <deck-stage>
    <section>
      <h1>Slide 1</h1>
    </section>
    <section>
      <h1>Slide 2</h1>
    </section>
  </deck-stage>

  <!-- ✅ Correct: script comes after deck-stage -->
  <script src="deck_stage.js"></script>

</body>
```

### 🛑 Hard Constraint on Script Position (2026-04-20 Field Failure)

**Do not put `<script src="deck_stage.js">` in `<head>`.** Even if it can define `customElements` from `<head>`, when the parser reaches the opening `<deck-stage>` tag it triggers `connectedCallback`; at that point child `<section>` nodes have not been parsed yet, so `_collectSlides()` gets an empty array, the counter shows `1 / 0`, and all slides render overlapped.

**Three valid patterns** (choose one):

```html
<!-- ✅ Most recommended: script after </deck-stage> -->
</deck-stage>
<script src="deck_stage.js"></script>

<!-- ✅ Also valid: script in head with defer -->
<head><script src="deck_stage.js" defer></script></head>

<!-- ✅ Also valid: module scripts are naturally deferred -->
<head><script src="deck_stage.js" type="module"></script></head>
```

`deck_stage.js` itself includes a defensive delayed collection through `DOMContentLoaded`, so a script in head will not completely break the deck. Still, `defer` or placing the script at the end of body is cleaner and avoids relying on the defensive branch.

### ⚠️ CSS Trap in Single-File Architecture (Read Carefully)

The most common single-file architecture failure is **a slide-level style stealing the `display` property**.

Common wrong pattern 1: writing `display: flex` directly on section:

```css
/* ❌ External CSS specificity 2 overrides shadow DOM ::slotted(section){display:none}, also specificity 2 */
deck-stage > section {
  display: flex;            /* All slides will render overlapped at once! */
  flex-direction: column;
  padding: 80px;
  ...
}
```

Common wrong pattern 2: section has a higher-specificity class:

```css
.emotion-slide { display: grid; }   /* specificity: 10, even worse */
```

Both cause **all slides to render overlapped at once**. The counter may still show `1 / 10` and look normal, but visually slide 1 is covering slide 2 covering slide 3.

### ✅ Starter CSS (Copy at Start to Avoid the Trap)

The **section itself** only handles visibility; **layout such as flex/grid belongs on `.active`**:

```css
/* section only defines non-display shared styles */
deck-stage > section {
  background: var(--paper);
  padding: 80px 120px;
  overflow: hidden;
  position: relative;
  /* ⚠️ Do not write display here! */
}

/* Lock "inactive means hidden" with both specificity and weight */
deck-stage > section:not(.active) {
  display: none !important;
}

/* Only the active slide gets the required display + layout */
deck-stage > section.active {
  display: flex;
  flex-direction: column;
  justify-content: center;
}

/* Print mode: all slides must display, overriding :not(.active) */
@media print {
  deck-stage > section { display: flex !important; }
  deck-stage > section:not(.active) { display: flex !important; }
}
```

Alternative: **put per-slide flex/grid on an inner wrapper `<div>`**. The section itself should always be only the `display: block/none` switch. This is the cleanest approach:

```html
<deck-stage>
  <section>
    <div class="slide-content flex-layout">...</div>
  </section>
</deck-stage>
```

### Custom Size

```html
<deck-stage width="1080" height="1920">
  <!-- 9:16 vertical -->
</deck-stage>
```

---

## Slide Labels

Both deck_stage and deck_index label each slide for counter display. Give them **meaningful** labels:

**Multi-file**: write `{ file, label: "04 Problem Statement" }` in `MANIFEST`
**Single-file**: add `<section data-screen-label="04 Problem Statement">`

**Key point: slide numbering starts at 1, not 0.**

When the user says "slide 5", they mean the 5th slide, never array index `[4]`. Humans do not speak in zero-indexed terms.

---

## Speaker Notes

**Do not add speaker notes by default**. Add them only when the user explicitly asks.

With speaker notes, you can reduce on-slide text to a minimum and focus on impactful visuals. Notes carry the full script.

### Format

**Multi-file**: write this in the `<head>` of `index.html`:

```html
<script type="application/json" id="speaker-notes">
[
  "Script for slide 1...",
  "Script for slide 2...",
  "..."
]
</script>
```

**Single-file**: same location.

### Notes Writing Guidelines

- **Complete**: not an outline; write the words that should actually be spoken
- **Conversational**: like normal speech, not formal prose
- **Mapped**: array item N corresponds to slide N
- **Length**: 200-400 words is ideal
- **Emotional line**: mark emphasis, pauses, and key points

---

## Slide Design Patterns

### 1. Establish a System (Required)

After exploring the design context, **first state the system you intend to use**:

```markdown
Deck system:
- Background colors: at most 2 (90% white + 10% dark section divider)
- Type: Instrument Serif for display, Geist Sans for body
- Rhythm: full-bleed colored section dividers with white type; ordinary slides on white
- Images: full-bleed photo for hero slides; charts for data slides

I will build with this system. Tell me if there is an issue.
```

Proceed only after the user confirms.

### 2. Common Slide Layouts

- **Title slide**: solid background + huge title + subtitle + author/date
- **Section divider**: colored background + section number + section title
- **Content slide**: white background + title + 1-3 bullet points
- **Data slide**: title + large chart/number + short explanation
- **Image slide**: full-bleed photo + small caption at the bottom
- **Quote slide**: whitespace + huge quote + attribution
- **Two-column**: left/right contrast such as vs / before-after / problem-solution

Use at most 4-5 layout types in one deck.

### 3. Scale (Repeated for Emphasis)

- Body text minimum **24px**, ideally 28-36px
- Titles **60-120px**
- Hero type **180-240px**
- Slides are read from 10 meters away; type must be large enough

### 4. Visual Rhythm

A deck needs **intentional variety**:

- Color rhythm: mostly white backgrounds + occasional colored section dividers + occasional dark moments
- Density rhythm: a few text-heavy slides + a few image-heavy slides + a few spacious quote slides
- Type-size rhythm: normal titles + occasional giant hero type

**Do not make every slide look the same**. That is a PPT template, not design.

### 5. Breathing Space (Required Reading for Data-Dense Slides)

**The easiest beginner mistake**: packing every possible piece of information onto one slide.

Information density does not equal effective communication. Academic and lecture decks especially need restraint:

- List/matrix slides: do not draw N elements at the same size. Use **primary/secondary layering**: enlarge the 5 items being discussed today as the protagonists, and shrink the remaining 16 into background hints.
- Big-number slides: the number itself is the visual lead. Surrounding captions should not exceed 3 lines, or the audience's eyes will keep bouncing around.
- Quote slides: leave whitespace between the quote and attribution; do not press them together.

Self-review with two questions: "Is the data the protagonist?" and "Is the text crowded?" Keep editing until the whitespace makes you slightly uncomfortable.

---

## Print to PDF

**Multi-file**: `deck_index.html` already handles the `beforeprint` event and outputs PDF page by page.

**Single-file**: `deck_stage.js` handles this too.

Print styles are already written; no extra `@media print` CSS is needed.

---

## Export to PPTX / PDF (Self-Serve Scripts)

HTML-first is the first-class path. Users often still need PPTX/PDF delivery, so two general scripts are provided. **Any multi-file deck can use them**. They live under `scripts/`:

### `export_deck_pdf.mjs` — Export Vector PDF (Multi-File Architecture)

```bash
node scripts/export_deck_pdf.mjs --slides <slides-dir> --out deck.pdf
```

**Features**:
- Text is **preserved as vectors**: copyable and searchable
- 100% visual fidelity: rendered by Playwright's embedded Chromium and printed
- **No HTML edits required**
- Each slide is independently rendered with `page.pdf()`, then merged with `pdf-lib`

**Dependencies**: `npm install playwright pdf-lib`

**Limit**: PDF text cannot be edited afterward. To make edits, return to the HTML.

### `export_deck_stage_pdf.mjs` — Dedicated to Single-File deck-stage Architecture ⚠️

**When to use it**: the deck is one HTML file, with a `<deck-stage>` web component wrapping N `<section>` nodes. In this Path B architecture, the `export_deck_pdf.mjs` approach of "one `page.pdf()` per HTML file" does not apply, so this dedicated script is required.

```bash
node scripts/export_deck_stage_pdf.mjs --html deck.html --out deck.pdf
```

**Why export_deck_pdf.mjs cannot be reused** (2026-04-20 field failure record):

1. **Shadow DOM beats `!important`**: deck-stage's shadow CSS contains `::slotted(section) { display: none }` and only the active slide gets `display: block`. Even if light DOM uses `@media print { deck-stage > section { display: block !important } }`, it cannot override the shadow rule. After `page.pdf()` triggers print media, Chromium ultimately renders only the active slide, so **the whole PDF has only 1 page**, repeating the current active slide.

2. **Looping goto per slide still produces 1 page**: the intuitive fix, "navigate once to each `#slide-N` and then `page.pdf({pageRanges:'1'})`", also fails. Because the print CSS outside shadow DOM has a `deck-stage > section { display: block }` rule that still gets overridden, final rendering is always the first section in the section list, not the slide you navigated to. The result: 17 loops produce 17 copies of P01 cover.

3. **Absolute children move to the next page**: even if all sections render successfully, if a section itself is `position: static`, its absolute-positioned `cover-footer`/`slide-footer` is positioned relative to the initial containing block. When print forces the section to 1080px height, the absolute footer may be pushed to the next page, producing a PDF with one extra page containing only an orphaned footer.

**Fix strategy** (already implemented in the script):

```js
// After opening the HTML, use page.evaluate to pull sections out of the deck-stage slot,
// attach them directly under a regular div in body, and inline styles to guarantee
// position:relative plus fixed dimensions.
await page.evaluate(() => {
  const stage = document.querySelector('deck-stage');
  const sections = Array.from(stage.querySelectorAll(':scope > section'));
  document.head.appendChild(Object.assign(document.createElement('style'), {
    textContent: `
      @page { size: 1920px 1080px; margin: 0; }
      html, body { margin: 0 !important; padding: 0 !important; }
      deck-stage { display: none !important; }
    `,
  }));
  const container = document.createElement('div');
  sections.forEach(s => {
    s.style.cssText = 'width:1920px!important;height:1080px!important;display:block!important;position:relative!important;overflow:hidden!important;page-break-after:always!important;break-after:page!important;background:#F7F4EF;margin:0!important;padding:0!important;';
    container.appendChild(s);
  });
  // Disable pagination on the last page to avoid a trailing blank page.
  sections[sections.length - 1].style.pageBreakAfter = 'auto';
  sections[sections.length - 1].style.breakAfter = 'auto';
  document.body.appendChild(container);
});

await page.pdf({ width: '1920px', height: '1080px', printBackground: true, preferCSSPageSize: true });
```

**Why this works**:
- It pulls sections out of the shadow DOM slot into a regular div in light DOM, fully bypassing the `::slotted(section) { display: none }` rule
- Inline `position: relative` makes absolute children position relative to the section, so they do not spill out
- `page-break-after: always` makes each section an independent page in browser print
- No pagination on `:last-child` avoids a trailing blank page

**When validating with `mdls -name kMDItemNumberOfPages`**: macOS Spotlight metadata is cached. After rewriting a PDF, run `mdimport file.pdf` to force refresh, or the old page count may still display. `pdfinfo` or counting files from `pdftoppm` gives the real count.

---

### `export_deck_pptx.mjs` — Export Editable PPTX

```bash
# Only mode: native editable text boxes; fonts may fall back to system fonts
node scripts/export_deck_pptx.mjs --slides <dir> --out deck.pptx
```

How it works: `html2pptx` reads computedStyle element by element and translates the DOM into PowerPoint objects: text frames, shapes, and pictures. Text becomes real text boxes that can be edited by double-clicking in PowerPoint.

**Hard constraints** (HTML must satisfy these, or the slide will be skipped; see `references/editable-pptx.md` for details):
- All text must be inside `<p>`/`<h1>`-`<h6>`/`<ul>`/`<ol>`; no bare text divs
- `<p>`/`<h*>` tags themselves must not have background/border/shadow; put those on an outer div
- Do not use `::before`/`::after` to insert decorative text; pseudo-elements cannot be extracted
- Inline elements such as span/em/strong must not have margin
- Do not use CSS gradients; they cannot be rendered
- Divs must not use `background-image`; use `<img>`

The script includes a built-in **automatic preprocessor** that wraps "bare text inside leaf divs" in `<p>` while preserving class names. This fixes the most common violation: bare text. Other violations, such as borders on `p` or margins on `span`, still must be corrected at the HTML source.

**Font fallback caveat**:
- Playwright uses webfonts to measure text-box dimensions; PowerPoint/Keynote render with local fonts
- If they differ, **overflow or misalignment** can occur. Every page still needs visual review.
- Ideally install the fonts used by the HTML on the target machine, or fall back to `system-ui`

**Do not use this path for visual-first scenarios** → export PDF with `export_deck_pdf.mjs` instead. PDF has 100% visual fidelity, vectors, cross-platform behavior, and searchable text; it is the right destination for visual-first decks, not an "uneditable compromise."

### Make HTML Export-Friendly from the Start

The most reliable deck for performance: **write HTML under editable constraints from the beginning**. Then `export_deck_pptx.mjs` can pass every slide directly. The extra cost is small:

```html
<!-- ❌ Bad -->
<div class="title">Key Finding</div>

<!-- ✅ Good: wrapped in p, class inherited -->
<p class="title">Key Finding</p>

<!-- ❌ Bad: border on p -->
<p class="stat" style="border-left: 3px solid red;">41%</p>

<!-- ✅ Good: border on outer div -->
<div class="stat-wrap" style="border-left: 3px solid red;">
  <p class="stat">41%</p>
</div>
```

### Which Format to Choose

| Scenario | Recommendation |
|------|------|
| For organizers / archive storage | **PDF**: universal, high fidelity, searchable text |
| Send to collaborators so they can tweak copy | **Editable PPTX**: accept font fallback |
| Live presentation with no content edits | **PDF**: vector fidelity, cross-platform |
| HTML is the preferred presentation medium | Play directly in browser; exports are backups |

## Deep Path for Exporting Editable PPTX (Long-Term Projects Only)

If your deck will be maintained long-term, repeatedly modified, and collaboratively edited, start by writing HTML under html2pptx constraints. Then `export_deck_pptx.mjs` can pass everything directly. See `references/editable-pptx.md` for details: 4 hard constraints, HTML template, common error quick reference, and fallback flow for existing visual drafts.

---

## FAQ

**Multi-file: a page inside iframe will not open / blank screen**
→ Check whether the `file` path in `MANIFEST` is correct relative to `index.html`. Use browser DevTools to see whether the iframe `src` is directly reachable.

**Multi-file: one slide's style conflicts with another**
→ Impossible because iframes isolate them. If it feels like a conflict, it is caching. Hard refresh with Cmd+Shift+R.

**Single-file: multiple slides render and overlap at once**
→ CSS specificity problem. See the "CSS Trap in Single-File Architecture" section above.

**Single-file: scaling looks wrong**
→ Check that every slide is a direct `<section>` child under `<deck-stage>`. There must not be an intermediate `<div>`.

**Single-file: want to jump to a specific slide**
→ Add a hash to the URL: `index.html#slide-5` jumps to the 5th slide.

**Both architectures: text positions differ across screens**
→ Use a fixed canvas size (1920×1080) and `px` units. Do not use `vw`/`vh` or `%`. Scaling is handled uniformly.

---

## Validation Checklist (Required After Finishing a Deck)

1. [ ] Open `index.html` directly in the browser, or the main HTML for single-file decks. Check the first page for broken images and loaded fonts.
2. [ ] Press the → key through every slide. There must be no blank pages and no layout shifts.
3. [ ] Press P for print preview. Each page should be exactly one A4 page, or one 1920×1080 page, with no cropping.
4. [ ] Hard-refresh 3 random pages with Cmd+Shift+R and confirm localStorage memory works correctly.
5. [ ] Run batch screenshots with Playwright. For multi-file architecture, iterate `slides/*.html`; for single-file architecture, switch with goTo. Review visually by eye.
6. [ ] Search for leftover `TODO` / `placeholder` and confirm they are cleared.
