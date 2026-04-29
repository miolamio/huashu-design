# Design Context: Start From Existing Context

**This is the most important one thing in this skill.**

Good hi-fi design must grow from existing design context. **Making hi-fi from nothing is a last resort and will almost certainly produce generic work**. So at the start of every design task, first ask: is there anything we can reference?

## What Counts as Design Context

In priority order, from highest to lowest:

### 1. The User's Design System/UI Kit

The user's own product component library, color tokens, typography specs, and icon system. **This is the ideal case**.

### 2. The User's Codebase

If the user provides a codebase, it contains real component implementations. Read those component files:
- `theme.ts` / `colors.ts` / `tokens.css` / `_variables.scss`
- Concrete components (`Button.tsx`, `Card.tsx`)
- Layout scaffold (`App.tsx`, `MainLayout.tsx`)
- Global stylesheets

**Read the code and copy exact values**: hex codes, spacing scale, font stack, border radius. Do not redraw from memory.

### 3. The User's Published Product

If the user has a live product but did not provide code, use Playwright or ask the user for screenshots.

```bash
# Use Playwright to screenshot a public URL
npx playwright screenshot https://example.com screenshot.png --viewport-size=1920,1080
```

This lets you see the real visual vocabulary.

### 4. Brand Guidelines / Logo / Existing Assets

The user may have logo files, brand color specs, marketing materials, or slide templates. These are all context.

### 5. Competitor References

If the user says "like the XX website", ask them for a URL or screenshot. **Do not** work from a vague impression in your training data.

### 6. Known Design Systems (Fallback)

If none of the above exists, use a recognized design system as the base:
- Apple HIG
- Material Design 3
- Radix Colors (color)
- shadcn/ui (components)
- Tailwind default palette

Clearly tell the user what you are using so they know it is a starting point, not the final answer.

## Process for Gathering Context

### Step 1: Ask the User

Required questions at task start (from `workflow.md`):

```markdown
1. Do you have an existing design system/UI kit/component library? Where is it?
2. Are there brand guidelines or color/typography specs?
3. Can you provide screenshots or URLs of the existing product?
4. Is there a codebase I can read?
```

### Step 2: If the User Says "No", Help Them Look

Do not give up immediately. Try:

```markdown
Let me see whether we can find clues:
- Do you have related designs from previous projects?
- What colors/typography does the company's marketing site use?
- What style is the product logo? Can you give me an image?
- Are there products you admire that can serve as references?
```

### Step 3: Read All Context You Can Find

If the user gives a codebase path, read:
1. **First list the file structure**: look for style/theme/component-related files.
2. **Read theme/token files**: lift concrete hex/px values.
3. **Read 2-3 representative components**: inspect the visual vocabulary (hover state, shadow, border, padding node pattern).
4. **Read global stylesheet**: base resets and font loading.
5. **If there is a Figma link/screenshot**: inspect the visual, but **trust code more**.

**Important**: **Do not** glance once and work from impression. You have not really lifted the system until you have extracted 30+ concrete values.

### Step 4: Vocalize the System You Will Use

After reading the context, tell the user what system you will use:

```markdown
Based on your codebase and product screenshots, I extracted this design system:

**Colors**
- Primary: #C27558 (from tokens.css)
- Background: #FDF9F0
- Text: #1A1A1A
- Muted: #6B6B6B

**Typography**
- Display: Instrument Serif (from @font-face in global.css)
- Body: Geist Sans
- Mono: JetBrains Mono

**Spacing** (from your scale system)
- 4, 8, 12, 16, 24, 32, 48, 64

**Shadow pattern**
- `0 1px 2px rgba(0,0,0,0.04)` (subtle card)
- `0 10px 40px rgba(0,0,0,0.1)` (elevated modal)

**Border radius**
- Small components 4px, cards 12px, buttons 8px

**Component vocabulary**
- Button: filled primary, outlined secondary, ghost tertiary, all with 8px radius
- Card: white background, subtle shadow, no border

I will start from this system. Does that look right?
```

Only begin after the user confirms.

## Designing From Nothing (Fallback When There Is No Context)

**Strong warning**: output quality will drop significantly in this situation. Tell the user clearly.

```markdown
You do not have design context, so I can only work from general intuition.
The output will be something that "looks OK but lacks distinctiveness".
Do you want to continue, or provide reference material first?
```

If the user insists on proceeding, make decisions in this order:

### 1. Choose an Aesthetic Direction

Do not produce a generic result. Pick a clear direction:
- brutally minimal
- editorial/magazine
- brutalist/raw
- organic/natural
- luxury/refined
- playful/toy
- retro-futuristic
- soft/pastel

Tell the user which one you chose.

### 2. Choose a Known Design System as the Skeleton

- Use Radix Colors for color (https://www.radix-ui.com/colors).
- Use shadcn/ui for component vocabulary (https://ui.shadcn.com).
- Use the Tailwind spacing scale (multiples of 4).

### 3. Choose a Distinctive Font Pairing

Do not use Inter/Roboto. Recommended pairings (free from Google Fonts):
- Instrument Serif + Geist Sans
- Cormorant Garamond + Inter Tight
- Bricolage Grotesque + Söhne (paid)
- Fraunces + Work Sans (note that Fraunces is already overused by AI)
- JetBrains Mono + Geist Sans (technical feel)

### 4. Give Reasoning for Every Key Decision

Do not choose silently. Write it in an HTML comment:

```html
<!--
Design decisions:
- Primary color: warm terracotta (oklch 0.65 0.18 25) — fits the "editorial" direction  
- Display: Instrument Serif for humanist, literary feel
- Body: Geist Sans for cleanness contrast
- No gradients — committed to minimal, no AI slop
- Spacing: 8px base, golden ratio friendly (8/13/21/34)
-->
```

## Import Strategy (When the User Provides a Codebase)

If the user says "import this codebase as reference":

### Small (<50 Files)

Read everything and internalize the context.

### Medium (50-500 Files)

Focus on:
- `src/components/` or `components/`
- All style/token/theme-related files
- 2-3 representative full-page components (`Home.tsx`, `Dashboard.tsx`)

### Large (>500 Files)

Ask the user to specify the focus:
- "I want to make a settings page" -> read existing settings-related code.
- "I want to make a new feature" -> read the overall shell + the closest reference.
- Do not try to be exhaustive; be precise.

## Working With Figma / Design Files

If the user provides a Figma link:

- **Do not** expect to directly "turn Figma into HTML"; that requires extra tools.
- Figma links are often not publicly accessible.
- Ask the user to export a **screenshot** and send it to you, and to provide specific color/spacing values.

If the user only provides a Figma screenshot, tell them:
- I can see the visual direction, but I cannot extract exact values.
- Please provide key numbers (hex, px), or export as code (Figma supports this).

## Final Reminder

**The quality ceiling of a project's design is determined by the quality of context you receive**.

Spending 10 minutes collecting context is more valuable than spending 1 hour drawing hi-fi from nothing.

**When there is no context, ask the user for it first instead of forcing your way forward**.
