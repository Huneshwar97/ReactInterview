# 🚀 Context API & State Management (FAANG-Level Deep Dive)

💡 Covers **Global State, Context API, Prop Drilling, Performance, and Trade-offs**

---

# 🔹 1. What is State Management?

**Answer:**

State management is the process of managing and sharing data across components in an application.

---

## 🔍 Deep Understanding

In small apps:
👉 Local state is enough

In large apps:
👉 Multiple components need same data

---

## 🔹 Example

```id="0l1d8r"
User data needed in:
- Navbar
- Profile
- Dashboard
```

---

## 🔹 Problem

👉 Passing props everywhere becomes messy

---

# 🔹 2. What is Prop Drilling?

**Answer:**

Prop drilling is passing data through multiple intermediate components to reach deeply nested components.

---

## 🔍 Example

```jsx id="2s9dks"
<App>
  <Parent>
    <Child>
      <GrandChild user={user} />
    </Child>
  </Parent>
</App>
```

---

## 🔹 Problems

* Hard to maintain
* Unnecessary props passing
* Tight coupling

---

## 🔹 Advanced Question

**Q: Why is prop drilling bad for scalability?**

Because:

* Increases dependency between components
* Makes refactoring difficult

---

# 🔹 3. What is Context API?

**Answer:**

Context API allows sharing data globally without passing props manually at every level.

---

## 🔍 Core Idea

👉 Provide data at top
👉 Consume anywhere

---

# 🔹 4. How Context Works?

## Step 1: Create Context

```js id="b2e7g9"
const UserContext = createContext();
```

---

## Step 2: Provide Data

```jsx id="z4k8t1"
<UserContext.Provider value={user}>
  <App />
</UserContext.Provider>
```

---

## Step 3: Consume Data

```js id="y7h2wq"
const user = useContext(UserContext);
```

---

## 🔹 Flow

```id="0p9d8s"
Provider → Context → Consumer
```

---

# 🔹 5. Why Context API is useful?

**Answer:**

* Avoids prop drilling
* Simplifies data sharing
* Cleaner architecture

---

## 🔹 Real-world Example

```id="o3m9dz"
Auth Context → used in:
- Navbar
- Profile
- Protected Routes
```

---

# 🔹 6. What are the limitations of Context API?

**Answer:**

* Causes unnecessary re-renders
* Not ideal for very large apps
* Hard to debug at scale

---

## 🔹 Advanced Insight

👉 Context updates re-render ALL consumers

---

## 🔹 Example Problem

```jsx id="t6p2s4"
<UserContext.Provider value={{ user }}>
```

👉 New object every render → all children re-render

---

# 🔹 7. How to optimize Context performance?

**Answer:**

### 1. Memoize value

```js id="q8k2p1"
const value = useMemo(() => ({ user }), [user]);
```

---

### 2. Split contexts

Instead of:
👉 One big context

Use:
👉 Multiple smaller contexts

---

### 3. Use selectors (advanced libs)

---

# 🔹 8. Context vs Redux

| Feature     | Context           | Redux      |
| ----------- | ----------------- | ---------- |
| Complexity  | Low               | High       |
| Setup       | Easy              | Complex    |
| Performance | Limited           | Better     |
| Use case    | Small-medium apps | Large apps |

---

## 🔹 Advanced Insight

👉 Context is NOT full state management solution

---

# 🔹 9. When should you use Context?

**Answer:**

Use Context for:

* Auth state
* Theme
* Language

---

## 🔹 Avoid Context for:

* Frequently changing data
* Large-scale apps

---

# 🔹 10. What is Global State?

**Answer:**

State shared across multiple components.

---

## 🔹 Example

```id="p9c3n1"
Global:
- User
- Cart
- Theme
```

---

# 🔹 11. What is Local vs Global State?

| Type   | Scope            |
| ------ | ---------------- |
| Local  | Single component |
| Global | Entire app       |

---

## 🔹 Insight

👉 Keep state as local as possible

---

# 🔹 12. What is Lifting State Up?

**Answer:**

Moving state to common ancestor to share between components.

---

## 🔹 Why important?

* Synchronizes UI
* Avoids duplication

---

# 🔹 13. What is State Co-location?

**Answer:**

Keeping state close to where it is used.

---

## 🔹 Why important?

* Reduces unnecessary re-renders
* Improves performance

---

# 🔹 14. Common Mistakes

* Overusing Context
* Large global state
* Not memoizing values
* Using Context for everything

---

# 🔹 15. Advanced Interview Questions

**Q1: Why Context can hurt performance?**
👉 Because it re-renders all consumers

---

**Q2: How to optimize Context?**
👉 Memoization + splitting

---

**Q3: When would you choose Redux over Context?**
👉 Large apps with complex state

---

**Q4: What is state co-location and why important?**
👉 Keeps state near usage → reduces re-renders

---

# 🔹 16. Real-world Architecture Insight

👉 Example:

```id="3f8h2k"
App
 ├── AuthContext
 ├── ThemeContext
 ├── Components
```

---

## 🔹 Key Idea

👉 Not everything should be global

---

# 🎯 FAANG-Level Insight

Top candidates:

* Know when NOT to use Context
* Understand re-render cost
* Optimize state placement
* Think in scalability

---

# ⚡ Quick Revision

* Context = global data
* Prop drilling = problem
* Memoization = optimization
* Global state = shared
* Local state = preferred

---

# ⭐ Author

Huneshwar Yadav
