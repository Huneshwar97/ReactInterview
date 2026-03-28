# 📘 CRWN Clothing v2 — Lesson 5 Notes

---

# 🔹 Lesson 5 — React Router v6

**Tag:** Routing | **Branch:** `lesson-5`

---

## 📌 What This Lesson Covers

Lesson 5 introduces **React Router DOM v6** — the industry-standard routing library for React. The app gains its first URL-based navigation. `App.js` becomes a **pure router** with zero data and zero UI of its own. A new `routes/` folder separates page-level components from reusable `components/`. This lesson also contains a **real bug** (a typo in an import path) that teaches how webpack resolves modules.

---

## 📁 Folder Structure

```
src/
  routes/                               ← NEW — page-level components tied to URLs
    home/
      home.component.jsx                ← NEW — owns homepage data & layout
  components/                           ← unchanged — reusable UI pieces
    category-item/
      category-item.component.jsx
      category-item.styles.scss
    directory/
      directory.component.jsx
      directory.styles.scss
  App.js                                ← now purely a router, zero data/UI
  index.js                              ← wraps app in BrowserRouter
  index.scss
```

---

## 📦 New Dependency: `react-router-dom`

```json
{
  "dependencies": {
    "react-router-dom": "6"    ← NEW
  }
}
```

React itself has **no routing built in**. `react-router-dom` is the industry-standard library that enables multi-page experiences in a Single Page App — the URL changes but the page never fully reloads. React Router v6 is a major rewrite from v5 with cleaner APIs and all-exact matching by default.

---

## 🔗 Full Component & Route Tree

```
BrowserRouter  (index.js)
  └── App
        └── Routes
              └── Route  path="/"
                    └── Home              ← owns categories data
                          └── Directory
                                └── CategoryItem (×5)
```

---

## 🔑 Key Concepts

### 1. BrowserRouter — The URL Sync Layer

```jsx
// index.js
import { BrowserRouter } from 'react-router-dom';

render(
  <React.StrictMode>
    <BrowserRouter>
      <App />
    </BrowserRouter>
  </React.StrictMode>,
  rootElement
);
```

`BrowserRouter` uses the **HTML5 History API** — specifically `pushState` and `replaceState` — to change the URL bar without triggering a full page reload. It also listens for `popstate` events (back/forward browser buttons) and tells React to re-render the matching components.

**Why it must wrap the entire app in `index.js`:**
- `<Routes>` and `<Route>` inside App.js consume routing **context**
- If `BrowserRouter` isn't an ancestor, there is no context to read — they throw an error
- It must be the outermost wrapper so every component in the tree can access routing info

| Router Type | URL Style | When to Use |
|-------------|-----------|-------------|
| `BrowserRouter` | `/path` | Standard — requires server config for deep links |
| `HashRouter` | `/#/path` | No server config needed — e.g. GitHub Pages |

---

### 2. Routes and Route — Mapping URLs to Components

```jsx
// App.js
import { Routes, Route } from 'react-router-dom';
import Home from './routes/home/home.component';

const App = () => (
  <Routes>
    <Route path='/' index element={<Home />} />
  </Routes>
);
```

| Part | Meaning |
|------|---------|
| `<Routes>` | Smart container — finds the FIRST matching `<Route>` for the current URL |
| `<Route path='/'>` | Match when URL is exactly `/` |
| `index` | Default child route — rendered when parent path matches but no child path does |
| `element={<Home />}` | The component to render when path matches |

**Key v6 improvement:** In React Router v5, you needed `exact` prop to prevent partial matches (`/` matching `/shop`). In v6, **all routes are exact by default** — eliminating a whole class of routing bugs.

---

### 3. App.js — Now a Pure Router

```jsx
// Lesson 4 App.js — had data
const App = () => {
  const categories = [...]; // ← data lived here
  return <Directory categories={categories} />;
};

// Lesson 5 App.js — pure router, zero data/UI
const App = () => (
  <Routes>
    <Route path='/' index element={<Home />} />
  </Routes>
);
```

**Evolution of App.js across lessons:**

```
Lesson 2: App = data + .map() + card UI
Lesson 3: App = data + .map()         (card → CategoryItem)
Lesson 4: App = data                  (.map() → Directory)
Lesson 5: App = routing only          (data → Home) ✅ cleanest
```

