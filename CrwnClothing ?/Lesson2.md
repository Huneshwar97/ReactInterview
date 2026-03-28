# 📘 CRWN Clothing v2 — Lesson 2 Notes

**Topic:** First Component, Homepage & SCSS Setup
**Branch:** `lesson-2`
**Tag:** Components + Styling

---

## 📌 What This Lesson Covers

Lesson 2 is about three things:
1. Creating your **first custom React component**
2. Building the **Homepage** component
3. Setting up **SCSS** (Sass) for styling instead of plain CSS

No routing yet. No state. Just components and styles.

---

## 📦 New Dependency — node-sass

```json
{
  "dependencies": {
    "node-sass": "^4.11.0",   // ← NEW
    "react": "^17.0.1",
    "react-dom": "^16.8.6",
    "react-scripts": "2.1.8"
  }
}
```

```bash
npm install node-sass
```

**What node-sass does:**
- It is the **compiler** that converts `.scss` files into regular CSS browsers understand
- Browsers don't know SCSS — node-sass sits inside webpack and does the translation at build time
- Without it, renaming a file to `.scss` throws an error
- With it, webpack automatically processes all `.scss` files

```
.scss file  →  node-sass compiler  →  regular .css  →  browser
```

---

## 📁 Folder Structure at Lesson 2

```
src/
├── App.js
├── App.scss                           ← renamed from App.css
├── index.js
├── index.css
└── components/
    └── homepage/
        ├── homepage.component.jsx     ← NEW
        └── homepage.styles.scss       ← NEW
```

---

## 🔑 Key Concept 1 — File Naming Convention

Every component gets its **own folder** with two files:

| File | Purpose |
|------|---------|
| `homepage.component.jsx` | The React component logic |
| `homepage.styles.scss` | The component's styles |

**Why this convention?**

This is called **co-location** — everything related to one component lives together:
- Need to delete a feature? Delete the whole folder — nothing left behind
- Need to find the styles? Always in the same folder, never hunting
- Scales to 50+ components without becoming a mess
- At FAANG this is called **feature-based folder structure**

---

## 🔑 Key Concept 2 — Creating Your First Component

```jsx
// homepage.component.jsx
import './homepage.styles.scss';

const Homepage = () => {
  return (
    <div className='home-page'>
      <h1>Crown Clothing</h1>
    </div>
  );
};

export default Homepage;
```

**Three things happening here:**
1. Import the SCSS file — webpack bundles styles with the component
2. Define a functional component — a function that returns JSX
3. `export default` — makes the component importable in other files

---

## 🔑 Key Concept 3 — Import & Use Pattern (Used Every Lesson)

Two steps to use a component in another file:

```jsx
// App.js

// Step 1: Import it
import Homepage from './components/homepage/homepage.component';

// Step 2: Use it like a tag in JSX
const App = () => {
  return (
    <div>
      <Homepage />
    </div>
  );
};

export default App;
```

**What happens if you forget the import?**
- App.js gets `undefined` instead of the component
- React throws: `Element type is invalid` error
- Nothing renders

> This import → use pattern repeats hundreds of times throughout the course. Drill it.

---

## 🔑 Key Concept 4 — className not class

```jsx
// ❌ WRONG
<div class="home-page">

// ✅ CORRECT
<div className="home-page">
```

**Why?** `class` is a **reserved keyword in JavaScript** (used for ES6 classes). Since JSX is JavaScript, using `class` as an attribute confuses the JS parser. Same reason `for` → `htmlFor` on label elements.

---

## 🔑 Key Concept 5 — export default vs named export

```jsx
// export default — no curly braces, any name works on import
export default Homepage;
import Homepage from '...';     // ✅
import Whatever from '...';     // ✅ name doesn't matter

// named export — curly braces required, name must match exactly
export { Homepage };
import { Homepage } from '...'; // ✅
import { Whatever } from '...'; // ❌ wrong name — will be undefined
```

The course uses `export default` for all components.

**If you forget export default:**
- The import gets `undefined`
- You get `Element type is invalid` in the browser
- Nothing renders

---

## 🔑 Key Concept 6 — SCSS Advantages over Plain CSS

### 1. Nesting — mirrors your HTML structure

```scss
// Plain CSS — repeat parent selector every time
.home-page { display: flex; }
.home-page h1 { font-size: 32px; }
.home-page .subtitle { color: grey; }

// SCSS — nest child selectors inside parent
.home-page {
  display: flex;

  h1 {
    font-size: 32px;       // targets .home-page h1
  }

  .subtitle {
    color: grey;           // targets .home-page .subtitle
  }
}
```

### 2. Variables — single source of truth

```scss
// Define once at top
$primary-color: #4a4a4a;
$font-large: 32px;

// Use everywhere
.title {
  color: $primary-color;
  font-size: $font-large;
}
```

Change the value in one place → updates everywhere. No hunting through files.

> Both features compile down to regular CSS at build time — browsers never see SCSS.

---

## 🧠 Quiz Results (Self-Assessment)

| # | Question | Score | Gap to Fix |
|---|----------|-------|------------|
| 1 | What does node-sass do? | 3/6 | It's the compiler — converts .scss to .css. SCSS is syntax, node-sass is the tool |
| 2 | File naming convention | 4/6 | Co-location: all component files in one folder for easy delete/find/scale |
| 3 | className vs class | 6/6 | ✅ Perfect — class is a reserved JS keyword |
| 4 | Two SCSS advantages | 3/6 | Nesting ✅ + Variables ❌ (missed variables) |
| 5 | How App.js uses Homepage | 0/6 | Import at top → use as JSX tag. Without import = undefined = error |
| 6 | export default | 6/6 | ✅ Perfect — ES6 module pattern, forget it = can't import |

**Total: 22/36**

**Strongest:** JS fundamentals (className, export default)
**Weakest:** node-sass purpose, import/use pattern — drill these

---

## ✅ Lesson 2 Checklist

- [ ] `git checkout lesson-2`
- [ ] `npm install` — picks up node-sass
- [ ] Rename `App.css` → `App.scss`, update import in `App.js`
- [ ] Create `src/components/homepage/homepage.component.jsx`
- [ ] Create `src/components/homepage/homepage.styles.scss`
- [ ] Write SCSS with nesting inside the styles file
- [ ] Import `Homepage` in `App.js` and render it
- [ ] Confirm page renders on `localhost:3000`

---

## 💡 Lesson 1 → Lesson 2 Changes

| | Lesson 1 | Lesson 2 |
|---|---|---|
| Styling | Plain CSS | SCSS (node-sass) |
| Components | Only App.js | App.js + Homepage |
| Folder structure | Flat `src/` | `src/components/homepage/` |
| File naming | `App.js` | `homepage.component.jsx` |
| New concepts | CRA setup, mounting | Components, SCSS, import/export |

---

*Notes by Huneshwar Yadav*
