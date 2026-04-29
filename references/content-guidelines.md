# Content Guidelines: Anti-AI Slop, Content Rules, Scale Specs

These are the easiest traps to fall into in AI design. This is a list of "what not to do", which is more important than "what to do", because AI slop is the default. If you do not actively avoid it, it will happen.

## Complete AI Slop Blacklist

### Visual Traps

**Bad: Aggressive Gradient Backgrounds**
- Purple -> pink -> blue full-screen gradients (the typical smell of AI-generated websites)
- Rainbow gradients in any direction
- Mesh gradients covering the whole background
- Good: if you use gradients, keep them subtle, within a single hue family, and intentionally placed as accents (for example button hover)

**Bad: Rounded Cards + Left Border Accent Color**
```css
/* This is the classic signature of an AI-flavored card */
.card {
  border-radius: 12px;
  border-left: 4px solid #3b82f6;
  padding: 16px;
}
```

These cards are everywhere in AI-generated dashboards. Want emphasis? Use something with more design intent: background color contrast, weight/size contrast, plain separators, or no card at all.

**Bad: Emoji Decoration**

Unless the brand itself uses emoji (for example Notion or Slack), do not put emoji in the UI. **Especially avoid**:
- Icons before headings such as rocket, lightning, sparkle, target, lightbulb
- Checkmarks in feature lists
- Arrows inside CTA buttons (a standalone arrow is OK; an emoji arrow is not)

If you lack icons, use a real icon library (Lucide/Heroicons/Phosphor) or a placeholder.

**Bad: SVG-Drawn Imagery**

Do not try to draw people, scenes, devices, objects, or abstract art with SVG. AI-drawn SVG imagery instantly reads as AI: childish and cheap. **A gray rectangle labeled "Illustration placeholder 1200x800" is 100 times better than a poor SVG hero illustration**.

The only acceptable SVG use cases:
- Real icons (16x16 to 32x32 scale)
- Geometric decorative elements
- Charts for data visualization

**Bad: Too Much Iconography**

Not every heading/feature/section needs an icon. Overusing icons makes the interface feel like a toy. Less is more.

**Bad: "Data Slop"**

Fabricated decorative stats:
- "10,000+ happy customers" (you do not know whether this is true)
- "99.9% uptime" (do not write it without real data)
- Decorative "metric cards" made from icon + number + label
- Fake data in mock tables dressed up too heavily

If there is no real data, leave a placeholder or ask the user for it.

**Bad: "Quote Slop"**

Fabricated user reviews or famous quotes as page decoration. Leave a placeholder and ask the user for real quotes.

### Typography Traps

**Avoid these overused fonts**:
- Inter (the default AI-generated web font)
- Roboto
- Arial / Helvetica
- Pure system font stack
- Fraunces (AI discovered it and overused it)
- Space Grotesk (a recent AI favorite)

**Use distinctive display + body pairings instead**. Directional ideas:
- Serif display + sans body (editorial feel)
- Mono display + sans body (technical feel)
- Heavy display + light body (contrast)
- Variable font for hero weight animation

Font resources:
- Less common good options from Google Fonts (Instrument Serif, Cormorant, Bricolage Grotesque, JetBrains Mono)
- Open-source font sites (Fraunces sibling fonts, Adobe Fonts)
- Do not invent font names out of thin air

### Color Traps

**Bad: Inventing Colors From Scratch**

Do not design an unfamiliar full color system from scratch. It usually will not harmonize.

**Good strategy**:
1. Brand color exists -> use the brand color, and fill missing color tokens with oklch interpolation.
2. No brand color but references exist -> sample colors from reference product screenshots.
3. Completely from zero -> choose a known color system (Radix Colors / Tailwind default palette / Anthropic brand), do not tune your own from scratch.

**Defining colors with oklch** is the most modern approach:
```css
:root {
  --primary: oklch(0.65 0.18 25);      /* warm terracotta */
  --primary-light: oklch(0.85 0.08 25); /* light shade in the same hue family */
  --primary-dark: oklch(0.45 0.20 25);  /* dark shade in the same hue family */
}
```

oklch keeps hue stable when adjusting lightness and is more useful than hsl.

**Bad: Casual Inverted Dark Mode**

Dark mode is not simple color inversion. Good dark mode requires reworking saturation, contrast, and accent colors. If you do not want to design dark mode, do not add it.

### Layout Traps

**Bad: Overused Bento Grid**

Every AI-generated landing page wants to use bento. Unless your information structure truly fits bento, use a different layout.

**Bad: Large Hero + 3-Column Features + Testimonials + CTA**

This landing page template is exhausted. If you want innovation, actually innovate.

**Bad: Every Card in a Card Grid Has the Same Size**

