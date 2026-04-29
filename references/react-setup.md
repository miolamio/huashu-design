# React + Babel Project Rules

Technical rules you must follow when building prototypes with HTML + React + Babel. If you ignore them, things will break.

## Pinned Script Tags (Required Versions)

Place these three script tags in the HTML `<head>`, using **fixed versions + integrity hashes**:

```html
<script src="https://unpkg.com/react@18.3.1/umd/react.development.js" integrity="sha384-hD6/rw4ppMLGNu3tX5cjIb+uRZ7UkRJ6BPkLpg4hAu/6onKUg4lLsHAs9EBPT82L" crossorigin="anonymous"></script>
<script src="https://unpkg.com/react-dom@18.3.1/umd/react-dom.development.js" integrity="sha384-u6aeetuaXnQ38mYT8rp6sbXaQe3NL9t+IBXmnYxwkUI2Hw4bsp2Wvmx4yRQF1uAm" crossorigin="anonymous"></script>
<script src="https://unpkg.com/@babel/standalone@7.29.0/babel.min.js" integrity="sha384-m08KidiNqLdpJqLq95G/LEi8Qvjl/xUYll3QILypMoQ65QorJ9Lvtp2RXYGBFj1y" crossorigin="anonymous"></script>
```

**Do not** use unpinned versions like `react@18` or `react@latest`; they create version drift and caching issues.

**Do not** omit `integrity`; if the CDN is ever hijacked or tampered with, this is the defense.

## File Structure

```
project-name/
├── index.html               # Main HTML
├── components.jsx           # Component file (loaded with type="text/babel")
├── data.js                  # Data file
└── styles.css               # Extra CSS (optional)
```

Load files in HTML like this:

```html
<!-- React + Babel first -->
<script src="https://unpkg.com/react@18.3.1/..."></script>
<script src="https://unpkg.com/react-dom@18.3.1/..."></script>
<script src="https://unpkg.com/@babel/standalone@7.29.0/..."></script>

<!-- Then your component files -->
<script type="text/babel" src="components.jsx"></script>
<script type="text/babel" src="pages.jsx"></script>

<!-- Main entry last -->
<script type="text/babel">
  const root = ReactDOM.createRoot(document.getElementById('root'));
  root.render(<App />);
</script>
```

**Do not** use `type="module"`; it conflicts with Babel.

## Three Non-Negotiable Rules

### Rule 1: `styles` Objects Must Use Unique Names

**Wrong** (guaranteed to break with multiple components):
```jsx
// components.jsx
const styles = { button: {...}, card: {...} };

// pages.jsx  ← overwritten by the same name!
const styles = { container: {...}, header: {...} };
```

**Correct**: give each component file's styles a unique prefix.

```jsx
// terminal.jsx
const terminalStyles = {
  screen: {...},
  line: {...}
};

// sidebar.jsx
const sidebarStyles = {
  container: {...},
  item: {...}
};
```

**Or use inline styles** (recommended for small components):
```jsx
<div style={{ padding: 16, background: '#111' }}>...</div>
```

This rule is **non-negotiable**. Every time you write `const styles = {...}`, replace it with a specific name; otherwise, loading multiple components will crash the whole stack.

### Rule 2: Scopes Are Not Shared; Export Manually

**Key concept**: every `<script type="text/babel">` is compiled independently by Babel, so their **scopes do not connect**. A `Terminal` component defined in `components.jsx` is **undefined by default** inside `pages.jsx`.

**Solution**: at the end of each component file, export shared components/utilities to `window`:

```jsx
// End of components.jsx
function Terminal(props) { ... }
function Line(props) { ... }
const colors = { green: '#...', red: '#...' };

Object.assign(window, {
  Terminal, Line, colors,
  // List everything you need to use elsewhere
});
```

Then `pages.jsx` can use `<Terminal />` directly, because JSX will look for `window.Terminal`.

### Rule 3: Do Not Use `scrollIntoView`

`scrollIntoView` pushes the entire HTML container upward and breaks the web harness layout. **Never use it**.

Use these alternatives:
```js
// Scroll to a position inside the container
container.scrollTop = targetElement.offsetTop;

// Or use element.scrollTo
container.scrollTo({
  top: targetElement.offsetTop - 100,
  behavior: 'smooth'
});
```

## Calling the Claude API (Inside HTML)

Some native design-agent environments (such as Claude.ai Artifacts) provide zero-config `window.claude.complete` through the host, but most local agent environments (Claude Code / Codex / Cursor / Trae / etc.) **do not**.

If your HTML prototype needs to call an LLM for a demo (for example, a chat interface), use one of these options:

### Option A: Do Not Make a Real Call; Use a Mock

Recommended for demos. Write a fake helper that returns a preset response:
```jsx
window.claude = {
  async complete(prompt) {
    await new Promise(r => setTimeout(r, 800)); // Simulate latency
    return "This is a mock response. Replace it with a real API in production.";
  }
};
```

### Option B: Call the Anthropic API for Real

Requires an API key; the user must enter their own key in the HTML before it can run. **Never hardcode a key in HTML**.

