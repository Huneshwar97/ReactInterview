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
