# 📘 CRWN Clothing v2 — Lesson 4 Notes

---

# 🔹 Lesson 4 — Component Hierarchy & Data Flow

**Tag:** Architecture | **Branch:** `lesson-4`

---

## 📌 What This Lesson Covers

Lesson 4 introduces a critical architectural upgrade: a **middle-layer component** called `Directory` is added between `App` and `CategoryItem`. Real images are now loaded via URLs, SCSS replaces plain CSS globally, and `sass` is added as a proper dependency. The goal: understand multi-level component trees, prop drilling, and why component hierarchy matters.

---

## 📁 Folder Structure

```
src/
  components/
    category-item/
      category-item.component.jsx     ← unchanged from lesson 3
      category-item.styles.scss       ← unchanged from lesson 3
    directory/
      directory.component.jsx         ← NEW — middle layer
      directory.styles.scss           ← NEW — directory styles
  App.js                              ← slimmed down, delegates to Directory
  index.js                            ← now imports index.scss (not index.css)
  index.scss                          ← NEW — global styles as SCSS
public/
  index.html
```

---

## 📦 New Dependency: `sass`

```json
{
  "dependencies": {
    "sass": "1.49.7"   ← NEW
  }
}
```

In Lesson 3, SCSS files were used but `sass` wasn't listed as a dependency — CRA included it internally. Now it's **explicit**. This matters because:

- `sass` is the official Dart Sass compiler
- It compiles `.scss` files to plain CSS that browsers understand
- Without it, importing `.scss` files would throw a build error
- `react-scripts` detects `sass` in dependencies and enables SCSS support automatically

---

## 🔗 The New Component Chain

```
App.js
  ↓ passes categories[] as prop
Directory
  ↓ maps over categories, passes each as prop
CategoryItem
  ↓ renders one card (imageUrl + title)
```

**Lesson 2:**  App → renders cards directly
**Lesson 3:**  App → CategoryItem
**Lesson 4:**  App → Directory → CategoryItem   ← NEW middle layer

---

## 🔑 Key Concepts

### 1. The Directory Component — Middle Layer

```jsx
// directory.component.jsx
const Directory = ({ categories }) => {
  return (
    <div className='directory-container'>
      {categories.map((category) => (
        <CategoryItem key={category.id} category={category} />
      ))}
    </div>
  );
};
```

`Directory` does exactly two things:
1. Receives `categories` from `App` via props
2. Renders a `CategoryItem` for each one

The `.map()` logic that used to live in `App` has been moved here. `App` is now clean:

```jsx
// App.js
return <Directory categories={categories} />;
```

This is the **Separation of Concerns** principle — App owns data, Directory owns layout, CategoryItem owns card UI.

---

### 2. Prop Drilling — Passing Data Through Layers

```
App.js
  categories = [{ id, title, imageUrl }, ...]
       ↓  prop: categories
Directory
       ↓  prop: category (single object)
CategoryItem
  uses imageUrl, title
```

This is called **prop drilling** — passing data through intermediate components that don't use it themselves (Directory doesn't use `imageUrl` — it just passes `category` down). In small apps this is fine. In large apps this becomes painful and is solved later with **Context API** or **Redux**.

---

### 3. Real Images via imageUrl

```js
// App.js — categories now have real image URLs
{
  id: 1,
  title: 'hats',
  imageUrl: 'https://i.ibb.co/cvpntL1/hats.png',
}
```

```jsx
// CategoryItem — uses it as inline style
style={{ backgroundImage: `url(${imageUrl})` }}
```

The images are hosted externally (imgbb CDN). This is the first time the app looks like a real e-commerce site. The pattern — store URL in data, render dynamically — is exactly how production apps work with databases.

---

### 4. SCSS Replaces CSS Globally

`index.css` is gone. `index.scss` takes over as the global stylesheet:

