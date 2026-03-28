# 📘 CRWN Clothing v2 — Lesson 13 Notes

---

# 🔹 Lesson 13 — Cart Context, CartIcon & Dropdown Toggle

**Tag:** Cart UI | **Branch:** `lesson-13`

---

## 📌 What This Lesson Covers

Lesson 13 introduces the shopping cart UI — a third Context (`CartContext`) manages the cart open/closed state globally. A `CartIcon` component toggles the cart. A `CartDropdown` appears/disappears using short-circuit conditional rendering. This lesson teaches the toggle pattern, mount/unmount vs CSS hide, z-index requirements, and why Context is the right tool when sibling components need to share state.

---

## 📁 New & Changed Files

```
src/
  contexts/
    cart.context.jsx              ← NEW — CartContext + CartProvider
  components/
    cart-icon/
      cart-icon.component.jsx     ← NEW — shopping bag icon + item count
      cart-icon.styles.scss       ← NEW
    cart-dropdown/
      cart-dropdown.component.jsx ← NEW — dropdown panel (stub)
      cart-dropdown.styles.scss   ← NEW — absolute positioning + z-index
  assets/
    shopping-bag.svg              ← NEW — cart icon SVG
  routes/navigation/
    navigation.component.jsx      ← UPDATED — CartIcon + CartDropdown + CartContext
  index.js                        ← UPDATED — CartProvider added
```

---

## 🔗 Full Provider Stack — Lesson 13

```jsx
<BrowserRouter>
  <UserProvider>
    <ProductsProvider>
      <CartProvider>        ← NEW
        <App />
      </CartProvider>
    </ProductsProvider>
  </UserProvider>
</BrowserRouter>
```

Three focused Contexts — each owns one slice of global state:

| Context | Owns |
|---------|------|
| `UserContext` | Auth state (`currentUser`) |
| `ProductsContext` | Product catalog (`products[]`) |
| `CartContext` | Cart UI state (`isCartOpen`, cart items later) |

---

## 🔑 Key Concepts

### 1. CartContext — When to Use Context vs Local State

```js
export const CartProvider = ({ children }) => {
  const [isCartOpen, setIsCartOpen] = useState(false);
  const value = { isCartOpen, setIsCartOpen };  // ← BOTH exposed
  return <CartContext.Provider value={value}>{children}</CartContext.Provider>;
};
```

**Unlike `ProductsContext`**, both `isCartOpen` AND `setIsCartOpen` are exposed — because two different components need them:

| Component | Needs |
|-----------|-------|
| `CartIcon` | `setIsCartOpen` — to toggle |
| `Navigation` | `isCartOpen` — to conditionally render `CartDropdown` |

**Decision rule for state location:**

| Situation | Solution |
|-----------|---------|
| Only one component needs it | Local `useState` |
| Parent → child relationship | Lift state to parent, pass as props |
| Sibling components or distant relatives | Context |

> 🔥 **Rule:** If you'd have to prop drill through a component that doesn't care about the value, use Context instead.

---

### 2. Toggle Pattern — and the Stale Closure Risk

```jsx
// CartIcon
const toggleIsCartOpen = () => setIsCartOpen(!isCartOpen);
```

This is the **toggle pattern** — `!currentValue` flips a boolean on each call.

**⚠️ The stale closure risk:**

```js
// ❌ Risky — captures isCartOpen from closure at time of definition
const toggleIsCartOpen = () => setIsCartOpen(!isCartOpen);
// On rapid double-click: both reads see isCartOpen=false → both set true → only one toggle

// ✅ Safe — functional update: React passes the guaranteed latest state
const toggleIsCartOpen = () => setIsCartOpen(prev => !prev);
// Each update always gets the real current value, not a stale snapshot
```

**Why it matters:** If two state updates are batched or triggered rapidly, `!isCartOpen` might read a stale value from when the function closed over it. `prev => !prev` receives the actual latest state from React's queue.

> 🔥 **FAANG interview staple:** "When should you use the functional update form of setState?" — Answer: whenever the new state depends on the current state (`prev => prev + 1`, `prev => !prev`, `prev => [...prev, item]`).

---

### 3. Conditional Rendering — `{isCartOpen && <CartDropdown />}`

```jsx
// Navigation
const { isCartOpen } = useContext(CartContext);

{isCartOpen && <CartDropdown />}
```

**Short-circuit evaluation:** If `isCartOpen` is `false`, JavaScript stops evaluating — `CartDropdown` is never created. If `true`, it renders.

**Mount/Unmount vs CSS display:none — critical difference:**

| | `{bool && <Component />}` | `display: none` |
|--|--------------------------|----------------|
| DOM presence | ❌ Removed entirely | ✅ Stays in DOM |
| Memory | ✅ Freed on unmount | ❌ Held always |
| Local state | ❌ Lost on unmount | ✅ Preserved |
| Use for | Dropdowns, modals, rare UI | Tabs, complex forms |