App keeps getting slimmer — this is correct architecture. Each lesson strips one more responsibility.

---

### 4. Home Component — Route Takes Over App's Job

```jsx
// home.component.jsx
import Directory from '../../components/directory/directory.component';

const Home = () => {
  const categories = [
    { id: 1, title: 'hats',     imageUrl: 'https://i.ibb.co/cvpntL1/hats.png' },
    { id: 2, title: 'jackets',  imageUrl: 'https://i.ibb.co/px2tCc3/jackets.png' },
    { id: 3, title: 'sneakers', imageUrl: 'https://i.ibb.co/0jqHpnp/sneakers.png' },
    { id: 4, title: 'womens',   imageUrl: 'https://i.ibb.co/GCCdy8t/womens.png' },
    { id: 5, title: 'mens',     imageUrl: 'https://i.ibb.co/R70vBrQ/men.png' },
  ];

  return <Directory categories={categories} />;
};
```

Data that lived in `App.js` in Lesson 4 has moved to `Home`. Each **route owns its own data** — this is the core principle of route-level data ownership.

---

### 5. routes/ vs components/ — Architectural Separation

| | `routes/` | `components/` |
|--|-----------|---------------|
| Tied to a URL? | ✅ Yes — maps 1:1 to a `<Route>` | ❌ No — URL-agnostic |
| Reused across pages? | ❌ No — page-specific | ✅ Yes — used anywhere |
| Owns page data? | ✅ Yes | ❌ No — receives via props |
| Example | `home/`, `shop/`, `checkout/` | `category-item/`, `button/`, `header/` |

**Rule:** If you'd put it in `<Route element={}>`, it belongs in `routes/`. If you'd reuse it across multiple pages, it belongs in `components/`.

---

### 6. 🐛 Real Bug in This Lesson's Code

```js
// App.js — line 3 (WRONG)
import Home from './routes/home/home.componentt'; // ← double 't' typo!

// Correct
import Home from './routes/home/home.component';
```

**What happens:** webpack resolves import paths at **build time**. When it can't find `home.componentt`, it throws:
```
Module not found: Can't resolve './routes/home/home.componentt'
```
The app fails to compile entirely — red error overlay in dev, build failure in CI.

**The macOS vs Linux trap:** On macOS (case-insensitive filesystem), `home.component` and `Home.Component` both resolve. On Linux (where CI and production run), it's **case-sensitive** and crashes. Works locally → breaks in production. This is one of the most common "works on my machine" bugs in React teams.

---

## 🔄 What Changed from Lesson 4 → Lesson 5

| Area | Lesson 4 | Lesson 5 |
|------|----------|----------|
| Routing | None | React Router v6 |
| App.js role | Owns data | Pure router only |
| Data lives in | App.js | home.component.jsx |
| New folder | — | `routes/` |
| BrowserRouter | — | Wraps app in index.js |
| New dependency | sass | react-router-dom |

---

## ✅ Lesson 5 Checklist

- [ ] `yarn` to install — `react-router-dom` is new
- [ ] Fix the typo: `home.componentt` → `home.component` in App.js
- [ ] `yarn start` → app loads at `localhost:3000/`
- [ ] Trace the full tree: BrowserRouter → App → Routes → Route → Home → Directory → CategoryItem
- [ ] Explain why BrowserRouter lives in index.js not App.js
- [ ] Explain the difference between `routes/` and `components/`
- [ ] Add a second route (e.g. `path='/shop'`) and create a dummy Shop component

---

## 💡 Why React Router at All?

