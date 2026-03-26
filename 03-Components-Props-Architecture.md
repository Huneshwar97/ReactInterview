# 🚀 Components, Props & Architecture (FAANG-Level Deep Dive)

---

# 🔹 1. What are Components in React?

**Answer:**

Components are reusable, independent pieces of UI that encapsulate both structure (JSX) and logic.

---

## 🔍 Deep Understanding

Think of components as:

👉 **Functions that return UI**

```js
UI = f(data)
```

Each component:

* Takes input (props/state)
* Returns UI (JSX)

---

## 🔹 Why Components are powerful?

* Reusability
* Maintainability
* Separation of concerns
* Scalable architecture

---

## 🔹 Advanced Question

**Q: Why is component-based architecture important for large applications?**

Because it:

* Breaks UI into manageable parts
* Enables team collaboration
* Reduces complexity

---

# 🔹 2. Types of Components

## 1. Functional Components (Modern)

```jsx
function Card() {
  return <div>Card</div>;
}
```

---

## 2. Class Components (Legacy)

```js
class Card extends React.Component {
  render() {
    return <div>Card</div>;
  }
}
```

---

## 🔍 Deep Insight

Functional components are preferred because:

* Simpler syntax
* Hooks support
* Better readability

---

# 🔹 3. What are Props?

**Answer:**

Props (properties) are inputs passed from parent to child components.

---

## 🔍 Deep Understanding

Props behave like function arguments:

```jsx
function User(props) {
  return <h1>{props.name}</h1>;
}
```

---

## 🔹 Key Characteristics

* Immutable
* Passed top-down
* Used for dynamic rendering

---

## 🔹 Advanced Question

**Q: Why are props immutable?**

Because:

* Ensures predictable data flow
* Prevents accidental side effects
* Maintains React’s one-way data flow

---

# 🔹 4. What is Component Composition?

**Answer:**

Component composition is the practice of combining multiple components to build complex UI.

---

## 🔍 Example

```jsx
function App() {
  return (
    <>
      <Navbar />
      <ProductList />
      <Footer />
    </>
  );
}
```

---

## 🔹 Why important?

* Encourages reuse
* Reduces duplication
* Improves readability

---

# 🔹 5. What is Children Prop?

**Answer:**

`props.children` allows passing content between component tags.

---

## 🔹 Example

```jsx
function Card({ children }) {
  return <div className="card">{children}</div>;
}

<Card>
  <h1>Hello</h1>
</Card>
```

---

## 🔹 Advanced Insight

Children make components:

* More flexible
* More reusable

---

# 🔹 6. What is Prop Drilling?

**Answer:**

Prop drilling is passing data through multiple intermediate components to reach deeply nested components.

---

## 🔹 Example

```jsx
<App>
  <Parent>
    <Child>
      <GrandChild data={data} />
    </Child>
  </Parent>
</App>
```

---

## 🔹 Problem

* Hard to maintain
* Unnecessary props passing
* Tight coupling

---

## 🔹 Solution

* Context API
* State management libraries

---

# 🔹 7. What is Lifting State Up?

**Answer:**

Moving state to the closest common ancestor so multiple components can share it.

---

## 🔍 Example

```jsx
function Parent() {
  const [value, setValue] = useState("");

  return (
    <>
      <Input setValue={setValue} />
      <Display value={value} />
    </>
  );
}
```

---

## 🔹 Why important?

* Synchronizes data across components
* Avoids duplication

---

# 🔹 8. Smart vs Dumb Components

## Smart (Container)

* Handles logic
* Manages state

## Dumb (Presentational)

* Only renders UI
* Uses props

---

## 🔹 Example

```jsx
// Smart
function App() {
  const [data, setData] = useState([]);
  return <List data={data} />;
}

// Dumb
function List({ data }) {
  return data.map(item => <li>{item}</li>);
}
```

---

## 🔹 Advanced Insight

👉 Separation improves:

* Testing
* Reusability
* Maintainability

---

# 🔹 9. What is Component Reusability?

**Answer:**

Designing components in a way they can be reused across multiple parts of the app.

---

## 🔹 Example

```jsx
<Button label="Login" />
<Button label="Signup" />
```

---

## 🔹 Advanced Insight

Reusable components:

* Reduce code duplication
* Improve consistency

---

# 🔹 10. What is Controlled vs Uncontrolled Component?

## Controlled:

State managed by React

```jsx
<input value={value} onChange={...} />
```

---

## Uncontrolled:

State managed by DOM

```jsx
<input ref={inputRef} />
```

---

## 🔹 Advanced Question

**Q: Why prefer controlled components?**

Because:

* Better control
* Easier validation
* Predictable behavior

---

# 🔹 11. What is Key in Component Lists?

**Answer:**

Keys help React identify elements uniquely.

---

