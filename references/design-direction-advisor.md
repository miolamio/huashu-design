# Design Direction Advisor — 8-Phase Execution Detail

> Detailed execution procedure for the Design Direction Advisor (Fallback Mode) in `SKILL.md`.
> SKILL.md keeps the trigger, skip-when, lightweight version, and the school-routing table. **This file is the full 8-phase flow** with phase-by-phase guidance, the showcase gallery routing table, and the personal-asset-index workflow.
>
> Use when the user request is vague, the user asks for style recommendations, the project has no design context, or the user says "I do not know what style I want."

## Contents

- [Phase 1 · Understand The Need](#phase-1--understand-the-need)
- [Phase 2 · Consultant Restatement](#phase-2--consultant-restatement)
- [Phase 3 · Recommend Three Design Philosophies](#phase-3--recommend-three-design-philosophies) — the school table is in SKILL.md; this section is the per-direction content requirements
- [Phase 4 · Show Prebuilt Showcase Gallery](#phase-4--show-prebuilt-showcase-gallery) — scenario-to-directory routing
- [Phase 5 · Generate Three Visual Demos](#phase-5--generate-three-visual-demos)
- [Phase 6 · User Choice](#phase-6--user-choice)
- [Phase 7 · Generate AI Prompt](#phase-7--generate-ai-prompt)
- [Phase 8 · Return To Main Flow](#phase-8--return-to-main-flow)
- [Real Asset Priority for Personal/Product Work](#real-asset-priority-for-personalproduct-work)

---

## Phase 1 · Understand The Need

Ask at most three questions about audience, core message, emotional tone, and output format. Skip if already clear.

## Phase 2 · Consultant Restatement

Restate the essence, audience, scenario, and tone in 100-200 words. End with: "Based on this, I prepared three design directions."

## Phase 3 · Recommend Three Design Philosophies

The five-school routing table lives in `SKILL.md` (§Design Direction Advisor). Never recommend two directions from the same school.

Each direction must include:

- A designer or studio name, such as "Kenya Hara-style Eastern minimalism".
- 50-100 words explaining why it fits.
- 3-4 visual traits, 3-5 tone keywords, and optional representative works.

See `references/design-styles.md` for the 20-style library and AI prompt templates.

## Phase 4 · Show Prebuilt Showcase Gallery

After recommending directions, check `assets/showcases/INDEX.md` for a matching prebuilt sample:

| Scenario | Directory |
|---|---|
| WeChat article cover | `assets/showcases/cover/` |
| PPT data slide | `assets/showcases/ppt/` |
| Vertical infographic | `assets/showcases/infographic/` |
| Personal homepage / AI directory / AI writing / SaaS / developer docs | `assets/showcases/website-*/` |

Suggested phrasing: "Before generating live demos, here is how these three styles look in similar scenarios →"

## Phase 5 · Generate Three Visual Demos

Seeing beats describing. Generate one demo for each direction. If the current agent supports parallel subagents, run three in parallel; otherwise generate serially.

- Use the user's real content/topic, not Lorem ipsum.
- Save HTML to `_temp/design-demos/demo-[style].html`.
- Screenshot with `npx playwright screenshot file:///path.html out.png --viewport-size=1200,900`.
- Show all three screenshots together.

| Best Path | Demo Method |
|---|---|
| HTML style | Build full HTML → screenshot |
| AI-generated style | `nano-banana-pro` with style DNA + content description |
| Hybrid style | HTML layout + AI illustration |

## Phase 6 · User Choice

User chooses one, mixes parts, tweaks, or asks to restart at Phase 3.

## Phase 7 · Generate AI Prompt

Use `[design philosophy constraints] + [content description] + [technical parameters]`.

- Use concrete traits, colors, ratios, spatial distribution, and output specs.
- Do not rely only on style names.
- Avoid AI-slop patterns.

## Phase 8 · Return To Main Flow

Once a direction is confirmed, continue with Core Philosophy + Junior Designer workflow.

## Real Asset Priority for Personal/Product Work

1. Check the user's private memory path for `personal-asset-index.json` (Claude Code default: `~/.claude/memory/`; other agents use their own convention).
2. On first use, copy `assets/personal-asset-index.example.json` to that private path and fill real data.
3. If missing, ask the user. Do not invent personal data. Do not place real personal data inside the skill directory.
