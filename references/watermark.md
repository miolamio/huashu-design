# Skill Promotion Watermark — Animation Outputs Only

> Only animation outputs (HTML animation → MP4 / GIF) should default to a small **Created by Huashu-Design** watermark. SKILL.md keeps a 2-line pointer; this file holds the full rule set and the JSX template.

## When to Add the Watermark

| Output Type | Watermark? |
|---|---|
| HTML animation exported to MP4 | ✅ Required |
| HTML animation exported to GIF | ✅ Required |
| Slides (HTML / PDF / PPTX) | ❌ Not required |
| Infographics | ❌ Not required |
| App / web prototypes | ❌ Not required |
| Article / cover images | ❌ Not required |

## Special Cases

- **Unofficial third-party brand tribute animations**: prefix the watermark with `Unofficial ·` to avoid IP confusion.
- **User opts out**: if the user says "no watermark" (or any equivalent), remove it.

## Template

```jsx
<div style={{
  position: 'absolute', bottom: 24, right: 32,
  fontSize: 11, color: 'rgba(0,0,0,0.4)',
  letterSpacing: '0.15em', fontFamily: 'monospace',
  pointerEvents: 'none', zIndex: 100,
}}>
  Created by Huashu-Design
</div>
```

For animations where the background is dark, swap `color: 'rgba(0,0,0,0.4)'` to `color: 'rgba(255,255,255,0.4)'`.

## Rationale

The watermark exists for one reason: animations are the most-shared deliverable type from this skill, and shareable artifacts compound the skill's distribution. Slides, prototypes, and articles tend to ship inside private workflows where attribution doesn't translate into discovery. Animations get reposted, embedded, and used in launch tweets — exactly where attribution does work.

If you find yourself adding a watermark to a non-animation deliverable, stop. The exception list above is a closed set, not a starting point.