```scss
/* index.scss */
body {
  margin: 0;
  font-family: 'Open Sans Condensed', sans-serif;
}
```

```js
// index.js — now imports .scss instead of .css
import './index.scss';
```

Every component now uses `.scss` files. The benefits:
| Plain CSS | SCSS |
|-----------|------|
| No nesting | Nesting with `&` |
| No variables | `$primary-color: #333` |
| No mixins | Reusable style blocks |
| Repetitive selectors | Write parent once |

---

### 5. yarn.lock — New Package Manager

Lesson 4 ships `yarn.lock` instead of `package-lock.json`. This means the project switched from `npm` to **Yarn**:

| | npm | Yarn |
|---|-----|------|
| Lock file | package-lock.json | yarn.lock |
| Install command | `npm install` | `yarn` |
| Run scripts | `npm start` | `yarn start` |
| Speed | Slower | Faster (parallel installs) |

Both do the same job — lock exact dependency versions for reproducible installs. Use whichever matches the lock file in the project.

---

## 🧠 Component Responsibility Table

| Component | Owns Data? | Renders List? | Renders Card? |
|-----------|-----------|---------------|---------------|
| App | ✅ Yes — categories[] | ❌ No | ❌ No |
| Directory | ❌ No — receives via props | ✅ Yes — .map() | ❌ No |
| CategoryItem | ❌ No — receives via props | ❌ No | ✅ Yes |

This is the **ideal separation** for a component tree. Each component has exactly one job.

---

## 🔄 What Changed from Lesson 3 → Lesson 4

| Area | Lesson 3 | Lesson 4 |
|------|----------|----------|
| Component layers | App → CategoryItem | App → Directory → CategoryItem |
| .map() lives in | App.js | directory.component.jsx |
| imageUrl in data | ❌ Missing | ✅ Real CDN URLs |
| Global styles | index.css | index.scss |
| sass dependency | Implicit (CRA internal) | ✅ Explicit in package.json |
| Package manager | npm (package-lock.json) | Yarn (yarn.lock) |

---

## ✅ Lesson 4 Checklist

- [ ] Checkout `lesson-4` branch
- [ ] Run `yarn` (not `npm install`) — uses yarn.lock
- [ ] `yarn start` → see real category images on screen
- [ ] Trace the full data flow: App → Directory → CategoryItem
- [ ] Understand why .map() moved from App to Directory
- [ ] Explain prop drilling to yourself out loud
- [ ] Identify what problem prop drilling causes at scale

---

## 💡 Why Add a Directory Layer?

Without `Directory`, `App` would need to know about layout (flex, wrap, spacing) AND data. By extracting `Directory`:

- `App` stays data-focused — easy to swap data source later (API, Redux, Firebase)
- `Directory` can be reused with any list of categories anywhere in the app
- `CategoryItem` stays purely presentational — no awareness of lists or data sources

This three-layer pattern (Data Owner → List Renderer → Item Renderer) appears in almost every real React codebase at FAANG companies.

---

## 🧠 FAANG Interview Q&A — Lesson 4

### Q1. What is the exact component hierarchy and why was Directory introduced?

**Answer:** App → Directory → CategoryItem.

Directory was extracted to follow the **Single Responsibility Principle** — App should only own data, not layout logic. Moving `.map()` into Directory means App can later swap its data source (Firebase, Redux, API) without touching any rendering logic. Directory also becomes reusable — it can be dropped anywhere in the app with any list of categories.

> 🔥 **Pattern name:** Container → List → Item — a standard React architecture pattern used at scale.

---

### Q2. What is Prop Drilling and what problem does it cause at scale?

**Answer:** Prop drilling is when data is passed through intermediate components that don't actually need it — they just forward it deeper.

Here, Directory receives the full `category` object including `imageUrl`, but never uses `imageUrl` itself — it just passes `category` down to CategoryItem. At scale this becomes a serious problem: if the data shape changes, you have to update every intermediate component in the chain even though they don't care about the data.

