# 🚀 React Hooks (FAANG-Level Deep Dive)

💡 Covers **useState, useEffect, useMemo, useCallback, useRef + internals + pitfalls + optimization**

---

# 🔹 1. What are Hooks?

**Answer:**

Hooks are functions that allow functional components to use state and lifecycle features.

---

## 🔍 Why Hooks were introduced?

Before Hooks:

* Class components required for state
* Complex lifecycle methods
* Hard to reuse logic

Hooks solved:
👉 Reusability
👉 Simplicity
👉 Cleaner code

---

# 🔹 2. What are Rules of Hooks?

**Answer:**

1. Only call hooks at top level
2. Only call hooks inside React functions

---

## 🔹 Why?

Because React relies on **call order** to track hooks.

---

## 🔹 Advanced Question

**Q: What happens if we call hooks inside condition?**

👉 Hook order breaks → React crashes / bugs

---

# 🔹 3. useState Deep Dive

**Answer:**

`useState` is used to manage local state.

---

## 🔹 Example

```jsx id="d8k2s1"
const [count, setCount] = useState(0);
```

---

## 🔍 Internals

React stores state in:
👉 **Fiber linked list**

Each render:
👉 React matches hooks by order

---

## 🔹 Advanced Question

**Q: Why state updates are asynchronous?**

Because:

* React batches updates
* Improves performance

---

## 🔹 Functional Update

```js id="g9w2k1"
setCount(prev => prev + 1);
```

👉 Prevents stale state

---

# 🔹 4. useEffect Deep Dive

**Answer:**

Used for side effects.

---

## 🔹 Example

```jsx id="v1k9d2"
useEffect(() => {
  console.log("Mounted");
}, []);
```

---

## 🔍 Types of Effects

### 1. Mount

```js id="e1"
useEffect(() => {}, []);
```

### 2. Update

```js id="e2"
useEffect(() => {}, [count]);
```

### 3. Cleanup

```js id="e3"
useEffect(() => {
  return () => console.log("cleanup");
}, []);
```

---

## 🔹 Execution Order

```id="order1"
Render → Commit → useEffect
```

---

## 🔹 Advanced Question

**Q: Why useEffect runs after render?**

Because:
👉 React keeps render pure
👉 Side effects happen after commit

---

## 🔹 Common Mistake

```js id="mist1"
useEffect(() => {
  setCount(count + 1);
}, [count]);
```

👉 Infinite loop

---

# 🔹 5. Dependency Array Deep Understanding

**Answer:**

Controls when effect runs.

---

## 🔹 Rules

* `[]` → run once
* `[x]` → run when x changes
* no array → run every render

---

## 🔹 Advanced Question

**Q: Why dependency array is needed?**

Because React cannot automatically detect dependencies.

---

# 🔹 6. useMemo Deep Dive

**Answer:**

Memoizes computed value.

---

## 🔹 Example

```js id="m1"
const expensive = useMemo(() => compute(), [data]);
```

---

## 🔹 Why?

Avoid expensive recalculations

---

## 🔹 Advanced Insight

👉 useMemo is for **value caching**, not side effects

---

# 🔹 7. useCallback Deep Dive

**Answer:**

Memoizes functions.

---

## 🔹 Example

```js id="c1"
const handleClick = useCallback(() => {
  console.log("click");
}, []);
```

---

## 🔹 Why?

Prevent unnecessary re-renders

---

## 🔹 Advanced Insight

👉 Functions get recreated every render
👉 useCallback stabilizes reference

---

# 🔹 8. useMemo vs useCallback

| Hook        | Purpose        |
| ----------- | -------------- |
| useMemo     | Cache value    |
| useCallback | Cache function |

---

# 🔹 9. useRef Deep Dive

**Answer:**

Stores mutable value without causing re-render.

---

## 🔹 Example

```js id="r1"
const ref = useRef(0);
```

---

## 🔹 Use cases

* DOM access
* Store previous values
* Avoid re-render

---

## 🔹 Advanced Question

**Q: Why useRef doesn’t trigger re-render?**

Because:
👉 It does not affect React state

---

# 🔹 10. useRef vs useState

| Feature   | useRef | useState |
| --------- | ------ | -------- |
| Re-render | ❌ No   | ✔️ Yes   |
| Mutable   | ✔️ Yes | ❌ No     |

---

# 🔹 11. useEffect vs useLayoutEffect

**Answer:**

| Hook            | Timing       |
| --------------- | ------------ |
| useEffect       | After paint  |
| useLayoutEffect | Before paint |

