# 🚀 React Fundamentals (FAANG-Level Notes)

---

# 🔹 1. What is React?

**Answer:**

React is a JavaScript library used for building user interfaces, especially single-page applications (SPAs).

It allows developers to build reusable UI components and efficiently update the UI using a virtual representation of the DOM.

---

## 🔹 Why React is popular?

**Answer:**

* Component-based architecture
* Efficient updates using Virtual DOM
* Strong ecosystem
* Declarative programming model

---

# 🔹 2. What is JSX?

**Answer:**

JSX (JavaScript XML) is a syntax extension that allows writing HTML-like code inside JavaScript.

---

## 🔹 Why JSX is used?

**Answer:**

* Improves readability
* Reduces complexity of UI creation
* Allows embedding JavaScript expressions

---

## 🔹 Example:

```jsx
const element = <h1>Hello World</h1>;
```

👉 This is converted internally into:

```js
React.createElement('h1', null, 'Hello World');
```

---

# 🔹 3. What is React.createElement?

**Answer:**

It is a function that creates a React element (object representation of UI).

---

## 🔹 Structure:

```js
React.createElement(type, props, children);
```

---

## 🔹 Example:

```js
React.createElement('div', { className: 'box' }, 'Hello');
```

---

# 🔹 4. What is a React Element?

**Answer:**

A React element is a plain JavaScript object that describes what should appear on the screen.

👉 It is NOT the actual DOM.

---

# 🔹 5. What is a Component?

**Answer:**

A component is a reusable piece of UI.

---

## 🔹 Types of Components:

1. Functional Components
2. Class Components (legacy)

---

## 🔹 Functional Component Example:

```jsx
function Greeting() {
  return <h1>Hello</h1>;
}
```

---

# 🔹 6. Rules for Components

**Answer:**

* Must start with capital letter
* Must return JSX
* Must return a single parent element

---

# 🔹 7. What is Fragment?

**Answer:**

A Fragment allows grouping multiple elements without adding extra DOM nodes.

---

## 🔹 Example:

```jsx
<>
  <h1>Hello</h1>
  <p>World</p>
</>
```

---

# 🔹 8. What are Props?

**Answer:**

Props (properties) are inputs passed to components.

---

## 🔹 Example:

```jsx
function User(props) {
  return <h1>{props.name}</h1>;
}

<User name="Hunesh" />
```

---

## 🔹 Key Characteristics:

* Read-only (immutable)
* Passed from parent to child
* Used for dynamic rendering

---

# 🔹 9. What is Embedded JavaScript in JSX?

**Answer:**

JavaScript expressions can be written inside `{}` in JSX.

---

## 🔹 Example:

```jsx
const name = "Hunesh";
<h1>Hello {name}</h1>
```

---

# 🔹 10. How are attributes different in JSX?

**Answer:**

* `class` → `className`
* `onclick` → `onClick`
* CamelCase naming is used

---

# 🔹 11. How are styles applied in React?

**Answer:**

Styles are passed as JavaScript objects.

---

## 🔹 Example:

```jsx
<h1 style={{ color: 'blue', textAlign: 'center' }}>Hello</h1>
```

---

# 🔹 12. What is StrictMode?

**Answer:**

StrictMode is a tool for highlighting potential problems in React applications during development.

---

## 🔹 Key Behavior:

* Runs components twice in development
* Helps detect side effects

---

# 🔹 13. Why does React require a single root element?

**Answer:**

React needs a single root to efficiently track changes during reconciliation.

---

# 🎯 FAANG Interview Insight

Good candidates:

* Explain JSX → createElement transformation
* Understand elements vs components
* Know why React uses declarative approach

---

# ⚡ Quick Revision

* React = UI library
* JSX = HTML inside JS
* Component = reusable UI
* Props = input
* Fragment = no extra DOM
* createElement = internal engine

---

# ⭐ Author

Huneshwar Yadav

# 🚀 React Mastery: Fundamentals to Architecture (FAANG Edition)

---

## 🔹 1. The Core Philosophy
**Mental Model:** $UI = f(State)$
React is a declarative JavaScript library for building user interfaces. Instead of manually manipulating the DOM (Imperative), you describe what the UI should look like for a specific state (Declarative), and React handles the "how."

---

## 🔹 2. JSX & The Transformation
**JSX (JavaScript XML)** is a syntax extension that looks like HTML but lives in JS.

* **Internal Engine:** JSX is transpiled into `React.createElement(type, props, children)` calls.
* **React Element:** The output of `createElement` is a plain JavaScript object—a "blueprint" of the UI, not the actual DOM.
* **Attributes:** Uses camelCase (`className`, `onClick`) because it is ultimately JavaScript.



---

## 🔹 3. Components & Composition
Components are the "factories" that produce React Elements.

* **Rules:** Must start with a Capital letter and return a single parent element (or a Fragment `<>...</>`).
* **Fragments:** Used because a JS function can only return a single value/object. They group elements without adding extra nodes to the DOM.
* **Props:** Immutable inputs passed from parent to child. They facilitate **One-Way Data Flow**.

---

## 🔹 4. The Virtual DOM & Reconciliation
This is how React stays efficient.

1.  **Virtual DOM (VDM):** A lightweight copy of the Real DOM kept in memory.
2.  **The Render Phase:** When state changes, React creates a new VDM tree.
3.  **Diffing Algorithm ($O(n)$):** React compares the new VDM with the previous version.
    * **Type Change:** If a `<div>` becomes a `<span>`, React destroys the old tree and builds a new one.
    * **Keys:** Essential for list stability. Keys help React identify which items moved, changed, or stayed the same.
4.  **The Commit Phase:** React applies only the necessary changes to the real browser DOM.



---

## 🔹 5. React Fiber (Modern Architecture)
Introduced in React 16, **Fiber** is the reimplementation of the core algorithm.

* **Concurrency:** Fiber allows React to pause, resume, or prioritize rendering work.
* **Responsibility:** It ensures high-priority tasks (like user typing) aren't blocked by heavy background rendering (like a large list).

---

## 🔹 6. Advanced Patterns & Best Practices
* **Lifting State Up:** Moving state to the closest common ancestor to keep sibling components in sync.
* **Immutability:** React uses shallow equality checks (`prev === next`). Always return new objects/arrays when updating state to trigger a re-render.
* **StrictMode:** A development-only tool that double-invokes renders to help find side effects and impure logic.

---

## 🔹 7. Quick Comparison Table

| Feature | Element | Component |
| :--- | :--- | :--- |
| **What is it?** | A plain JS object (Blueprint) | A function or class (Factory) |
| **Mutability** | Immutable | Can manage internal state |
| **Syntax** | `const el = <div />` | `function App() { ... }` |

---

## 🎯 Interview "Pro" Tips
* **Explain JSX:** Mention that it's syntactic sugar for object creation.
* **Explain VDM:** Clarify that it's not "faster" than the DOM, but it makes updates **predictable** and batches them to avoid layout thrashing.
* **Explain Keys:** Emphasize that using `index` as a key is dangerous if the list order can change.

---
**Author:** Huneshwar Yadav | **Level:** FAANG Prep
