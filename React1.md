# 🚀 React Interview Notes (FAANG-Level Q&A with Explanation)

💡 These notes include **clear explanations, reasoning, and trade-offs** — ideal for product-based companies like Amazon / Flipkart.

---

## 🔹 1. What problems existed before React?

**Answer:**

Before React, frontend development mainly used HTML, CSS, and JavaScript, along with libraries like jQuery and Backbone.js.

The main issue was that developers had to manually manipulate the DOM (Document Object Model). As applications grew larger, this became difficult to manage and error-prone. Updating one part of the UI could unintentionally affect another part.

There were also browser inconsistencies, meaning the same code behaved differently across browsers.

👉 In short, there was no efficient and scalable way to manage UI updates in complex applications.

---

## 🔹 2. What is a Single Page Application (SPA)?

**Answer:**

A Single Page Application (SPA) is a web application that loads a single HTML page initially and then dynamically updates the content using JavaScript, without refreshing the entire page.

Instead of requesting a new page from the server every time, only the required data is fetched and the UI is updated accordingly.

### Why it is useful:

* It provides a smooth and fast user experience
* Reduces unnecessary server requests
* Makes applications feel more like mobile apps

### Trade-offs:

* The initial load can be heavier
* SEO was challenging earlier (though now improved)
* Managing state becomes more complex

---

## 🔹 3. How is React different from Angular?

**Answer:**

React is a library focused only on building UI, whereas Angular is a full-fledged framework that provides everything needed to build an application.

React gives developers flexibility to choose tools (like routing, state management), while Angular enforces a strict structure using the MVC pattern.

👉 This flexibility makes React easier to adopt and scale in large teams, while Angular is more opinionated and structured.

---

## 🔹 4. Why is DOM manipulation expensive?

**Answer:**

DOM operations are expensive because every change in the DOM can trigger:

* **Reflow** (recalculating layout)
* **Repaint** (redrawing the UI)

These operations are handled by the browser and are computationally heavy, especially when dealing with large UI trees.

For example, updating multiple elements directly can cause repeated layout calculations, slowing down the application.

👉 This is why optimizing DOM updates is critical for performance.

---

## 🔹 5. What is Virtual DOM?

**Answer:**

The Virtual DOM is a lightweight JavaScript representation of the real DOM.

When the state of a component changes, React does not directly update the real DOM. Instead, it creates a new Virtual DOM and compares it with the previous one using a process called diffing.

Only the parts that have changed are updated in the real DOM.

### Why this is useful:

* Minimizes direct DOM operations
* Improves performance
* Makes UI updates more predictable

### Trade-offs:

* Extra memory is used to maintain Virtual DOM
* Diffing itself has some computational cost

👉 So, Virtual DOM is beneficial especially when there are frequent UI updates.

---

## 🔹 6. What is Reconciliation?

**Answer:**

Reconciliation is the process React uses to update the DOM efficiently.

When a component’s state or props change, React creates a new Virtual DOM tree. It then compares this new tree with the previous one to identify differences.

Based on these differences, React updates only the necessary parts of the real DOM.

👉 This process ensures that updates are minimal and efficient, rather than re-rendering the entire UI.

---

## 🔹 7. What does “Don’t touch the DOM” mean in React?

**Answer:**

This means developers should avoid directly manipulating the DOM using methods like `document.getElementById`.

React internally manages the DOM using its Virtual DOM and reconciliation process. If developers manually change the DOM, it can lead to inconsistencies because React is unaware of those changes.

👉 So, all UI updates should be handled through state and props, not direct DOM manipulation.

---

## 🔹 8. What is Imperative vs Declarative programming?

**Answer:**

In the imperative approach, we write step-by-step instructions describing how to update the UI.

In the declarative approach (used by React), we describe what the UI should look like based on the current state, and React takes care of updating it.

### Why declarative is better:

* Code is easier to read and maintain
* Reduces complexity
* Minimizes bugs caused by manual updates

👉 This shift is one of the biggest reasons React became popular.

---

## 🔹 9. What are Components in React?

**Answer:**

Components are reusable pieces of UI that help break the interface into smaller, manageable parts.

Each component can manage its own logic and rendering, making the application modular.

### Benefits:

* Reusability across the application
* Better organization of code
* Easier testing and debugging

For example, in an e-commerce app, components could be Navbar, ProductCard, or CartItem.

---

## 🔹 10. What is Unidirectional Data Flow?

**Answer:**

In React, data flows in one direction — from parent to child components.

The parent component passes data to child components using props, and the child cannot directly modify that data.

### Why this is important:

* Makes data flow predictable
* Easier to debug issues
* Prevents unintended side effects

### Trade-off:

* Can lead to prop drilling in deeply nested components

---

## 🔹 11. What is JSX?

**Answer:**

JSX is a syntax extension that allows writing HTML-like code inside JavaScript.

It makes it easier to define UI structure in a readable way.

Internally, JSX is converted into JavaScript function calls (`React.createElement`), so browsers can understand it.

👉 JSX is not required, but it significantly improves developer experience.

---

## 🔹 12. What is the core responsibility of React?

**Answer:**

React is responsible for:

* Rendering UI
* Updating UI efficiently when data changes

It focuses only on the view layer of the application.

👉 Other concerns like routing or global state management are handled by additional libraries.

---

## 🔹 13. How does React update UI internally?

**Answer:**

When state or props change:

1. React re-renders the component
2. A new Virtual DOM is created
3. React compares it with the previous Virtual DOM
4. Only the changed parts are updated in the real DOM

👉 This ensures efficient and optimized rendering.

---

## 🔹 14. What is the relationship between State, Props, and UI?

**Answer:**

State and props together define how the UI should look.

* State represents internal data
* Props represent external data passed from parent

These are used to generate JSX, which forms the Virtual DOM, and finally updates the real DOM.

👉 Flow:
State + Props → UI (JSX) → Virtual DOM → Real DOM

---

## 🎯 FAANG Interview Tip

Always structure answers like:

👉 Definition → Working → Why it matters → Trade-offs → Example

---

## ⚡ Quick Revision

* React = UI Library
* Virtual DOM = Optimized updates
* JSX = UI syntax
* State = Dynamic data
* Props = Data passing
* One-way data flow = Predictable

---

## ⭐ Author

Huneshwar Yadav
