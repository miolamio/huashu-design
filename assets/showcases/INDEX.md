# Design Philosophy Showcases — Sample Asset Index

> 8 scenarios × 3 styles = 24 prebuilt design samples.
> Used during Phase 3 design-direction recommendations to show what each style looks like in practice.

## Style Guide

| Code | School | Style Name | Visual Character |
|------|------|---------|---------|
| **Pentagram** | Information Architecture | Pentagram / Michael Bierut | Restrained black and white, Swiss grid, strong type hierarchy, #E63946 red accent |
| **Build** | Minimalism | Build Studio | Luxury-grade whitespace (70%+), subtle weights (200-600), #D4A574 warm gold, refined |
| **Takram** | Eastern Philosophy | Takram | Soft technology, natural colors (beige/gray/green), rounded corners, charts as art |

## Scenario Lookup

### Content Design Scenarios

| # | Scenario | Size | Pentagram | Build | Takram |
|---|------|------|-----------|-------|--------|
| 1 | WeChat article cover | 1200×510 | `cover/cover-pentagram` | `cover/cover-build` | `cover/cover-takram` |
| 2 | PPT data slide | 1920×1080 | `ppt/ppt-pentagram` | `ppt/ppt-build` | `ppt/ppt-takram` |
| 3 | Vertical infographic | 1080×1920 | `infographic/infographic-pentagram` | `infographic/infographic-build` | `infographic/infographic-takram` |

### Website Design Scenarios

| # | Scenario | Size | Pentagram | Build | Takram |
|---|------|------|-----------|-------|--------|
| 4 | Personal homepage | 1440×900 | `website-homepage/homepage-pentagram` | `website-homepage/homepage-build` | `website-homepage/homepage-takram` |
| 5 | AI directory | 1440×900 | `website-ai-nav/ainav-pentagram` | `website-ai-nav/ainav-build` | `website-ai-nav/ainav-takram` |
| 6 | AI writing tool | 1440×900 | `website-ai-writing/aiwriting-pentagram` | `website-ai-writing/aiwriting-build` | `website-ai-writing/aiwriting-takram` |
| 7 | SaaS landing page | 1440×900 | `website-saas/saas-pentagram` | `website-saas/saas-build` | `website-saas/saas-takram` |
| 8 | Developer docs | 1440×900 | `website-devdocs/devdocs-pentagram` | `website-devdocs/devdocs-build` | `website-devdocs/devdocs-takram` |

> Each entry has both an `.html` source file and a `.png` screenshot.

## Usage

### Reference During Phase 3 Recommendations

After recommending design directions, show the matching prebuilt screenshot:

```text
"This is what a WeChat article cover looks like in the Pentagram style → [show cover/cover-pentagram.png]"
"This is how a PPT data slide feels in the Takram style → [show ppt/ppt-takram.png]"
```

### Scenario Matching Priority

1. Exact match for the user's scenario → show that scenario directly.
2. No exact match but similar type → show the closest scenario, such as "product website" → SaaS landing page.
3. No reasonable match → skip prebuilt samples and generate live in Phase 3.5.

### Side-by-Side Comparison

The three styles for one scenario work well side by side for direct comparison:
- "This is the same WeChat article cover executed in three styles."
- Display order: Pentagram (rational and restrained) → Build (luxury minimalism) → Takram (soft and warm).

## Content Details

### WeChat Article Cover (`cover/`)

- Content: Claude Code agent workflow, 8-agent parallel architecture.
- Pentagram: huge red "8" + Swiss grid lines + data bars.
- Build: ultra-light "Agent" floating in 70% whitespace + warm-gold fine lines.
- Takram: 8-node radial flowchart as artwork + beige background.

### PPT Data Slide (`ppt/`)

- Content: GLM-4.7 open-source model coding breakthrough (AIME 95.7 / SWE-bench 73.8% / τ²-Bench 87.4).
- Pentagram: 260px "95.7" anchor + red/gray/light-gray comparison bars.
- Build: three floating 120px ultra-light numbers + warm-gold gradient comparison bars.
- Takram: SVG radar chart + three-color overlay + rounded data cards.

### Vertical Infographic (`infographic/`)

- Content: AI memory system, CLAUDE.md optimized from 93KB to 22KB.
- Pentagram: huge "93→22" number + numbered sections + CSS data bars.
- Build: extreme whitespace + soft-shadow cards + warm-gold connectors.
- Takram: SVG donut chart + organic curved flowchart + frosted-glass cards.

### Personal Homepage (`website-homepage/`)

- Content: portfolio homepage for independent developer Alex Chen.
- Pentagram: 112px name + Swiss grid columns + editorial numbers.
- Build: glass navigation + floating stat cards + ultra-light type.
- Takram: paper texture + small circular avatar + hairline separators + asymmetric layout.

### AI Directory (`website-ai-nav/`)

- Content: AI Compass, a directory of 500+ AI tools.
- Pentagram: square search box + numbered tool list + uppercase category labels.
- Build: rounded search box + refined white tool cards + pill tags.
- Takram: organic offset card layout + soft category labels + diagram-like connections.

### AI Writing Tool (`website-ai-writing/`)

- Content: Inkwell, an AI writing assistant.
- Pentagram: 86px headline + wireframe editor model + gridded feature columns.
- Build: floating editor card + warm-gold CTA + luxury writing experience.
- Takram: poetic serif headline + organic editor + flow diagram.

### SaaS Landing Page (`website-saas/`)

- Content: Meridian, a business-intelligence analytics platform.
- Pentagram: black/white split columns + structured dashboard + 140px "3x" anchor.
- Build: floating dashboard cards + SVG area chart + warm-gold gradient.
- Takram: rounded bar chart + flow nodes + soft earth colors.

### Developer Docs (`website-devdocs/`)

- Content: Nexus API, a unified AI model gateway.
- Pentagram: left navigation + square code blocks + red string highlights.
- Build: centered floating code card + soft shadow + warm-gold icons.
- Takram: beige code block + flowchart connections + dashed feature cards.

## File Count

- HTML source files: 24
- PNG screenshots: 24
- Total assets: 48 files

---

**Version**: v1.0
**Created**: 2026-02-13
**Applies to**: design-philosophy skill Phase 3 recommendations
