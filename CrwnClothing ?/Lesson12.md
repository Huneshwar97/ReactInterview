# 📘 CRWN Clothing v2 — Lesson 12 Notes

---

# 🔹 Lesson 12 — Products Context, Shop Route & ProductCard

**Tag:** Products + Shop UI | **Branch:** `lesson-12`

---

## 📌 What This Lesson Covers

Lesson 12 brings the Shop page to life. A second Context (`ProductsContext`) is added for product data. A `shop-data.json` file provides static product data imported directly via webpack. The stub `Shop` component is replaced with a real route that reads from Context and renders a `ProductCard` grid. CSS Grid powers the 4-column layout. The "Add to cart" button uses pure CSS hover — no React state needed.

---

## 📁 New & Changed Files

```
src/
  contexts/
    products.context.jsx          ← NEW — ProductsContext + ProductsProvider
  routes/
    shop/
      shop.component.jsx          ← NEW — real Shop page (was a placeholder)
      shop.styles.scss            ← NEW — CSS Grid layout
  components/
    product-card/
      product-card.component.jsx  ← NEW — individual product card
      product-card.styles.scss    ← NEW — hover button effect
  shop-data.json                  ← NEW — static product data
  index.js                        ← UPDATED — ProductsProvider added
  App.js                          ← UPDATED — Shop imported from routes/
```

---

## 🔗 Full Component & Provider Tree

```
BrowserRouter
  └── UserProvider
        └── ProductsProvider        ← NEW
              └── App
                    └── Routes
                          └── Route '/'  →  Navigation
                                ├── Route index  →  Home
                                ├── Route 'auth' →  Authentication
                                └── Route 'shop' →  Shop    ← REAL NOW
                                                      └── ProductCard ×9
```

---

## 🔑 Key Concepts

### 1. ProductsContext — Second Context, Minimal Exposure

```js
// products.context.jsx
import { createContext, useState } from 'react';
import PRODUCTS from '../shop-data.json';

export const ProductsContext = createContext({ products: [] });

export const ProductsProvider = ({ children }) => {
  const [products, setProducts] = useState(PRODUCTS);
  const value = { products };   // ← setProducts intentionally NOT exposed

  return (
    <ProductsContext.Provider value={value}>
      {children}
    </ProductsContext.Provider>
  );
};
```

**Why `setProducts` is NOT in `value`:**

This is the **Principle of Least Privilege** — only expose what consumers actually need. Exposing `setProducts` would let any component wipe the catalog with `setProducts([])`. By keeping it private, mutations can only happen through controlled functions you explicitly choose to expose.

| Exposed in value | Not exposed |
|-----------------|-------------|
| `products` — read-only access | `setProducts` — internal implementation |

> 🔥 **FAANG pattern name:** Encapsulation — hiding implementation details, only exposing a clean interface. When the app needs filtering/sorting, expose a specific `filterProducts` function, not the raw setter.

---

### 2. JSON Data Import — webpack Module System

```js
import PRODUCTS from '../shop-data.json';
```

webpack treats JSON files as modules. When it sees this import it:
1. Reads `shop-data.json` at **build time**
2. Calls `JSON.parse()` internally
3. Injects the resulting JS array directly into the bundle

**No `fetch()`, no async, no loading state needed.**

| Approach | When data loads | Network request | Bundle impact |
|----------|----------------|-----------------|---------------|
| JSON import | Build time | ❌ None | ✅ Included in bundle |
| `fetch()` / Firestore | Runtime | ✅ Yes | ❌ Not bundled |

> 🔥 **Trade-off:** Static JSON is baked into the bundle — faster first load but data can't update without a rebuild. Large catalogs (1MB+) should always use runtime fetching. This JSON is temporary — later lessons replace it with Firestore.

---

### 3. Shop Route — Real Implementation

```jsx
// shop.component.jsx
const Shop = () => {
  const { products } = useContext(ProductsContext);

  return (
    <div className='products-container'>
      {products.map((product) => (
        <ProductCard key={product.id} product={product} />
      ))}
    </div>
  );
};
```

The old `const Shop = () => <h1>I am the shop page</h1>` placeholder is replaced. Shop now:
- Reads `products` from `ProductsContext` via `useContext`
- Maps over products to render one `ProductCard` per item
- Uses `product.id` as the key (stable, unique — correct pattern)

---

### 4. ProductCard — Pure Presentational Component

```jsx
const ProductCard = ({ product }) => {
  const { name, price, imageUrl } = product;
  return (
    <div className='product-card-container'>
      <img src={imageUrl} alt={`${name}`} />
      <div className='footer'>
        <span className='name'>{name}</span>
        <span className='price'>{price}</span>
      </div>
      <Button buttonType='inverted'>Add to card</Button>
    </div>
  );
};
```

