# 📘 CRWN Clothing v2 — Lesson-by-Lesson Notes

---

# 🔹 Lesson 1 — Project Setup & Folder Structure

**Tag:** Foundation | **Branch:** `lesson-1`

---

## 📌 What This Lesson Covers

Lesson 1 is pure setup — no routing, no state, no Firebase. You bootstrap the project using **Create React App**, understand what CRA generates, and run the dev server. The goal: know what every file is for before writing real code.

---

## 📁 Folder Structure

```
crwn-clothing-v2/
├── public/
│   ├── index.html          ← The ONE html page. React mounts here.
│   ├── favicon.ico
│   ├── manifest.json       ← PWA config
│   └── robots.txt
├── src/
│   ├── App.js              ← Root component (entry to your UI)
│   ├── App.css             ← Styles for App component
│   ├── index.js            ← Entry point — mounts App into #root
│   ├── index.css           ← Global styles
│   ├── App.test.js         ← Starter test
│   ├── setupTests.js       ← Configures jest-dom matchers
│   └── reportWebVitals.js  ← Performance metrics (safe to ignore)
├── .gitignore
├── package.json            ← Project manifest & dependencies
└── package-lock.json       ← Locks exact dependency versions
```

---

## 📦 Dependencies at Lesson 1 (Minimal)

```json
{
  "dependencies": {
    "react": "^17.0.2",       // UI library
    "react-dom": "^17.0.2",   // Renders React into browser DOM
    "react-scripts": "5.0.0", // CRA build tool (webpack + babel)
    "web-vitals": "2.1.4"     // Performance metrics
  }
}
```

> Everything else (Router, Firebase, Redux) is added in later lessons.

---

## 🔗 The Mounting Chain

```
public/index.html  →  src/index.js  →  src/App.js
       ↓                   ↓
  <div id="root">    ReactDOM.render(<App />, #root)
```

**In plain English:**
1. Browser loads `public/index.html` — has one empty `<div id="root">`
2. webpack loads `src/index.js` as entry point
3. `index.js` calls `ReactDOM.render()` — injects entire React tree into `#root`
4. Everything you see is React writing into that one div

---

## 🔑 Key Concepts

### public/ vs src/

| | public/ | src/ |
|---|---|---|
| Processed by webpack? | ❌ No — served as-is | ✅ Yes — bundled & optimised |
| What lives here? | index.html, icons, manifest | Components, CSS, JS |
| Can import from here? | No | Yes |

---

### index.js — The Entry Point

```jsx
import React from 'react';
import ReactDOM from 'react-dom';
import './index.css';
import App from './App';

ReactDOM.render(
  <React.StrictMode>
    <App />
  </React.StrictMode>,
  document.getElementById('root')
);
```

> You almost never edit this file.

---

### React.StrictMode

* **Development only** — stripped completely in production
* **Double-invokes** renders and effects to surface unintended side effects
* Warns about deprecated React APIs
* Zero performance cost in production

---

### JSX vs HTML — Key Differences

| HTML | JSX |
|------|-----|
| `class="box"` | `className="box"` |
| `onclick="fn()"` | `onClick={fn}` |
| `for="id"` | `htmlFor="id"` |

```jsx
// JSX you write:
const el = <div className="app">Hello</div>;

// What Babel compiles it to:
const el = React.createElement("div", { className: "app" }, "Hello");
```

---

### react-scripts — What It Gives You

| Tool | Purpose |
|------|---------|
| webpack | Bundles all JS/CSS into optimised output |
| Babel | Transpiles JSX + modern JS to browser JS |
| ESLint | Lints your code for errors |
| Jest | Runs your tests |

**Tradeoff:** Zero config, but customisation requires **ejecting** — which is permanent and exposes all config files.

---

### node_modules & .gitignore

* `npm install` reads `package.json` → downloads everything into `node_modules/`
* `node_modules/` can be **hundreds of MB** with thousands of files
* It's 100% reproducible — never commit it
* `package-lock.json` locks exact versions for reproducible installs across machines

---

## 🧠 Quiz Results (Self-Assessment)

| # | Question | Score | Gap to fix |
|---|----------|-------|------------|
| 1 | public/ vs src/ | 5/6 | Remember: index.html lives in public/ |
| 2 | What index.js does | 3/6 | It mounts React into #root via ReactDOM.render() |
| 3 | Why .gitignore node_modules | 6/6 | ✅ Perfect |
| 4 | React.StrictMode | 4/6 | It catches side effects & deprecated APIs — not syntax errors |
| 5 | JSX vs HTML differences | 4/6 | Remember: className, onClick, htmlFor |
| 6 | react-scripts | 1/6 | It's webpack + Babel + ESLint + Jest — not related to ReactDOM |

**Total: 23/36** — Good foundation. Weakest areas: index.js mounting chain and react-scripts.

---

## ✅ Lesson 1 Checklist

- [ ] Fork & clone the repo
- [ ] `git checkout lesson-1`
- [ ] `npm install`
- [ ] `npm start` → see spinning React logo on localhost:3000
- [ ] Explore every file in public/ and src/
- [ ] Read index.js and trace the mounting chain
- [ ] Edit App.js, save, watch hot-reload update browser
- [ ] Can answer all 6 quiz questions without peeking

---

## 💡 Why Start So Minimal?

The course adds **one tool at a time** — Router → Firebase → Context → Redux. Starting minimal means you feel exactly what each library adds. If everything was installed upfront you'd never understand *why* each piece exists.

---
