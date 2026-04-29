# POV-First Gate — Full Handling

> Detailed guidance for the **POV-First Gate** in `SKILL.md`. SKILL.md keeps the rule in one paragraph; this file has the full handling: what counts as a POV vs what doesn't, how to write one well, and how to resolve conflicts when the user later requests a change that contradicts it.
>
> **Why the gate exists**: blocklists ("don't use purple gradients") are reactive and age. A POV is **generative** — it forces commitment, then refuses anything that doesn't serve the commitment. Slop is the average of all directions; a POV is one direction.

## Contents

- [The Gate Format](#the-gate-format)
- [What Counts as a POV (And What Doesn't)](#what-counts-as-a-pov-and-what-doesnt)
- [Examples by Archetype](#examples-by-archetype)
- [Conflict Resolution](#conflict-resolution) — when the user requests something that violates the POV
- [Relationship to Other Rules](#relationship-to-other-rules)

---

## The Gate Format

**Before Step 1** of any design task, write one sentence in this shape:

> "This {deliverable} is for {audience}, so it {primary design move}, not {anti-pattern}."

The four slots are non-optional:

1. **{deliverable}** — concrete artifact noun (pricing page, launch animation, slide deck, app prototype, infographic, portfolio). Not "the design" or "the website."
2. **{audience}** — a specific group. *Skeptical CFOs* not "users." *Hiring managers under time pressure* not "viewers." *Design enthusiasts on Twitter* not "the public."
3. **{primary design move}** — the single load-bearing decision the deliverable makes. *Leads with numbers*. *Earns attention through craft*. *Surfaces work above bio*. *Treats every screen as a poster*.
4. **{anti-pattern}** — what the deliverable explicitly refuses. *Vibes*. *Flash*. *Stock-photo positivity*. *Bento grids*. The anti-pattern is what makes the POV testable.

If you can fill all four slots in one fluent sentence, you have a POV. If you can't, you don't yet understand the task — return to clarification questions.

## What Counts as a POV (And What Doesn't)

**A POV is**: the single load-bearing argument the deliverable must win. Singular. Testable. Refusable.

**A POV is not**:
- The brand voice ("Friendly but professional") — too vague to refuse anything.
- The visual aesthetic ("Editorial with a touch of brutalism") — that's an archetype choice, separate from the argument.
- A list of priorities ("Conversion, then trust, then SEO") — POV is one thing, not three.
- A description of the audience ("Designers in their 30s") — that's the audience slot only.
- A feature list ("Pricing comparison + testimonials + FAQ") — that's content, not POV.

**Test for a real POV**: take the inverse and ask if it's a coherent design choice. *"This pricing page leads with vibes, not numbers"* is coherent (it's just a different POV — maybe a B2C consumer-tech page). *"This pricing page leads with neither numbers nor vibes"* is incoherent. If the inverse is incoherent, your POV isn't sharp enough.

## Examples by Archetype

| Archetype (per `named-allowlist.md`) | Example POV |
|---|---|
| Editorial / Magazine | "This essay landing page is for committed readers who came from a long tweet, so it earns the click by treating type as a first-class citizen, not as decoration around a hero illustration." |
| Startup / Modern SaaS | "This pricing page is for skeptical CFOs evaluating us against three competitors, so it leads with numbers comparable to those competitors, not with our brand vibe." |
| Technical / Developer Tools | "This API doc is for developers who already left their IDE to find an example, so it surfaces the runnable code in the first 100px, not after a value-prop paragraph." |
| Code / Terminal UI | "This launch site for an AI-coding tool is for skeptical Hacker News readers, so it shows the tool working in real terminal sessions, not in stylized 3D illustrations." |
| Industrial / Brutalist | "This music label site is for fans who already know the genre, so it refuses explanation and treats the page as a poster, not as a marketing funnel." |
| Eastern Minimal | "This tea-ceremony product page is for buyers who already understand restraint, so it gives a single object 70% of the screen and refuses to crowd it with feature bullets." |
| Playful / Memphis | "This kids' app onboarding is for parents using the app on their child's behalf, so it wins through clarity of what their child will see, not through cuteness aimed at the parent." |

## Conflict Resolution

When the user later requests a change that contradicts the POV:

1. **Restate the POV** verbatim. ("The current POV is: 'This pricing page is for skeptical CFOs, so it leads with numbers, not vibes.'")
2. **Identify the conflict** in one sentence. ("Adding a hero illustration of a smiling team would shift the page toward vibes-first, contradicting the POV.")
3. **Offer the user a choice**, not a compromise:
   - **Option A**: keep the POV, refuse the change. Explain why the change works against what the POV is trying to win.
   - **Option B**: change the POV explicitly, then accept the new direction. Restate what new POV the user is choosing. ("New POV would be: 'This pricing page is for prospects who want to feel they belong, so it leads with social proof, not numbers.'")
4. **Do not silently absorb the conflict.** Half-changes — adding the illustration but making it small, hedging the numbers but keeping them — produce slop. The whole point of the POV is to forbid hedging.

## Relationship to Other Rules

- **Core Principle #0** (fact-verify) runs **before** POV-first when a brief names a specific product. You verify the product exists, then write the POV.
- **§1.a Core Asset Protocol** runs **after** POV-first. The POV shapes which assets matter. ("Skeptical CFO" POV → product UI screenshots matter more than founder photos.)
- **Anti-AI-slop list** (`SKILL.md` §6.2 + `references/content-guidelines.md`) is the *fallback* when the agent is uncertain. POV is the *primary* defense; anti-slop is the safety net.
- **Named allowlist** (`references/named-allowlist.md`) is the execution layer. POV picks the archetype; allowlist names the specific fonts/colors/grids that match.

POV → archetype → allowlist → execution. Reversing the order produces slop.