```
App.js          → has categories (including imageUrl)
  ↓ passes category
Directory       → doesn't use imageUrl, just forwards it
  ↓ passes category
CategoryItem    → actually uses imageUrl ✅
```

**Solutions:** Context API (built into React) or Redux — both allow deep components to access data directly without threading props through every layer.

> 🔥 **Key interview phrase:** *"Prop drilling creates tight coupling between components that shouldn't know about each other."*

> ⚠️ **Common wrong answer:** "Directory stores imageUrl in its own state" — state is internal data a component manages itself. Props are external data passed in. These are completely different concepts.

---

### Q3. Why was `sass` added explicitly to package.json in Lesson 4?

**Answer:** CRA internally bundled sass in earlier versions, so SCSS worked even without listing it in package.json. But relying on implicit/transitive dependencies is fragile — if CRA changes its internal deps, your build silently breaks.

Listing `sass` explicitly:
- Pins the exact version (`1.49.7`)
- Makes the dependency graph transparent
- Ensures any machine running `yarn` gets the correct compiler

> 🔥 **Bonus term:** Implicit dependencies are called **phantom dependencies** — a real problem in monorepos at FAANG scale.

---

### Q4. What is the core purpose of a lock file and why must you never mix npm and yarn?

**Answer:** A lock file records the exact resolved version of every dependency and sub-dependency so that `yarn` on any machine — developer laptop, CI server, production container — installs byte-for-byte identical packages.

You must never mix `npm install` and `yarn` because:
- They generate **different lock files** (`package-lock.json` vs `yarn.lock`)
- Running both corrupts the lock file with potentially different resolved versions
- This creates "works on my machine" bugs that are nearly impossible to debug

**Rule:** Pick one package manager per project and commit its lock file. Never run the other.

> 🔥 **FAANG context:** At Meta (who created Yarn), lock file integrity is enforced by CI — the build fails if the lock file is out of sync with package.json.

---

### Q5. ⭐ Hard: Explain `style={{ backgroundImage: \`url(${imageUrl})\` }}` — every part

**Answer:**

```jsx
style={{                          // outer {} = JSX expression escape (enter JS land)
  backgroundImage:                // camelCase JS property (CSS: background-image)
    `url(${imageUrl})`            // template literal — injects imageUrl variable
}}                                // inner {} = the style object literal
```

- **Outer `{}`** — escapes from JSX into JavaScript
- **Inner `{}`** — a plain JavaScript object (the style object)
- **`backgroundImage`** — camelCase because JSX maps to `element.style` DOM object, which uses JS property names, not CSS hyphenated names
- **Template literal** — dynamically injects the `imageUrl` string into the CSS `url()` function

**All multi-word CSS properties become camelCase in JSX:**

| CSS | JSX |
|-----|-----|
| `background-image` | `backgroundImage` |
| `font-size` | `fontSize` |
| `z-index` | `zIndex` |
| `border-radius` | `borderRadius` |

> 🔥 **Common bug:** Writing `style="background-image: url(...)"` like HTML throws a React error: *"The style prop expects a mapping from style properties to values, not a string."*

---

## 📊 Quiz Results

| # | Question | Score | Gap to Fix |
|---|----------|-------|------------|
| Q1 | Component hierarchy & why Directory was added | ✅ Correct | — |
| Q2 | Prop drilling pattern & scale problem | ❌ Missed | Prop drilling ≠ storing in state. It means forwarding props through layers that don't need them |
| Q3 | Why sass was added explicitly | ✅ Correct | — |
| Q4 | Lock files & never mixing npm + yarn | ✅ Correct | Remember: mixing corrupts the lock file |
| Q5 | Double braces & camelCase backgroundImage | ✅ Correct | — |

**Total: 4/5** — Strong. Weakest area: **Prop Drilling** — revise before interviews, it comes up every time.