Asymmetry, different card sizes, some with images and some only with text, some spanning columns: that is what real designer work feels like.

## Content Rules

### 1. Don't Add Filler Content

Every element must earn its place. Empty space is a design problem solved through **composition** (contrast, rhythm, whitespace), **not** by filling it with content.

**Questions for judging filler**:
- If this content is removed, does the design get worse? If the answer is "no", remove it.
- What real problem does this element solve? If the answer is "it makes the page less empty", delete it.
- Does this stat/quote/feature have real data behind it? If not, do not invent it.

"One thousand no's for every yes."

### 2. Ask Before Adding Material

Do you think adding another paragraph/page/section would make it better? Ask the user first; do not add it unilaterally.

Reasons:
- The user understands their audience better than you do.
- Adding content has cost, and the user may not want it.
- Unilaterally adding content violates the "junior designer reporting work" relationship.

### 3. Create a System Up Front

After exploring the design context, **state the system you plan to use out loud** and ask the user to confirm:

```markdown
My design system:
- Colors: #1A1A1A main + #F0EEE6 background + #D97757 accent (from your brand)
- Typography: Instrument Serif for display + Geist Sans for body
- Rhythm: section titles use full-bleed colored backgrounds + white text; regular sections use white backgrounds
- Imagery: hero uses a full-bleed photo; feature sections use placeholders until you provide assets
- Use at most 2 background colors to avoid clutter

If you confirm this direction, I will start building.
```

Only start after the user confirms. This check-in prevents "getting halfway done and discovering the direction is wrong."

## Scale Specs

### Slides (1920x1080)

- Body text minimum **24px**, ideal 28-36px
- Headings 60-120px
- Section titles 80-160px
- Hero headlines can use very large type at 180-240px
- Never use text smaller than 24px on slides

### Printed Documents

- Body text minimum **10pt** (about 13.3px), ideal 11-12pt
- Headings 18-36pt
- Caption 8-9pt

### Web and Mobile

- Body text minimum **14px** (use 16px for older-adult-friendly experiences)
- Mobile body text **16px** (prevents iOS auto-zoom)
- Hit target (clickable elements) minimum **44x44px**
- Line height 1.5-1.7 (Chinese 1.7-1.8)

### Contrast

- Body text vs background **at least 4.5:1** (WCAG AA)
- Large text vs background **at least 3:1**
- Check with Chrome DevTools accessibility tools

## CSS Superpowers

**Advanced CSS features** are a designer's friend. Use them boldly:

### Typography

```css
/* Makes heading line breaks more natural, avoiding a lonely last word */
h1, h2, h3 { text-wrap: balance; }

/* Body copy wrapping, reducing widows and orphans */
p { text-wrap: pretty; }

/* Powerful Chinese typography: punctuation spacing compression and line-edge control */
p { 
  text-spacing-trim: space-all;
  hanging-punctuation: first;
}
```

### Layout

```css
/* CSS Grid + named areas = extremely readable */
.layout {
  display: grid;
  grid-template-areas:
    "header header"
    "sidebar main"
    "footer footer";
  grid-template-columns: 240px 1fr;
  grid-template-rows: auto 1fr auto;
}

/* Subgrid aligns card content */
.card { display: grid; grid-template-rows: subgrid; }
```

### Visual Effects

```css
/* Scrollbar with design intent */
* { scrollbar-width: thin; scrollbar-color: #666 transparent; }

/* Glassmorphism (use with restraint) */
.glass {
  backdrop-filter: blur(20px) saturate(150%);
  background: color-mix(in oklch, white 70%, transparent);
}

/* View Transitions API makes page transitions smooth */
@view-transition { navigation: auto; }
```

### Interaction

```css
/* The :has() selector makes conditional styling easier */
.card:has(img) { padding-top: 0; } /* cards with images have no top padding */

/* Container queries make components truly responsive */
@container (min-width: 500px) { ... }

/* New color-mix function */
.button:hover {
  background: color-mix(in oklch, var(--primary) 85%, black);
}
```

## Decision Cheatsheet: When You Hesitate

- Want to add a gradient? -> Usually do not.
- Want to add an emoji? -> Do not.
- Want to add rounded corners + border-left accent to a card? -> Do not; use another method.
- Want to draw a hero illustration with SVG? -> Do not draw it; use a placeholder.
- Want to add a decorative quote? -> First ask whether the user has a real quote.
- Want to add a row of icon features? -> First ask whether icons are needed; they may not be.
- Using Inter? -> Switch to something more distinctive.
- Using a purple gradient? -> Switch to a color system with a reason.

**When you think "adding this would make it prettier", that is usually a sign of AI slop**. Start with the simplest version, and only add when the user asks.
