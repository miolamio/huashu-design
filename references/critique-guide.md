# In-Depth Design Critique Guide

> Detailed reference for Phase 7. Provides scoring criteria, scenario-specific priorities, and a checklist of common issues.

---

## Contents

- [Scoring Criteria Details](#scoring-criteria-details) — philosophy alignment, visual hierarchy, craft quality, functionality, originality (5 axes × 10 points)
- [Scenario-Specific Critique Priorities](#scenario-specific-critique-priorities)
- [Top 10 Common Design Issues](#top-10-common-design-issues) — AI tech cliché, weak hierarchy, too many colors, inconsistent spacing, insufficient whitespace, too many fonts, alignment drift, decoration over content, cyber neon, density mismatch
- [Critique Output Template](#critique-output-template) — Strengths (Keep), Issues (Fix), Quick Wins

---

## Scoring Criteria Details

### 1. Philosophy Alignment

| Score | Criteria |
|------|------|
| 9-10 | The design perfectly embodies the core spirit of the selected philosophy; every detail has philosophical grounding |
| 7-8 | Overall direction is correct, core traits are present, with a few details drifting off |
| 5-6 | The intent is visible, but execution mixes in elements from other styles and lacks purity |
| 3-4 | Surface-level imitation only, without understanding the philosophy's core |
| 1-2 | Basically unrelated to the selected philosophy |

**Review focus**:
- Does it use signature techniques from that designer/institution?
- Do color, typography, and layout match that philosophical system?
- Are there "self-contradictory" elements? (For example, choosing Kenya Hara while stuffing the design with content.)

### 2. Visual Hierarchy

| Score | Criteria |
|------|------|
| 9-10 | The user's eye naturally follows the designer's intended path, with zero friction in information intake |
| 7-8 | Primary and secondary relationships are clear, with only 1-2 slightly ambiguous hierarchy points |
| 5-6 | Titles and body text can be distinguished, but middle levels are confusing |
| 3-4 | Information is flat, with no clear visual entry point |
| 1-2 | Chaotic; the user does not know where to look first |

**Review focus**:
- Is the font-size contrast between headings and body sufficient? (At least 2.5x)
- Do color/weight/size establish 3-4 clear levels?
- Is whitespace guiding the eye?
- "Squint test": if you squint, is the hierarchy still clear?

### 3. Craft Quality

| Score | Criteria |
|------|------|
| 9-10 | Pixel-level precision, with no flaws in alignment, spacing, or color |
| 7-8 | Overall refined, with 1-2 minor alignment/spacing issues |
| 5-6 | Basically aligned, but spacing is inconsistent and color usage is not systematic enough |
| 3-4 | Obvious alignment errors, chaotic spacing, too many colors |
| 1-2 | Rough; looks like a draft |

**Review focus**:
- Does it use a consistent spacing system (such as an 8pt grid)?
- Is spacing consistent among similar elements?
- Is the number of colors controlled? (Usually no more than 3-4)
- Are font families consistent? (Usually no more than 2)
- Are edges precisely aligned?

### 4. Functionality

| Score | Criteria |
|------|------|
| 9-10 | Every design element serves the goal, with zero redundancy |
| 7-8 | Clear functional direction, with a small amount of removable decoration |
| 5-6 | Basically usable, but obvious decorative elements distract attention |
| 3-4 | Form over function; users must work to find information |
| 1-2 | Completely overwhelmed by decoration and unable to communicate information |

**Review focus**:
- If any element is removed, does the design get worse? (If not, it should be removed.)
- Is the CTA/key information in the most prominent position?
- Are there elements added "because they look good"?
- Does the information density match the medium? (PPT should not be too dense; PDF can be denser.)

### 5. Originality

| Score | Criteria |
|------|------|
| 9-10 | Fresh and striking, with a unique expression found within the philosophical framework |
| 7-8 | Has its own ideas; not a simple template application |
| 5-6 | Conventional; looks like a template |
| 3-4 | Uses many clichés (such as gradient spheres representing AI) |
| 1-2 | Entirely template-based or assembled from stock materials |

**Review focus**:
- Does it avoid common clichés? (See the "Common Issues Checklist" below.)
- Does it have personal expression while following the design philosophy?
- Are there design decisions that are "unexpected but reasonable"?

---

## Scenario-Specific Critique Priorities

Different output types require different review priorities:

| Scenario | Most Important Dimensions | Secondary | Can Be Relaxed |
|------|-----------|--------|--------|
| WeChat official account cover/illustration | Originality, visual hierarchy | Philosophy alignment | Functionality (single images have no interaction) |
| Infographic | Functionality, visual hierarchy | Craft quality | Originality (accuracy first) |
| PPT/Keynote | Visual hierarchy, functionality | Craft quality | Originality (clarity first) |
| PDF/white paper | Craft quality, functionality | Visual hierarchy | Originality (professionalism first) |
| Landing page/official website | Functionality, visual hierarchy | Originality | — (all-around requirements) |
| App UI | Functionality, craft quality | Visual hierarchy | Philosophy alignment (usability first) |
| Xiaohongshu image | Originality, visual hierarchy | Philosophy alignment | Craft quality (vibe first) |

---

## Top 10 Common Design Issues

### 1. AI Tech Cliché

**Problem**: gradient spheres, digital rain, blue circuit boards, robot faces
**Why it is a problem**: users are already tired of these visuals and cannot distinguish you from others
**Fix**: replace literal symbols with abstract metaphors (for example, use a "conversation" metaphor instead of a chat bubble icon)

### 2. Insufficient Type Hierarchy

**Problem**: heading and body sizes are too close (<2.5x)
**Why it is a problem**: users cannot quickly locate key information
**Fix**: make headings at least 3x the body size (for example body 16px -> heading 48-64px)

### 3. Too Many Colors

**Problem**: using more than 5 colors without primary/secondary structure
**Why it is a problem**: visual chaos and weak brand feeling
**Fix**: limit to 1 primary color + 1 secondary color + 1 accent color + grayscale

### 4. Inconsistent Spacing

**Problem**: arbitrary spacing between elements, with no system
**Why it is a problem**: looks unprofessional and creates chaotic visual rhythm
**Fix**: establish an 8pt grid system (use only 8/16/24/32/48/64px spacing)

### 5. Insufficient Whitespace

**Problem**: every space is filled with content
**Why it is a problem**: crowded information causes reading fatigue and reduces communication efficiency
**Fix**: whitespace should occupy at least 40% of the total area (60%+ for minimal styles)

### 6. Too Many Fonts

**Problem**: using more than 3 fonts
**Why it is a problem**: visual noise that weakens unity
**Fix**: use at most 2 fonts (1 for headings + 1 for body), and create variation with weight and size

### 7. Inconsistent Alignment

**Problem**: some elements are left-aligned, some centered, some right-aligned
**Why it is a problem**: destroys visual order
**Fix**: choose one alignment mode (left alignment recommended) and apply it globally

### 8. Decoration Over Content

**Problem**: background patterns/gradients/shadows steal attention from the main content
**Why it is a problem**: priorities are reversed; users came for information, not decoration
**Fix**: ask, "If I remove this decoration, does the design get worse?" If not, remove it

### 9. Overused Cyber Neon

**Problem**: dark blue background (#0D1117) + neon glow effects
**Why it is a problem**: it is a default aesthetic exclusion zone (the taste baseline of this skill), and it has become one of the biggest clichés; users can override this based on their own brand
**Fix**: choose a more distinctive color scheme (reference the color systems from the 20 styles)

### 10. Information Density Does Not Match the Medium

**Problem**: putting a full page of text into a PPT slide / stuffing 10 elements into a cover image
**Why it is a problem**: different media have different ideal information densities
**Fix**:
- PPT: 1 core point per slide
- Cover image: 1 visual focus
- Infographic: layered presentation
- PDF: can be denser, but needs clear navigation

---

## Critique Output Template

```
## Design Critique Report

**Overall Score**: X.X/10 [Excellent (8+)/Good (6-7.9)/Needs Improvement (4-5.9)/Failing (<4)]

**Category Scores**:
- Philosophy alignment: X/10 [one-sentence explanation]
- Visual hierarchy: X/10 [one-sentence explanation]
- Craft quality: X/10 [one-sentence explanation]
- Functionality: X/10 [one-sentence explanation]
- Originality: X/10 [one-sentence explanation]

### Strengths (Keep)
- [Point out specific things done well, using design language]

### Issues (Fix)
[Sorted by severity]

**1. [Issue name]** — ⚠️ Critical / ⚡ Important / 💡 Optimization
- Current: [describe current state]
- Problem: [why this is a problem]
- Fix: [specific operation, including values]

### Quick Wins
If you only have 5 minutes, prioritize these 3 things:
- [ ] [highest-impact fix]
- [ ] [second most important fix]
- [ ] [third most important fix]
```

---

**Version**: v1.0
**Updated**: 2026-02-13
