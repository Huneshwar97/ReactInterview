# 🚀 Advanced Patterns: Observer, Streams & Functional Programming (FAANG-Level Deep Dive)

---

# 🔹 1. What is the Observer Pattern?

**Answer:**

The Observer Pattern is a design pattern where a subject (data source) maintains a list of observers (subscribers) and notifies them whenever the state changes.

---

## 🔍 Deep Understanding

👉 Think of it like:

* You subscribe to YouTube channel
* Creator uploads video
* You get notified

👉 Same concept in programming:

* Subject = data source
* Observer = listener

---

## 🔹 Structure

```id="p5s1nq"
Subject → Observers
```

---

## 🔹 Real-world Example

* Event listeners
* Firebase real-time updates
* React state updates

---

## 🔹 Advanced Question

**Q: Why is Observer Pattern useful?**

Because:

* Decouples components
* Enables real-time updates
* Avoids polling

---

# 🔹 2. What is a Stream?

**Answer:**

A stream is a sequence of asynchronous data/events over time.

---

## 🔍 Examples

* User typing
* API responses
* Button clicks

---

## 🔹 Key Idea

👉 Instead of pulling data:
👉 We react to incoming data

---

## 🔹 Advanced Question

**Q: Difference between synchronous vs stream-based systems?**

| Sync     | Stream     |
| -------- | ---------- |
| Blocking | Async      |
| One-time | Continuous |
| Pull     | Push       |

---

# 🔹 3. What are Observer Methods?

**Answer:**

Observers typically have:

* `next(value)` → new data
* `error(err)` → error
* `complete()` → stream end

---

## 🔹 Example

```js id="b2g5wp"
observer.next("data");
observer.error("error");
observer.complete();
```

---

## 🔹 Advanced Insight

👉 Used in:

* RxJS
* Event systems
* Reactive programming

---

# 🔹 4. How React relates to Observer Pattern?

**Answer:**

React follows a similar pattern internally:

* State = subject
* Component = observer

---

## 🔍 Flow

```id="g8b7kn"
State changes → Component notified → Re-render
```

---

## 🔹 Insight

👉 React is NOT pure observer pattern
👉 But inspired by reactive principles

---

# 🔹 5. Observer Pattern in Firebase

**Answer:**

Firebase uses observer pattern for real-time updates.

---

## 🔹 Example

```js id="7y1n4x"
onAuthStateChanged(auth, (user) => {
  console.log(user);
});
```

---

## 🔹 Explanation

* Firebase = subject
* Callback = observer

---

## 🔹 Why powerful?

* Real-time sync
* No manual polling
* Event-driven

---

# 🔹 6. What is Functional Programming?

**Answer:**

A programming paradigm based on:

* Pure functions
* Immutability
* No side effects

---

## 🔹 Key Principles

1. Pure functions
2. Immutability
3. First-class functions

---

# 🔹 7. What is a Pure Function?

**Answer:**

A function that:

* Same input → same output
* No external dependency
* No side effects

---

## 🔹 Example

```js id="0dzmrd"
const add = (a, b) => a + b;
```

---

## 🔹 Why important in React?

Because:
👉 Predictable UI
👉 Easier debugging
👉 Efficient rendering

---

# 🔹 8. What is an Impure Function?

**Answer:**

A function that depends on external state or causes side effects.

---

## 🔹 Example

```js id="slf0cd"
let count = 10;
const add = (x) => x + count;
```

---

## 🔹 Problem

* Unpredictable
* Hard to debug

---

# 🔹 9. What is Immutability?

**Answer:**

Data should not be modified directly — instead create new copies.

---

## 🔹 Example

❌ Wrong:

```js id="7u9mnk"
arr.push(1);
```

✔️ Correct:

```js id="m3cqka"
setArr([...arr, 1]);
```

---

## 🔹 Why important?

React uses:
👉 Reference comparison

---

# 🔹 10. What are Side Effects?

**Answer:**

Operations that affect outside world.

---

## 🔹 Examples

* API calls
* DOM updates
* Timers

---

## 🔹 In React

Handled using:
👉 `useEffect`

---

# 🔹 11. What is Declarative Programming?

**Answer:**

Describing **what UI should look like**, not how to update it.

---

## 🔹 Example

```jsx id="sp9q2o"
{isLoggedIn ? <Dashboard /> : <Login />}
```

---

## 🔹 Why important?

* Cleaner code
* Less bugs
* Better abstraction

---

# 🔹 12. What is Event-Driven Architecture?

**Answer:**

System reacts to events instead of executing sequential steps.

---

## 🔹 Examples

* Click events
* API responses
* Firebase updates

---

## 🔹 Insight

👉 Modern frontend apps are event-driven systems

---

# 🔹 13. Real-world System Thinking

👉 Example:

User login flow:

```id="2lpx3c"
User clicks login → API call → response → state update → UI update
```

---

## 🔹 Key Idea

Everything is:
👉 Event → State → UI

---

# 🔹 14. Common Mistakes

* Mutating state
* Writing impure components
* Ignoring side effects
* Mixing logic with UI

---

# 🔹 15. Advanced Interview Questions

**Q1: How does React follow reactive programming?**
👉 State-driven UI updates

---

**Q2: Why is immutability important for performance?**
👉 Enables shallow comparison

---

**Q3: Difference between Observer and Pub-Sub?**
👉 Observer = direct dependency
👉 Pub-Sub = decoupled via broker

---

# 🎯 FAANG-Level Insight

Top candidates:

* Think in events + state
* Understand reactive systems
* Connect patterns to real-world systems
* Explain trade-offs

---

# ⚡ Quick Revision

* Observer = subscribe system
* Stream = async data flow
* Pure function = predictable
* Immutability = required
* Side effects = controlled

---

# ⭐ Author

Huneshwar Yadav