```html
<input id="api-key" placeholder="Paste your Anthropic API key" />
<script>
window.claude = {
  async complete(prompt) {
    const key = document.getElementById('api-key').value;
    const res = await fetch('https://api.anthropic.com/v1/messages', {
      method: 'POST',
      headers: {
        'x-api-key': key,
        'anthropic-version': '2023-06-01',
        'content-type': 'application/json',
      },
      body: JSON.stringify({
        model: 'claude-haiku-4-5',
        max_tokens: 1024,
        messages: [{ role: 'user', content: prompt }]
      })
    });
    const data = await res.json();
    return data.content[0].text;
  }
};
</script>
```

**Note**: calling the Anthropic API directly from the browser can run into CORS issues. If the preview environment the user gave you does not support a CORS bypass, this path will not work. In that case, use the Option A mock or tell the user they need a proxy backend.

### Option C: Use Agent-Side LLM Capabilities to Generate Mock Data

If this is only for a local demo, you can temporarily call the current agent session's LLM capability (or a multi-model skill the user installed) to generate mock response data, then hardcode that data into the HTML. The HTML then has no runtime dependency on any API.

## Typical HTML Starter Template

Copy this template as the skeleton for a React prototype:

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Your Prototype Name</title>

  <!-- React + Babel pinned -->
  <script src="https://unpkg.com/react@18.3.1/umd/react.development.js" integrity="sha384-hD6/rw4ppMLGNu3tX5cjIb+uRZ7UkRJ6BPkLpg4hAu/6onKUg4lLsHAs9EBPT82L" crossorigin="anonymous"></script>
  <script src="https://unpkg.com/react-dom@18.3.1/umd/react-dom.development.js" integrity="sha384-u6aeetuaXnQ38mYT8rp6sbXaQe3NL9t+IBXmnYxwkUI2Hw4bsp2Wvmx4yRQF1uAm" crossorigin="anonymous"></script>
  <script src="https://unpkg.com/@babel/standalone@7.29.0/babel.min.js" integrity="sha384-m08KidiNqLdpJqLq95G/LEi8Qvjl/xUYll3QILypMoQ65QorJ9Lvtp2RXYGBFj1y" crossorigin="anonymous"></script>

  <style>
    * { box-sizing: border-box; margin: 0; padding: 0; }
    html, body { height: 100%; width: 100%; }
    body {
      font-family: -apple-system, 'SF Pro Text', sans-serif;
      background: #FAFAFA;
      color: #1A1A1A;
    }
    #root { min-height: 100vh; }
  </style>
</head>
<body>
  <div id="root"></div>

  <!-- Your component files -->
  <script type="text/babel" src="components.jsx"></script>

  <!-- Main entry -->
  <script type="text/babel">
    const { useState, useEffect } = React;

    function App() {
      return (
        <div style={{padding: 40}}>
          <h1>Hello</h1>
        </div>
      );
    }

    const root = ReactDOM.createRoot(document.getElementById('root'));
    root.render(<App />);
  </script>
</body>
</html>
```

## Common Errors and Fixes

**`styles is not defined` or `Cannot read property 'button' of undefined`**
→ You defined `const styles` in one file and another file overwrote it. Rename each one with a specific name.

**`Terminal is not defined`**
→ Scopes do not connect across files. Add `Object.assign(window, {Terminal})` at the end of the file that defines `Terminal`.

**The whole page is blank, with no console error**
→ This is usually a JSX syntax error that Babel did not report clearly in the console. Temporarily replace `babel.min.js` with the unminified `babel.js` for clearer errors.

**`ReactDOM.createRoot is not a function`**
→ Wrong version. Confirm you are using `react-dom@18.3.1` (not 17 or another version).

**`Objects are not valid as a React child`**
→ You rendered an object instead of JSX/a string. Usually `{someObj}` should be `{someObj.name}`.

## How to Split Large Projects Into Files

**Single files over 1000 lines** are hard to maintain. Split them like this:

```
project/
├── index.html
├── src/
│   ├── primitives.jsx      # Base elements: Button, Card, Badge...
│   ├── components.jsx      # Business components: UserCard, PostList...
│   ├── pages/
│   │   ├── home.jsx        # Home page
│   │   ├── detail.jsx      # Detail page
│   │   └── settings.jsx    # Settings page
│   ├── router.jsx          # Simple routing (React state switching)
│   └── app.jsx             # Entry component
└── data.js                 # Mock data
```

Load them in HTML in order:
```html
<script type="text/babel" src="src/primitives.jsx"></script>
<script type="text/babel" src="src/components.jsx"></script>
<script type="text/babel" src="src/pages/home.jsx"></script>
<script type="text/babel" src="src/pages/detail.jsx"></script>
<script type="text/babel" src="src/pages/settings.jsx"></script>
<script type="text/babel" src="src/router.jsx"></script>
<script type="text/babel" src="src/app.jsx"></script>
```

At the **end of every file**, use `Object.assign(window, {...})` to export anything that must be shared.
