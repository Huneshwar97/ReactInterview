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