> ⚠️ **Common misconception:** `{bool && <Component />}` does NOT just hide the component — it **unmounts** it. `display: none` hides it visually but it stays in the DOM. They are opposites.

**When to choose each:**
- Cart dropdown (shown rarely, simple) → `{isCartOpen && <CartDropdown />}` ✅
- Tab content (user switches between them, complex forms) → `display: none` ✅

---

### 4. CartIcon — Shopping Bag Toggle

```jsx
const CartIcon = () => {
  const { isCartOpen, setIsCartOpen } = useContext(CartContext);
  const toggleIsCartOpen = () => setIsCartOpen(prev => !prev);

  return (
    <div className='cart-icon-container' onClick={toggleIsCartOpen}>
      <ShoppingIcon className='shopping-icon' />
      <span className='item-count'>0</span>  {/* hardcoded — wired up in later lessons */}
    </div>
  );
};
```

- SVG imported as React component (`ReactComponent`) — same pattern as `CrwnLogo`
- Item count hardcoded `0` — future lessons connect to real cart items count
- `onClick` on the container `<div>` — entire icon area is clickable

---

### 5. CartDropdown — Absolute Positioning & z-index

```jsx
const CartDropdown = () => (
  <div className='cart-dropdown-container'>
    <div className='cart-items' />       {/* empty stub — items added later */}
    <Button>GO TO CHECKOUT</Button>
  </div>
);
```

```scss
.cart-dropdown-container {
  position: absolute;   // ← taken out of normal flow
  top: 90px;            // ← 90px from top of nearest positioned ancestor
  right: 40px;          // ← 40px from right
  z-index: 5;           // ← appears above other content
  width: 240px;
  height: 340px;
  background-color: white;
  border: 1px solid black;
}
```

**`position: absolute`:**
- Removed from the normal document flow — doesn't push other elements
- Positioned relative to the nearest ancestor with `position` set (not static)
- Without it, the dropdown would push the navbar content down

**`z-index: 5` — requires position to work:**

```
position: static (default) → z-index has NO EFFECT ❌
position: absolute/relative/fixed/sticky → z-index works ✅

z-index: 5 → appears above z-index: 1, 2, 3, 4
z-index: 5 → appears below z-index: 6, 7, 8...
```

> 🔥 **Classic CSS interview question:** "Why isn't my z-index working?" → Almost always because the element has `position: static`. z-index only creates a stacking context for positioned elements.

---

### 6. Context Sync — How Siblings Share State

```jsx
// CartIcon — WRITES to context
const { setIsCartOpen } = useContext(CartContext);
const toggleIsCartOpen = () => setIsCartOpen(prev => !prev);

// Navigation — READS from context
const { isCartOpen } = useContext(CartContext);
{isCartOpen && <CartDropdown />}
```

Both components call `useContext(CartContext)` — they subscribe to the **same context value**. When `CartIcon` updates `isCartOpen`:
1. `CartContext` value changes
2. React re-renders ALL subscribers
3. `Navigation` reads the new `isCartOpen` instantly
4. `CartDropdown` mounts or unmounts

**Zero prop passing. Zero prop drilling. Automatic sync.**

**What if `isCartOpen` was local state in `CartIcon`?**
- `Navigation` has no access to `CartIcon`'s local state
- `CartIcon` is rendered *inside* `Navigation`'s JSX — you'd need `Navigation` to own the state and pass it down as a prop to `CartIcon`
- This means `Navigation` owns cart state it conceptually doesn't care about — wrong ownership
- Context keeps cart state in `CartContext` where it belongs

---

## 🔄 Cart Toggle Flow

```
User clicks CartIcon
  → toggleIsCartOpen()
  → setIsCartOpen(prev => !prev)    false → true
  → CartContext value: { isCartOpen: true, setIsCartOpen }
  → All subscribers re-render:
      CartIcon: re-renders (same icon)
      Navigation: re-renders
        → {true && <CartDropdown />} → CartDropdown MOUNTS
        → Dropdown appears at position: absolute, z-index: 5

User clicks CartIcon again
  → setIsCartOpen(prev => !prev)    true → false
  → Navigation re-renders
        → {false && <CartDropdown />} → CartDropdown UNMOUNTS
        → Dropdown disappears, memory freed
```

---

## 🔄 What Changed from Lesson 12 → Lesson 13

| Area | Lesson 12 | Lesson 13 |
|------|----------|-----------|
| Cart state | None | `isCartOpen` in CartContext |
| Cart UI | None | CartIcon + CartDropdown |
| Navigation | User auth links only | + CartIcon + conditional CartDropdown |
| Provider stack | UserProvider + ProductsProvider | + CartProvider |
| Conditional rendering | Not used | `{isCartOpen && <CartDropdown />}` |
| Toggle pattern | Not used | `setIsCartOpen(prev => !prev)` |

---

## ✅ Lesson 13 Checklist

