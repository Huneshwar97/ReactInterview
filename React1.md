# 🚀 React Interview Preparation (Amazon / Flipkart Style)

This repository contains **React interview questions and answers** structured for top product-based companies like Amazon, Flipkart, etc.

---

## 🔹 1. What is React?

React is a JavaScript library developed by Meta for building user interfaces, especially for Single Page Applications (SPAs).

* Uses component-based architecture
* Uses Virtual DOM for performance
* Focuses on UI layer

---

## 🔹 2. Why React?

React solves:

* Complex UI updates
* DOM manipulation inefficiency
* Poor scalability

**Key features:**

* Reusable components
* Virtual DOM
* One-way data flow

---

## 🔹 3. What is Virtual DOM?

Virtual DOM is a lightweight copy of the real DOM.

### How it works:

1. React creates Virtual DOM
2. Compares old vs new (diffing)
3. Updates only changed parts

👉 Improves performance

---

## 🔹 4. What is Reconciliation?

Reconciliation is the process of comparing old and new Virtual DOM and updating only the differences.

---

## 🔹 5. State vs Props

| Feature    | State        | Props        |
| ---------- | ------------ | ------------ |
| Mutability | Mutable      | Immutable    |
| Owned by   | Component    | Parent       |
| Purpose    | Dynamic data | Data passing |

---

## 🔹 6. Unidirectional Data Flow

Data flows from:

Parent → Child

### Benefits:

* Predictable behavior
* Easy debugging

---

## 🔹 7. Declarative vs Imperative

### Imperative

You define **how** to update UI

### Declarative (React)

You define **what** UI should look like

---

## 🔹 8. Why DOM manipulation is expensive?

* Causes reflow & repaint
* Impacts performance

👉 React avoids this using Virtual DOM

---

## 🔹 9. Components in React

Reusable building blocks of UI.

### Types:

* Functional components
* Class components

---

## 🔹 10. What is JSX?

JSX allows writing HTML inside JavaScript.

Example:

```jsx
const element = <h1>Hello</h1>;
```

---

## 🔹 11. Is React a library or framework?

React is a **library** because it only handles UI.

---

## 🔹 12. React vs Angular

| Feature        | React   | Angular     |
| -------------- | ------- | ----------- |
| Type           | Library | Framework   |
| Flexibility    | High    | Opinionated |
| Learning Curve | Easy    | Hard        |

---

## 🔹 13. How React improves performance?

* Virtual DOM
* Diffing algorithm
* Efficient updates

---

## 🔹 14. What happens when state changes?

1. Component re-renders
2. Virtual DOM updates
3. Diffing happens
4. Real DOM updates

---

## 🔹 15. Key Interview Tip

Use this structure while answering:

Definition → Working → Benefit → Example

---

## 📌 Quick Revision

* React = UI Library
* JSX = HTML in JS
* State = Dynamic data
* Props = Data passing
* Virtual DOM = Performance booster

---

## ⭐ Author

Huneshwar Yadav