---

## 🔹 Insight

👉 useLayoutEffect blocks UI → use carefully

---

# 🔹 12. Common Performance Issues

* Unnecessary re-renders
* Recreating functions
* Heavy calculations

---

## 🔹 Solutions

* useMemo
* useCallback
* React.memo

---

# 🔹 13. Custom Hooks

**Answer:**

Reusable logic extracted into functions.

---

## 🔹 Example

```js id="custom1"
function useCounter() {
  const [count, setCount] = useState(0);
  return { count, setCount };
}
```

---

## 🔹 Why important?

* Code reuse
* Clean architecture

---

# 🔹 14. Common Mistakes

* Missing dependencies
* Overusing useMemo
* Infinite loops
* Misusing useRef

---

# 🔹 15. Advanced Interview Questions

**Q1: Why React Hooks rely on order?**
👉 Linked list structure

---

**Q2: When to use useMemo vs useCallback?**
👉 Value vs function

---

**Q3: How to prevent re-renders?**
👉 Memoization

---

**Q4: Why useEffect causes bugs?**
👉 Incorrect dependencies

---

# 🎯 FAANG-Level Insight

Top candidates:

* Understand hook internals
* Avoid unnecessary re-renders
* Know trade-offs
* Use hooks correctly

---

# ⚡ Quick Revision

* useState = state
* useEffect = side effects
* useMemo = cache value
* useCallback = cache function
* useRef = mutable value

---

# ⭐ Author

Huneshwar Yadav



# 🚀 React Hooks: State, Lifecycle & Performance Optimization

---

## 🔹 1. The Hook Philosophy
**Mental Model:** Hooks allow you to "hook into" React's internal state and lifecycle from functional components. They rely on a **Linked List** structure internally; therefore, they must always be called in the same order (never inside loops or conditions).



---

## 🔹 2. State & Reference Management
* **`useState`:** The foundation of dynamic UI. Updates are asynchronous and batched. Always use the functional update `setCount(prev => prev + 1)` to avoid stale closures.
* **`useRef`:** A "stealth" hook. It persists values between renders but **does not trigger a re-render** when the value changes. Perfect for DOM access or storing previous state values.

---

## 🔹 3. The Side Effect Sandbox (`useEffect`)
`useEffect` handles the "outside world" (API calls, subscriptions).
* **Timing:** It runs **after** the render is committed to the screen (asynchronous).
* **Cleanup:** Always return a cleanup function to prevent memory leaks (e.g., `clearInterval` or `socket.disconnect`).
* **`useLayoutEffect`:** The synchronous version. It runs **before** the browser paints. Use it only for measuring DOM elements to prevent UI flickering.



---

## 🔹 4. Performance: Memoization
Memoization is the process of caching results to avoid expensive re-calculations.

1. **`useMemo`:** Caches a **calculated value**. Use for heavy data filtering or transformation.
2. **`useCallback`:** Caches a **function definition**. Use when passing functions to children wrapped in `React.memo` to maintain referential identity.

> [!WARNING]
> **Over-optimization Trap:** Do not wrap every variable in a hook. Memoization has its own memory and comparison cost. Only use it for expensive logic or to prevent broken re-render chains.

---

## 🔹 5. Summary: Hook Decision Matrix

| Requirement | Use This Hook | Triggers Render? |
| :--- | :--- | :--- |
| **Store UI Data** | `useState` | Yes |
| **Complex State Logic** | `useReducer` | Yes |
| **Sync with API/DOM** | `useEffect` | Yes (via state update) |
| **Measure DOM (Pre-paint)** | `useLayoutEffect` | Yes |
| **Store Value (No Render)** | `useRef` | No |
| **Cache Expensive Math** | `useMemo` | No |

---

## 🎯 FAANG Interview "Hooks" Questions
* **Q: Why can't Hooks be used in loops?**
    * *A: React tracks hooks by their index in a linked list. If a loop changes the number of hooks called, the index will mismatch with the stored state, causing a crash.*
* **Q: What is a "Stale Closure" in `useEffect`?**
    * *A: When an effect captures a variable from an old render cycle and doesn't update because that variable wasn't included in the dependency array.*
* **Q: How do you implement `componentDidMount` using hooks?**
    * *A: Use `useEffect(() => { ... }, [])`. The empty dependency array ensures the code runs only once after the initial mount.*

---
**Author:** Huneshwar Yadav | **Specialization:** React Hooks & Performance