- [ ] `yarn start` → see shopping bag icon in navbar
- [ ] Click cart icon → dropdown appears
- [ ] Click cart icon again → dropdown disappears
- [ ] Inspect DOM: confirm CartDropdown is removed from DOM when closed (not just hidden)
- [ ] Explain why isCartOpen is in Context not local CartIcon state
- [ ] Explain mount/unmount vs display:none difference
- [ ] Explain why z-index requires position to work
- [ ] Write the safe toggle: `setIsCartOpen(prev => !prev)`

---

## 💡 Why Not Local State for isCartOpen?

If `isCartOpen` was local inside `CartIcon`:

```jsx
// ❌ Wrong approach
const CartIcon = () => {
  const [isCartOpen, setIsCartOpen] = useState(false);
  // Navigation has no idea this exists
};

const Navigation = () => (
  // How does Navigation know to show CartDropdown? It can't!
  {isCartOpen && <CartDropdown />}  // ← isCartOpen is undefined here
);
```

The state would need to be **lifted** to `Navigation` (their common ancestor) and passed down as props — but `Navigation` conceptually shouldn't own cart state. `CartContext` is the right owner — components subscribe to exactly what they need.

---

## 🧠 FAANG Interview Q&A — Lesson 13

### Q1. Why is `isCartOpen` in Context, not local CartIcon state?

**Answer:** `CartIcon` needs to **write** `isCartOpen` (toggle it) and `Navigation` needs to **read** it (render `CartDropdown`). They're siblings — neither is an ancestor of the other — so you can't connect them without prop drilling through `Navigation` which doesn't conceptually own cart state. Context lets both components subscribe independently to the same shared value with no prop chains.

> 🔥 **Decision rule:** Local state for one component. Lift to parent for parent/child. Context for siblings or distant relatives.

---

### Q2. What is the stale closure risk with `!isCartOpen`?

**Answer:** `setIsCartOpen(!isCartOpen)` captures `isCartOpen` from the closure at the time the function is defined. On rapid double-clicks, both reads might see the stale value and produce the wrong result. The safe form is the **functional update**: `setIsCartOpen(prev => !prev)` — React passes the guaranteed latest state as `prev`, bypassing any closure issues.

> 🔥 **Rule:** Use functional update whenever new state depends on current state: `prev => !prev`, `prev => prev + 1`, `prev => [...prev, item]`.

---

### Q3. `{isCartOpen && <CartDropdown />}` vs `display: none` — what's the difference?

**Answer:** `{isCartOpen && <CartDropdown />}` **unmounts** the component when false — it's removed from the DOM entirely, memory freed, local state lost. `display: none` keeps the component in the DOM — invisible but still holding memory and local state. Mount/unmount is better for rarely-shown UI like dropdowns. `display: none` is better for tabs or complex forms where you want to preserve state.

> ⚠️ **Common misconception:** `{bool && <Component />}` does NOT just hide — it fully unmounts.

---

### Q4. Why does z-index require position to work?

**Answer:** Elements with `position: static` (the default) are in the normal document flow and don't participate in stacking contexts — z-index has **no effect** on them. Setting `position` to `absolute`, `relative`, `fixed`, or `sticky` creates a positioned element that participates in stacking. `z-index` then controls the stacking order — higher numbers appear on top. `CartDropdown` needs both: `position: absolute` to be taken out of flow (positioned over content) and `z-index: 5` to appear above other elements.

---

### Q5. ⭐ Hard: How do CartIcon and Navigation stay in sync? What breaks with local state?

**Answer:** Both call `useContext(CartContext)` — they subscribe to the **same context value**. When `CartIcon` calls `setIsCartOpen`, React updates the context and re-renders all subscribers including `Navigation`, which immediately reads the new `isCartOpen` and mounts/unmounts `CartDropdown` automatically. Zero props, zero drilling.

With local state in `CartIcon`: `Navigation` has no access — it can't conditionally render `CartDropdown`. You'd need to lift state to `Navigation` and pass it as a prop to `CartIcon` — making `Navigation` own cart state it doesn't conceptually care about. Context keeps ownership where it belongs.

---

## 📊 Quiz Results

| # | Question | Score | Gap to Fix |
|---|----------|-------|------------|
| Q1 | isCartOpen in Context — sibling sharing | ✅ Correct | — |
| Q2 | Toggle pattern + stale closure risk | ⚠️ Partial | Use `prev => !prev` not `!isCartOpen` — functional update for safety |
| Q3 | Mount/unmount vs display:none | ❌ Missed | **Reversed!** `{bool && ...}` = unmounts. `display: none` = stays in DOM |
| Q4 | z-index requires position | ❌ Missed | z-index has NO effect on `position: static`. Needs absolute/relative/fixed/sticky |
| Q5 | Context sync vs local state | ❌ Missed | Both use `useContext(same context)` → automatic sync. Local state = Navigation can't read it |

**Total: 2/5** — Toughest lesson so far. Four key gaps to fix before next lesson: functional update pattern, mount/unmount vs display:none (reversed!), z-index position requirement, Context sync mechanism.