## 🔹 Example

```jsx
items.map(item => <li key={item.id}>{item.name}</li>)
```

---

## 🔹 Advanced Insight

Keys:

* Improve diffing performance
* Prevent UI bugs

---

# 🔹 12. What is Component Lifecycle (Functional View)?

**Answer:**

Lifecycle in functional components is handled using hooks:

* Mount → useEffect
* Update → useEffect
* Unmount → cleanup

---

# 🔹 13. How to design scalable component architecture?

**Answer:**

👉 Follow these principles:

* Small, focused components
* Separation of concerns
* Reusable components
* Proper state placement

---

## 🔹 Real-world Example

E-commerce app:

* App

  * Navbar
  * ProductList
  * Cart
  * Footer

---

# 🔹 14. Common Mistakes

* Overusing props
* Deep prop drilling
* Large components
* Incorrect key usage

---

# 🔹 15. Advanced Interview Questions

**Q1: How would you design a scalable component system?**
👉 Use composition + reusable components + proper state placement

**Q2: When would you avoid Context API?**
👉 When frequent updates cause unnecessary re-renders

**Q3: How do you prevent unnecessary re-renders?**
👉 React.memo, useMemo, useCallback

---

# 🎯 FAANG-Level Insight

Top candidates:

* Think in component hierarchy
* Understand data flow deeply
* Design scalable UI architecture
* Avoid prop drilling

---

# ⚡ Quick Revision

* Component = UI function
* Props = input
* Children = flexible composition
* State lifting = shared data
* Smart vs Dumb = separation

---

# ⭐ Author

Huneshwar Yadav

# 🚀 React Architecture: Components, Props & Design Patterns

---

## 🔹 1. The Component Philosophy
**Mental Model:** $UI = f(data)$
Components are independent, reusable pieces of UI. In a scalable architecture, they should follow the **Single Responsibility Principle**: one component should do one thing well.

* **Functional Components:** The modern standard. They are simpler, easier to test, and support Hooks.
* **Component Identity:** React identifies components by their position in the tree and their `key`. 

---

## 🔹 2. Props & Data Flow
Props are the "API" of your component. They allow data to flow **down** the tree (One-Way Data Flow).

* **Immutability:** Props are read-only. This makes the data flow predictable and easier to debug.
* **Children Prop:** Used for **Composition**. It allows you to pass JSX elements as data, making components flexible "shells."
* **The "Slot" Pattern:** Passing components as named props (e.g., `header={<Header />}`) to avoid deep nesting and prop drilling.



---

## 🔹 3. State Placement & Management
Where you put your state determines how much of your app re-renders.

1.  **Lifting State Up:** Moving state to the *closest common ancestor* so siblings can share data.
2.  **Colocation:** The opposite of lifting—keeping state as local as possible. If only one sub-tree needs data, don't put it in a global Context. This limits the "render blast radius."
3.  **Prop Drilling:** Passing props through components that don't need them. 
    * *Solution:* **Context API** for global data (Theme, User) or **Composition** for UI structure.

---

## 🔹 4. Smart vs. Dumb Components
A classic architectural split for maintainability:

* **Smart (Container):** Concerned with *how things work*. They fetch data, manage state, and deal with side effects.
* **Dumb (Presentational):** Concerned with *how things look*. They receive data via props and have no dependencies on the rest of the app.

---

## 🔹 5. Controlled vs. Uncontrolled Components
| Feature | Controlled | Uncontrolled |
| :--- | :--- | :--- |
| **Source of Truth** | React State | The DOM (Refs) |
| **Best For** | Validation, Dynamic inputs | File uploads, Simple forms |
| **Performance** | Re-renders on every change | No re-renders during typing |

---

## 🔹 6. Advanced Pattern: Inversion of Control (IoC)
Instead of a component having 20 props to handle every edge case, use **Composition**.
* **Bad:** `<List showIcon={true} iconSize={20} textColor="red" />`
* **Good:** `<List><ListItem icon={<UserIcon />} color="red">Content</ListItem></List>`
* **Why?** It gives the consumer of the component control over the UI without bloating the component logic.



---

## 🎯 FAANG Architectural Interview Questions
* **Q: How do you prevent a Context update from re-rendering the whole app?**
    * *A: Split contexts into smaller ones, wrap children in `React.memo`, or use the "Children as Props" pattern so the child tree isn't re-created.*
* **Q: When is "Prop Drilling" actually okay?**
    * *A: When the data flow is explicit and the components are closely related. Overusing Context makes components harder to reuse in isolation.*
* **Q: Why avoid defining components inside components?**
    * *A: Because React will see it as a new function reference every render, unmounting and remounting the entire sub-tree, which destroys performance and state.*

---
**Author:** Huneshwar Yadav | **Focus:** Scalable UI Architecture
