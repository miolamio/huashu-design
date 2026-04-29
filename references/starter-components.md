# Starter Components — Quick Reference

> Components shipped with this skill that are designed to be **read and inlined** into user-generated single-file HTML (running from `file://`, no module system). SKILL.md keeps a one-line pointer; this file is the full inventory.

## Contents

- [Inventory](#inventory) — every starter component / script with purpose
- [How to Use Each](#how-to-use-each) — the inline-then-slot pattern
- [Architecture Constraints](#architecture-constraints) — why these are JSX/JS files instead of npm packages

---

## Inventory

| File | When To Use | Provides |
|---|---|---|
| `assets/deck_index.html` | Default base deliverable for slide decks | iframe stitching, keyboard navigation, scale, counter, print merge, isolated per-slide HTML |
| `assets/deck_stage.js` | Single-file slide decks ≤10 slides | Web component with auto-scale, keyboard navigation, counter, localStorage, speaker notes |
| `scripts/export_deck_pdf.mjs` | Multi-file HTML → PDF | Per-slide Playwright PDF + pdf-lib merge, vector searchable text |
| `scripts/export_deck_stage_pdf.mjs` | Single-file deck-stage HTML → PDF | Handles Shadow DOM slot and absolute-positioning pitfalls |
| `scripts/export_deck_pptx.mjs` | HTML → editable PPTX | Calls `html2pptx.js`; requires four hard constraints (see `references/editable-pptx.md`) |
| `scripts/html2pptx.js` | DOM → PPTX element translator | Converts `computedStyle` to PowerPoint objects |
| `assets/design_canvas.jsx` | Static variations | Labeled grid layout |
| `assets/animations.jsx` | Animation HTML | Stage + Sprite + useTime + Easing + interpolate. **Read the JSDoc header** for the runtime contract (`window.__ready`, `window.__recording`) |
| `assets/ios_frame.jsx` | iOS app mockup (mandatory; see SKILL.md §App / iOS Prototype Rules) | iPhone bezel, status bar, rounded screen |
| `assets/android_frame.jsx` | Android app mockup | Device bezel |
| `assets/macos_window.jsx` | Desktop app mockup | Window chrome + traffic lights |
| `assets/browser_window.jsx` | Webpage shown in browser | URL bar + tab bar |

## How to Use Each

The pattern is uniform across components:

1. **Read** the asset file from this skill (`assets/<component>.jsx` or `assets/<component>.js`).
2. **Inline** it into your target HTML's `<script type="text/babel">` block. For `.js` files like `deck_stage.js`, inline into a regular `<script>` block.
3. **Slot** the user's content into the component's expected JSX shape:

   ```jsx
   <IosFrame time="9:41" battery={85}>
     <YourScreen />
   </IosFrame>
   ```

For multi-file decks (`deck_index.html` + per-slide HTML), copy `deck_index.html` into the project root, rename to `index.html`, and place per-slide files under `slides/`.

## Architecture Constraints

These components are JSX/JS files, not an npm package, because the generated HTML runs from `file://` (double-click to open). Cross-origin restrictions block external `<script src="...">` and forcing a local HTTP server breaks the "double-click" deliverable intuition.

Consequences:

- Components cannot rely on `import` / `require` at runtime — they must be inlined.
- All dependencies (React, Babel) must come from CDN URLs reachable from `file://` (CDNs serve over HTTPS, which `file://` allows for `<script src=...>` to public domains).
- Local images must be base64 data URLs, not relative file paths.

If a deliverable exceeds about 1000 lines or needs a real module system, the workflow switches to multi-file architecture with a local HTTP server (`python3 -m http.server`). See `references/react-setup.md`.

## App / iOS Prototype Skeletons

Two patterns for app prototypes (per SKILL.md §App / iOS Prototype Rules):

### Overview Grid (default for design review)

Static complete screens, one iPhone per screen, no click requirement:

```jsx
<div style={{display: 'flex', gap: 32, flexWrap: 'wrap', padding: 48, alignItems: 'flex-start'}}>
  {screens.map(s => (
    <div key={s.id}>
      <div style={{fontSize: 13, color: '#666', marginBottom: 8, fontStyle: 'italic'}}>{s.label}</div>
      <IosFrame>
        <ScreenComponent data={s} />
      </IosFrame>
    </div>
  ))}
</div>
```

### Flow Demo (clickable walkthrough)

One iPhone with an `AppPhone` state manager; tabs/buttons/annotations clickable:

```jsx
function AppPhone({ initial = 'today' }) {
  const [screen, setScreen] = React.useState(initial);
  const [modal, setModal] = React.useState(null);
  // Render ScreenComponent by screen and pass callbacks.
}
```

Pass callback props such as `onEnter`, `onClose`, `onTabChange`, `onOpen`, and `onAnnotation`. Buttons, tabs, and cards need `cursor: pointer` plus hover feedback.