- Pure presentational — receives one `product` prop, renders UI
- Destructures `name`, `price`, `imageUrl` from product object
- Reuses the `Button` component with `buttonType='inverted'`

> ⚠️ **Real bug in lesson code:** `Add to card` should be `Add to cart` — typo in the button text. Common real-world lesson: always review copy!

---

### 5. CSS Hover — Pure CSS Button Reveal

```scss
.product-card-container {
  button {
    display: none;         // ← hidden by default
    position: absolute;
    top: 255px;
    opacity: 0.7;
  }

  &:hover {
    img {
      opacity: 0.8;        // ← image dims on hover
    }
    button {
      display: flex;       // ← button appears on hover
      opacity: 0.85;
    }
  }
}
```

**How it works:** The CSS `&:hover button` selector means "when the parent `.product-card-container` is hovered, apply these styles to its child `button`." The child responds to the parent's hover state — pure CSS, zero JavaScript.

**CSS hover vs React state hover:**

| | Pure CSS (this lesson) | React useState |
|--|----------------------|----------------|
| Re-renders on hover | ❌ Zero | ✅ Every mousemove |
| Performance (100 cards) | ✅ Browser-native | ❌ 100s of re-renders |
| Code complexity | ✅ 3 lines of CSS | ❌ `useState` + `onMouseEnter` + `onMouseLeave` |
| When to use | Visual-only changes | Logic-driven changes |

> 🔥 **The rule:** CSS for visual-only changes (hover, focus, active). React state for logic-driven changes (add to cart, modal open/close, form validation).
> 🔥 **Performance trap:** `useState` for hover on 100 product cards = hundreds of re-renders on every mouse movement. CSS hover has zero JavaScript cost — the browser handles it natively in the rendering engine.

---

### 6. CSS Grid — 4-Column Product Layout

```scss
.products-container {
  display: grid;
  grid-template-columns: repeat(4, 1fr);
  column-gap: 10px;
  row-gap: 50px;
}
```

**Breaking down `repeat(4, 1fr)`:**
- `repeat(4, ...)` — shorthand for writing the same value 4 times
- `1fr` — one **fractional unit** = one equal share of available space after gaps are subtracted
- `repeat(4, 1fr)` → 4 equal columns, always filling the container

**Why `1fr` beats fixed pixels:**

| `repeat(4, 200px)` | `repeat(4, 1fr)` |
|--------------------|-----------------|
| Overflows on small screens | Adapts to any container width |
| Leaves gaps on wide screens | Always fills available space |
| Needs media queries | Naturally flexible |

> 🔥 **Responsive upgrade:** `repeat(auto-fill, minmax(250px, 1fr))` — creates as many columns as fit at minimum 250px, wrapping automatically. No media queries needed.

---

### 7. Provider Composition — Multiple Providers

```jsx
// index.js
<BrowserRouter>
  <UserProvider>
    <ProductsProvider>
      <App />
    </ProductsProvider>
  </UserProvider>
</BrowserRouter>
```

This is **Provider Composition** — stacking multiple Context Providers. Rules:

- Each Provider only needs to wrap components that consume it
- **Order matters** when one Provider needs to read another's context
- Here they're independent — order doesn't functionally matter
- Convention: more global concerns (auth) wrap less global ones (products)

> 🔥 **"Provider hell"** — many nested providers become unreadable. The solution is a custom `AppProviders` wrapper:
> ```jsx
> const AppProviders = ({ children }) => (
>   <UserProvider>
>     <ProductsProvider>
>       {children}
>     </ProductsProvider>
>   </UserProvider>
> );
> ```

---

## 🔄 What Changed from Lesson 11 → Lesson 12

| Area | Lesson 11 | Lesson 12 |
|------|----------|-----------|
| Shop route | `<h1>` placeholder | Real Shop with ProductCards |
| Product data | None | `shop-data.json` imported via webpack |
| Second context | None | `ProductsContext` + `ProductsProvider` |
| ProductCard | None | New reusable component |
| Shop layout | None | CSS Grid 4-column |
| Hover button | None | Pure CSS — `display: none` → `display: flex` |
| Providers in index.js | UserProvider only | UserProvider + ProductsProvider |

---

## ✅ Lesson 12 Checklist

- [ ] `yarn start` → navigate to `/shop` → see 9 product cards in 4-column grid
- [ ] Hover over a card → confirm button appears and image dims
- [ ] Inspect: button is `display: none` by default, `display: flex` on hover
- [ ] Check `shop-data.json` — understand the data shape `{ id, name, imageUrl, price }`
- [ ] Explain why `setProducts` is not in the context value
- [ ] Explain the CSS hover vs React useState trade-off
- [ ] Explain what `repeat(4, 1fr)` means

---

## 💡 Why Not useState for Hover?

If you used React for the hover effect:

```jsx
// ❌ Unnecessary React state for visual-only hover
const ProductCard = ({ product }) => {
  const [isHovered, setIsHovered] = useState(false);

  return (
    <div
      onMouseEnter={() => setIsHovered(true)}
      onMouseLeave={() => setIsHovered(false)}
    >
      <button style={{ display: isHovered ? 'flex' : 'none' }}>
        Add to cart
      </button>
    </div>
  );
};
```

With 100 cards in the catalog, mousing over generates:
- 100 `onMouseEnter` handlers tracking events
- State updates on enter and leave for each card
- Re-renders of the component tree

CSS handles this **entirely in the browser's rendering engine** — zero JavaScript, zero re-renders, infinitely scalable.

---

## 🧠 FAANG Interview Q&A — Lesson 12

### Q1. Why is `setProducts` not exposed in ProductsContext's value?

**Answer:** This is the **Principle of Least Privilege** — only expose what consumers need. Exposing `setProducts` would let any component call `setProducts([])` and wipe the catalog uncontrollably. By keeping it private inside the Provider, all mutations must go through controlled functions the Provider explicitly exposes. When sorting or filtering is needed later, expose a specific `filterProducts` function — not the raw setter.

> 🔥 **Pattern name:** Encapsulation — hiding implementation details, exposing only a clean interface.

---

### Q2. How does `import PRODUCTS from '../shop-data.json'` work?

**Answer:** webpack treats JSON files as modules. At build time it reads the file, calls `JSON.parse()` internally, and injects the resulting JavaScript value directly into the bundle. No `fetch()`, no async operation, no loading state. The data is baked into the bundle — fast first load, but can't update without a rebuild. For small, static data like this it's perfect. For large or frequently-changing data, use runtime fetching from Firestore or an API.

---

### Q3. What is Provider Composition and does nesting order matter?

**Answer:** Provider Composition is stacking multiple Context Providers so the entire app has access to multiple global states. Order matters when one Provider needs to consume another's context — if `ProductsProvider` needed `currentUser` from `UserContext`, `UserProvider` must be the outer wrapper. Here they're independent so order doesn't functionally matter, but by convention more global concerns (auth) wrap less global ones (products).

> 🔥 **Provider hell solution:** Custom `AppProviders` wrapper component — clean and readable.

---

### Q4. What does `repeat(4, 1fr)` mean in CSS Grid?

**Answer:** `repeat(4, 1fr)` creates 4 equal columns, each taking one fractional unit of available space after gaps are subtracted. `1fr` is better than fixed pixels because it adapts to any container width — 4 columns always fill the available space regardless of screen size. With `px` columns you'd overflow on small screens or have gaps on wide ones. `repeat()` is the DRY principle applied to CSS — write the pattern once.

> 🔥 **Responsive upgrade:** `repeat(auto-fill, minmax(250px, 1fr))` — auto-creates columns, no media queries needed.

---

### Q5. ⭐ Hard: CSS hover vs React useState for hover interactions

**Answer:** The button uses `display: none` by default and `display: flex` inside the CSS `:hover` pseudo-class on the parent — pure CSS, zero JavaScript. The parent's hover state cascades to the child button via `&:hover button`.

**React useState approach would be:**
```jsx
const [isHovered, setIsHovered] = useState(false);
// onMouseEnter/onMouseLeave handlers, state update, re-render
```

**Trade-off:**
- CSS hover: browser-native, zero re-renders, zero JavaScript cost — perfect for visual-only changes
- React useState hover: adds re-renders on every mouse event — with 100 product cards this is hundreds of unnecessary re-renders

**The rule:** CSS for visual-only changes (hover, focus, active states). React state for logic-driven changes (add to cart, modal, form validation).

> 🔥 **Performance impact:** `useState` hover on 100 cards = hundreds of re-renders per mouse movement. CSS hover cost = zero JS execution.

---

## 📊 Quiz Results

| # | Question | Score | Gap to Fix |
|---|----------|-------|------------|
| Q1 | setProducts not exposed — Principle of Least Privilege | ✅ Correct | Bonus: Encapsulation pattern name |
| Q2 | JSON import via webpack | ✅ Correct | Bonus: data baked into bundle — trade-off for large catalogs |
| Q3 | Provider Composition — nesting order | ✅ Correct | Bonus: Provider hell solution with AppProviders wrapper |
| Q4 | `repeat(4, 1fr)` — CSS Grid | ✅ Correct | Bonus: `auto-fill` + `minmax` for responsive grids |
| Q5 | CSS hover vs React useState | ❌ Missed | CSS `display: none` → `display: flex` on hover — zero JS, zero re-renders. React useState for hover = performance trap on large lists |

**Total: 4/5** — Strong. Key gap: CSS hover is pure CSS, not useState. The rule: CSS for visual-only, React state for logic-driven changes.
