# Tweaks: Real-Time Parameter Control for Design Variants

Tweaks are a core capability in this skill: they let users switch variations and adjust parameters in real time without editing code.

**Cross-agent environment adaptation**: some native design-agent environments (such as Claude.ai Artifacts) rely on the host's postMessage to write tweak values back into source code for persistence. This skill uses a **pure frontend localStorage approach** instead. The effect is the same (state persists after refresh), but persistence happens in browser localStorage rather than in source files. This approach works in any agent environment (Claude Code / Codex / Cursor / Trae / etc.).

## Contents

- [When to Add Tweaks](#when-to-add-tweaks)
- [Implementation (Pure Frontend Version)](#implementation-pure-frontend-version) — basic structure, Tweaks Panel UI, applying tweaks
- [Typical Tweak Options](#typical-tweak-options) — general, slide deck, product prototype, animation, landing page
- [Tweaks Design Principles](#tweaks-design-principles) — meaningful options, less-is-more, defaults are complete, group sensibly
- [Forward Compatibility with Source-Level Persistence Hosts](#forward-compatibility-with-source-level-persistence-hosts)
- [FAQ](#faq)

---

## When to Add Tweaks

- The user explicitly asks for "adjustable parameters" or "multiple version switching"
- The design has multiple variations that should be compared
- The user did not ask explicitly, but you judge that **a few meaningful tweaks would help the user see the possibility space**

Default recommendation: **add 2-3 tweaks to every design** (color theme / font size / layout variant), even if the user did not request them. Showing the user the possibility space is part of the design service.

## Implementation (Pure Frontend Version)

### Basic Structure

```jsx
const TWEAK_DEFAULTS = {
  "primaryColor": "#D97757",
  "fontSize": 16,
  "density": "comfortable",
  "dark": false
};

function useTweaks() {
  const [tweaks, setTweaks] = React.useState(() => {
    try {
      const stored = localStorage.getItem('design-tweaks');
      return stored ? { ...TWEAK_DEFAULTS, ...JSON.parse(stored) } : TWEAK_DEFAULTS;
    } catch {
      return TWEAK_DEFAULTS;
    }
  });

  const update = (patch) => {
    const next = { ...tweaks, ...patch };
    setTweaks(next);
    try {
      localStorage.setItem('design-tweaks', JSON.stringify(next));
    } catch {}
  };

  const reset = () => {
    setTweaks(TWEAK_DEFAULTS);
    try {
      localStorage.removeItem('design-tweaks');
    } catch {}
  };

  return { tweaks, update, reset };
}
```

### Tweaks Panel UI

Floating panel in the bottom-right corner. Collapsible:

```jsx
function TweaksPanel() {
  const { tweaks, update, reset } = useTweaks();
  const [open, setOpen] = React.useState(false);

  return (
    <div style={{
      position: 'fixed',
      bottom: 20,
      right: 20,
      zIndex: 9999,
    }}>
      {open ? (
        <div style={{
          background: 'white',
          border: '1px solid #e5e5e5',
          borderRadius: 12,
          padding: 20,
          boxShadow: '0 10px 40px rgba(0,0,0,0.12)',
          width: 280,
          fontFamily: 'system-ui',
          fontSize: 13,
        }}>
          <div style={{
            display: 'flex',
            justifyContent: 'space-between',
            alignItems: 'center',
            marginBottom: 16,
          }}>
            <strong>Tweaks</strong>
            <button onClick={() => setOpen(false)} style={{
              border: 'none', background: 'none', cursor: 'pointer', fontSize: 16,
            }}>×</button>
          </div>

          {/* Color */}
          <label style={{ display: 'block', marginBottom: 12 }}>
            <div style={{ marginBottom: 4, color: '#666' }}>Primary color</div>
            <input
              type="color"
              value={tweaks.primaryColor}
              onChange={e => update({ primaryColor: e.target.value })}
              style={{ width: '100%', height: 32 }}
            />
          </label>

          {/* Font size slider */}
          <label style={{ display: 'block', marginBottom: 12 }}>
            <div style={{ marginBottom: 4, color: '#666' }}>Font size ({tweaks.fontSize}px)</div>
            <input
              type="range"
              min={12} max={24} step={1}
              value={tweaks.fontSize}
              onChange={e => update({ fontSize: +e.target.value })}
              style={{ width: '100%' }}
            />
          </label>

          {/* Density options */}
          <label style={{ display: 'block', marginBottom: 12 }}>
            <div style={{ marginBottom: 4, color: '#666' }}>Density</div>
            <select
              value={tweaks.density}
              onChange={e => update({ density: e.target.value })}
              style={{ width: '100%', padding: 6 }}
            >
              <option value="compact">Compact</option>
              <option value="comfortable">Comfortable</option>
              <option value="spacious">Spacious</option>
            </select>
          </label>

          {/* Dark mode toggle */}
          <label style={{
            display: 'flex',
            alignItems: 'center',
            gap: 8,
            marginBottom: 16,
          }}>
            <input
              type="checkbox"
              checked={tweaks.dark}
              onChange={e => update({ dark: e.target.checked })}
            />
            <span>Dark mode</span>
          </label>

          <button onClick={reset} style={{
            width: '100%',
            padding: '8px 12px',
            background: '#f5f5f5',
            border: 'none',
            borderRadius: 6,
            cursor: 'pointer',
            fontSize: 12,
          }}>Reset</button>
        </div>
      ) : (
        <button
          onClick={() => setOpen(true)}
          style={{
            background: '#1A1A1A',
            color: 'white',
            border: 'none',
            borderRadius: 999,
            padding: '10px 16px',
            fontSize: 12,
            cursor: 'pointer',
            boxShadow: '0 4px 12px rgba(0,0,0,0.15)',
          }}
        >⚙ Tweaks</button>
      )}
    </div>
  );
}
```

### Applying Tweaks

Use Tweaks in the main component:

```jsx
function App() {
  const { tweaks } = useTweaks();

  return (
    <div style={{
      '--primary': tweaks.primaryColor,
      '--font-size': `${tweaks.fontSize}px`,
      background: tweaks.dark ? '#0A0A0A' : '#FAFAFA',
      color: tweaks.dark ? '#FAFAFA' : '#1A1A1A',
    }}>
      {/* Your content */}
      <TweaksPanel />
    </div>
  );
}
```

Use variables in CSS:

```css
button.cta {
  background: var(--primary);
  color: white;
  font-size: var(--font-size);
}
```

## Typical Tweak Options

Which tweaks to add for different types of designs:

### General
- Primary color (color picker)
- Font size (slider 12-24px)
- Typeface (select: display font vs body font)
- Dark mode (toggle)

### Slide Deck
- Theme (light/dark/brand)
- Background style (solid/gradient/image)
- Type contrast (more decorative vs more restrained)
- Information density (minimal/standard/dense)

### Product Prototype
- Layout variant (layout A / B / C)
- Interaction speed (animation speed 0.5x-2x)
- Data volume (mock data count 5/20/100)
- State (empty/loading/success/error)

### Animation
- Speed (0.5x-2x)
- Looping (once/loop/ping-pong)
- Easing (linear/easeOut/spring)

### Landing Page
- Hero style (image/gradient/pattern/solid)
- CTA copy (several variants)
- Structure (single column / two column / sidebar)

## Tweaks Design Principles

### 1. Meaningful Options, Not Busywork

Every tweak must expose a **real design option**. Do not add tweaks that no one would actually switch (for example, a border-radius slider from 0-50px where every intermediate value looks bad).

Good tweaks expose **discrete, considered variations**:
- "Corner style": square / slightly rounded / large rounded corners (three options)
- Not: "Corner radius": 0-50px slider

### 2. Less Is More

A design's Tweaks panel should have **at most 5-6 options**. More than that becomes a "configuration page" and loses the point of quickly exploring variations.

### 3. The Default Value Is a Complete Design

Tweaks are **an enhancement**. The default state must already be a complete, publishable design. What the user sees after closing the Tweaks panel is the output.

### 4. Group Sensibly

When there are many options, show them in groups:

```
---- Visual ----
Primary color | Font size | Dark mode

---- Layout ----
Density | Sidebar position

---- Content ----
Displayed data volume | State
```

## Forward Compatibility with Source-Level Persistence Hosts

If you later want the design to run in environments that support source-level tweaks (such as Claude.ai Artifacts), keep the **EDITMODE marker block**:

```jsx
const TWEAK_DEFAULTS = /*EDITMODE-BEGIN*/{
  "primaryColor": "#D97757",
  "fontSize": 16,
  "density": "comfortable",
  "dark": false
}/*EDITMODE-END*/;
```

The marker block has **no effect** in the localStorage approach (it is just a normal comment), but hosts that support source rewriting can read it and implement source-level persistence. Adding it is harmless in the current environment and keeps forward compatibility.

## FAQ

**The Tweaks panel blocks the design content**
→ Make it closable. Keep it closed by default, showing only a small button until the user expands it.

**The user has to repeat settings after switching tweaks**
→ localStorage is already used. If settings do not persist after refresh, check whether localStorage is available (private browsing can fail, so catch errors).

**Multiple HTML pages need to share tweaks**
→ Add the project name to the localStorage key: `design-tweaks-[projectName]`.

**I want tweaks to affect each other**
→ Add logic inside `update`:

```jsx
const update = (patch) => {
  let next = { ...tweaks, ...patch };
  // Coupling: when dark mode is selected, automatically switch text color
  if (patch.dark === true && !patch.textColor) {
    next.textColor = '#F0EEE6';
  }
  setTweaks(next);
  localStorage.setItem(...);
};
```
