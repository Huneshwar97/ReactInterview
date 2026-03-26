# 🚀 React Elements, JSX & Rendering (FAANG-Level Q&A)

💡 Covers **React elements, JSX rules, attributes, embedded JS, styling, fragments, and Strict Mode** — commonly asked in interviews.

---

## 🔹 1. What is a React Element?

**Answer:**

A React Element is the smallest building block of a React application. It is a plain JavaScript object that describes what should appear on the screen.

Although it looks like HTML, it is not actual HTML — it gets converted into real DOM elements by React.

👉 Example:

```jsx id="e1a2b3"
const element = <h1>Hello</h1>;
```

👉 This creates a React element, not a real DOM node.

---

## 🔹 2. How are React elements different from HTML?

**Answer:**

React elements look like HTML but behave differently.

* They are written using JSX
* Internally converted into JavaScript objects
* Finally rendered into real DOM

👉 Key idea:
React elements are **descriptions of UI**, not actual UI.

---

## 🔹 3. Why do we use `className` instead of `class` in JSX?

**Answer:**

In JavaScript, `class` is a reserved keyword. Since JSX is JavaScript-like syntax, React uses `className` instead.

👉 Example:

```jsx id="x9a1c2"
<div className="container"></div>
```

👉 This avoids conflicts with JavaScript keywords.

---

## 🔹 4. Why do we use camelCase for attributes in JSX?

**Answer:**

JSX follows JavaScript conventions, so attributes are written in camelCase instead of lowercase.

### Examples:

* `onclick` → `onClick`
* `tabindex` → `tabIndex`

👉 This ensures consistency with JavaScript naming conventions.

---

## 🔹 5. What are JSX expressions?

**Answer:**

JSX allows embedding JavaScript expressions inside curly braces `{}`.

### Example:

```jsx id="p9k2m3"
const name = "Huneshwar";
<h1>Hello {name}</h1>
```

👉 This allows dynamic content rendering.

---

## 🔹 6. Can we use JavaScript inside JSX?

**Answer:**

Yes, we can use JavaScript expressions inside JSX using `{}`.

### Example:

```jsx id="q1w2e3"
const color = "blue";
<div className={color}></div>
```

👉 Only expressions are allowed (not statements like `if`, `for` directly).

---

## 🔹 7. How does inline styling work in React?

**Answer:**

In React, inline styles are written as JavaScript objects instead of strings.

### Example:

```jsx id="z8x7c6"
<h1 style={{ color: "blue", textAlign: "center" }}>Hello</h1>
```

### Key points:

* Use double curly braces `{{}}`
* CSS properties are written in camelCase

👉 This makes styles dynamic and JavaScript-friendly.

---

## 🔹 8. What is a React Fragment?

**Answer:**

A React Fragment allows grouping multiple elements without adding extra nodes to the DOM.

### Example:

```jsx id="f3g4h5"
<>
  <h1>Hello</h1>
  <p>World</p>
</>
```

👉 It avoids unnecessary `<div>` wrappers.

---

## 🔹 9. Why must a component return a single root element?

**Answer:**

React components must return a single root element because React needs a single entry point to render and manage updates efficiently.

### Example (Incorrect):

```jsx id="bad1"
return (
  <h1>Hello</h1>
  <p>World</p>
);
```

### Correct:

```jsx id="good1"
return (
  <>
    <h1>Hello</h1>
    <p>World</p>
  </>
);
```

👉 Fragments solve this problem without adding extra DOM nodes.

---

## 🔹 10. What is React Strict Mode?

**Answer:**

React Strict Mode is a development tool that helps identify potential problems in an application.

### Behavior:

* Runs certain functions twice in development
* Helps detect side effects
* Improves code quality

👉 Important:

* Only affects development mode
* Does not impact production

---

## 🔹 11. Why does Strict Mode run components twice?

**Answer:**

Strict Mode intentionally runs components twice to detect:

* Side effects
* Unsafe operations
* Improper state handling

👉 This helps developers catch bugs early.

---

## 🔹 12. How does JSX improve development?

**Answer:**

JSX reduces the gap between UI and logic by allowing developers to write HTML-like syntax inside JavaScript.

### Benefits:

* Better readability
* Cleaner code
* Easier maintenance

👉 It eliminates the need to switch between HTML and JS files.

---

## 🔹 13. What are common mistakes in JSX?

**Answer:**

* Using `class` instead of `className`
* Not using camelCase (e.g., `onclick`)
* Returning multiple root elements
* Using statements instead of expressions

👉 These are frequently asked in interviews.

---

## 🔹 14. What is the flow from JSX to UI?

**Answer:**

JSX → JavaScript object (React Element) → Virtual DOM → Real DOM

👉 This is how React converts code into visible UI.

---

## 🎯 FAANG Interview Insight

👉 Strong candidates:

* Understand JSX as abstraction, not HTML
* Know why React enforces rules (single root, camelCase)
* Can explain internal flow clearly

---

## ⚡ Quick Revision

* React Element = UI description
* JSX = HTML-like syntax in JS
* class → className
* Inline style = JS object
* Fragment = no extra DOM node
* Strict Mode = dev debugging tool

---

## ⭐ Author

Huneshwar Yadav