Without React Router, navigating between "pages" would require:
- Conditional rendering based on some state variable
- Manual URL management
- No browser back/forward button support
- No deep-linkable URLs (can't share a link to `/shop`)

React Router solves all of this — and the URL becomes the **single source of truth** for what the user is looking at.

> 🔥 **Future lessons:** React Router v6.4+ adds `loader` functions — each route declares its own data fetching. This takes route-level data ownership (introduced here) to its logical conclusion.

---

## 🧠 FAANG Interview Q&A — Lesson 5

### Q1. What does BrowserRouter do and why must it wrap the entire app in index.js?

**Answer:** BrowserRouter uses the **HTML5 History API** — `pushState` and `replaceState` — to change the URL without a full page reload. It listens for `popstate` events (back/forward buttons) and updates React's rendering accordingly.

It must wrap the entire app because `<Routes>` and `<Route>` consume a **routing context** that BrowserRouter provides. If BrowserRouter isn't an ancestor of those components, they have no context and throw an error. Placing it in `index.js` guarantees it's the outermost wrapper in the entire component tree.

> 🔥 **Alternative:** `HashRouter` uses `/#/path` — no server config needed, useful for static hosting like GitHub Pages.

---

### Q2. What is the difference between `<Routes>` and `<Route>` in React Router v6?

**Answer:**
- `<Routes>` is the **smart container** — on every URL change it scans its `<Route>` children and renders only the **first matching** one
- `<Route>` is a **config declaration** — it maps a `path` string to an `element` (component)

```jsx
<Routes>                              {/* scans children, picks first match */}
  <Route path='/' element={<Home />} />     {/* config: '/' → Home */}
  <Route path='/shop' element={<Shop />} /> {/* config: '/shop' → Shop */}
</Routes>
```

**Key v6 change:** All routes are **exact by default** — no more `exact` prop needed. In v5, `path='/'` would match `/shop` unless you added `exact`. In v6 it doesn't — eliminating a whole class of bugs.

> 🔥 **`index` prop:** Marks the default child route — rendered when the parent path matches but no specific child path does. Like `index.html` for a directory.

---

### Q3. Why did categories data move from App.js to home.component.jsx?

**Answer:** App.js should be a **pure router** — its only job is mapping URLs to components. If data lives in App, then App needs to change when the homepage data changes AND when routing changes — two reasons to change = SRP violation.

By moving data to `Home`, each route is **self-contained**: it owns its own data, layout, and logic. This also enables **code splitting** later — `React.lazy()` can load Home's bundle only when the user navigates to `/`, not upfront.

> 🔥 **Future:** This pattern is the foundation of React Router v6.4+ **data loaders** — each route declares a `loader` function that fetches its own data before rendering.

---

### Q4. What is the architectural difference between routes/ and components/?

**Answer:**

| | `routes/` | `components/` |
|--|-----------|---------------|
| Tied to a URL | ✅ Yes | ❌ No |
| Reusable across pages | ❌ No | ✅ Yes |
| Owns page data | ✅ Yes | ❌ No |

`routes/` contains page-level components — each maps 1:1 to a URL, never imported directly by other components. `components/` contains reusable UI — no URL awareness, used anywhere.

**Rule:** If it goes in `<Route element={}>` → `routes/`. If it's reused across pages → `components/`.

> 🔥 **At scale:** Large codebases also add `layouts/` (shared nav+footer frames), `hooks/` (custom hooks), `utils/` (pure functions), and `store/` (Redux/Context).

---

### Q5. ⭐ Hard: What happens when App.js imports `home.componentt` (double 't')?

**Answer:** webpack resolves import paths at **build time**. When it can't find the file, it throws immediately:

```
Module not found: Can't resolve './routes/home/home.componentt'
```

The app **fails to compile entirely** — red error overlay in dev, build failure in CI. There is no runtime — webpack stops before the browser ever runs any code.

**How to debug:**
1. Read the exact error — it shows the full unresolved path
2. Check the actual filename in the filesystem
3. Fix the typo and save — webpack hot-reloads

**The macOS vs Linux trap:** macOS uses a case-insensitive filesystem — `home.component` and `Home.Component` both resolve. Linux (CI servers, production) is **case-sensitive** — mismatched casing compiles locally but crashes in production. This is one of the most common "works on my machine" bugs at FAANG.

> 🔥 **Fix:** Always match import paths exactly to the filename — both typos and casing.

---

## 📊 Quiz Results

| # | Question | Score | Gap to Fix |
|---|----------|-------|------------|
| Q1 | BrowserRouter & HTML5 History API | ✅ Correct | — |
| Q2 | Routes vs Route in v6 | ✅ Correct | Remember: v6 = exact by default, no more `exact` prop |
| Q3 | Data moved to Home — why | ✅ Correct | — |
| Q4 | routes/ vs components/ architecture | ✅ Correct | — |
| Q5 | Import typo — what happens & how to debug | ✅ Correct | Bonus: mention macOS vs Linux case-sensitivity trap |

**Total: 5/5** — Perfect score. Strong understanding of React Router fundamentals and architecture.
