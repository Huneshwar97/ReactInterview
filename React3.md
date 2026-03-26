# 🚀 React Setup & Developer Thinking (FAANG-Level Q&A)

💡 Covers **how good developers think + tooling + ecosystem (npm, Vite, CRA)**

---

## 🔹 1. How does a good frontend developer approach building a React app?

**Answer:**

A good developer does not start coding immediately. They first think about structure and data flow.

### Step-by-step approach:

1. **Decide components**

   * Break UI into reusable pieces (Navbar, ProductCard, etc.)
   * Helps in modular and scalable design

2. **Decide state and where it should live**

   * Identify what data changes over time
   * Place state at the correct level (lifting state up if needed)

3. **Understand what changes when state changes**

   * Determine which components re-render
   * Avoid unnecessary re-renders

👉 **FAANG Insight:**
This approach ensures scalability, maintainability, and performance optimization from the start.

---

## 🔹 2. What is npm and yarn? How are they different?

**Answer:**

Both npm and yarn are **package managers** used to install and manage dependencies in JavaScript projects.

### npm:

* Default package manager for Node.js
* Widely used and supported

### yarn:

* Developed to improve performance and consistency
* Faster installs (in earlier versions)
* Better dependency locking

👉 Today, both are quite similar, and the choice depends on team preference.

---

## 🔹 3. Common npm/yarn commands (with explanation)

**Answer:**

### Install dependencies:

```bash id="y4g0g9"
npm install
```

👉 Installs all dependencies listed in `package.json`

---

### Install a package:

```bash id="w0o3v4"
npm install package-name
```

👉 Adds dependency to project

---

### Install dev dependency:

```bash id="3z6k9y"
npm install package-name --save-dev
```

👉 Used for tools like testing, bundlers, linters

---

### Remove a package:

```bash id="k4q5yo"
npm uninstall package-name
```

---

### Update package:

```bash id="z7c0jf"
npm update
```

---

### Install globally:

```bash id="7qq8y3"
npm install -g package-name
```

👉 Global packages are available system-wide (e.g., CLI tools)

---

## 🔹 4. What does the browser understand?

**Answer:**

Browsers only understand:

* HTML → Structure
* CSS → Styling
* JavaScript → Logic

👉 They do NOT understand:

* JSX
* TypeScript
* Modern JS directly (in all cases)

---

## 🔹 5. Why do we need tools like React, Vite, Webpack?

**Answer:**

Because modern development uses advanced syntax and features that browsers cannot directly understand.

### Tools help to:

* Convert JSX → JavaScript
* Bundle multiple files into one
* Optimize performance

👉 These tools act as a **bridge between developer code and browser-compatible code**.

---

## 🔹 6. What is a bundler, compiler, and transpiler?

**Answer:**

### Bundler:

* Combines multiple files into one (e.g., Webpack, Vite)

### Compiler:

* Converts code from one form to another

### Transpiler:

* Converts modern JavaScript → older JavaScript
* Example: Babel

👉 These tools make modern React apps possible.

---

## 🔹 7. Why is React called a “superpower” over vanilla JavaScript?

**Answer:**

React simplifies complex UI development.

In vanilla JS:

* You manually update DOM
* Hard to manage large apps

In React:

* UI updates automatically based on state
* Code is more structured and reusable

👉 React abstracts complexity and improves developer productivity.

---

## 🔹 8. What is Create React App (CRA)?

**Answer:**

Create React App was a tool used to quickly set up a React project with pre-configured settings.

### Features:

* No setup required
* Includes Webpack, Babel, etc.

### Current Status:

* Not actively maintained by React team (modern shift)

👉 Used earlier, but now replaced by better tools.

---

## 🔹 9. What is Vite and why is it popular?

**Answer:**

Vite is a modern frontend build tool designed for fast development.

### Why Vite is better:

* Extremely fast startup
* Uses native ES modules
* Faster hot module replacement (HMR)

👉 It avoids heavy bundling during development.

---

## 🔹 10. CRA vs Vite (Interview Answer)

**Answer:**

| Feature        | CRA      | Vite      |
| -------------- | -------- | --------- |
| Speed          | Slow     | Fast      |
| Setup          | Easy     | Easy      |
| Dev Experience | Moderate | Excellent |
| Maintenance    | Low      | Active    |

👉 **Conclusion:**
Vite is preferred in modern React development.

---

## 🔹 11. How do you create a React app using Vite?

**Answer:**

### Step 1: Create project

```bash id="y3m2q2"
npm create vite@latest
```

---

### Step 2: Install dependencies

```bash id="zzqjzq"
npm install
```

---

### Step 3: Run project

```bash id="3e2q2d"
npm run dev
```

👉 This starts the development server.

---

## 🔹 12. What role do tools play in React development?

**Answer:**

React itself only handles UI logic.

Tools like Vite, Webpack, and Babel:

* Prepare code for browser
* Optimize performance
* Improve developer experience

👉 Without these tools, modern React development would be difficult.

---

## 🎯 FAANG Interview Insight

👉 Strong candidates:

* Think in **components + state flow**
* Understand **tooling deeply**
* Know **why tools exist, not just how to use them**

---

## ⚡ Quick Revision

* Good dev = Components + State + Data flow
* npm/yarn = Package managers
* Browser understands only HTML/CSS/JS
* Vite = Fast modern build tool
* CRA = Old setup tool
* Bundler = Combines files
* Transpiler = Converts modern JS

---

## ⭐ Author

Huneshwar Yadav
