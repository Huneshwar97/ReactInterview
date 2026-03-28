🎯 Score: 4/5 — Excellent! Almost Perfect! 💪
You got Q5 wrong — and that's actually the most important one for FAANG. Let's break them all down.

Q1 ✅ — Props are Read-Only
Correct! Here's the senior engineer version:

"Props are immutable by design — they enforce unidirectional data flow. If a child could mutate props, it would create unpredictable side effects since multiple components might share the same data. If a child needs to trigger a change, it should call a callback function passed down as a prop — like onChange or onDelete — and let the parent update its own state, which then flows back down."

🔥 FAANG follow-up they'll ask: "Then how does a child ever communicate back to the parent?"
Answer: Lifting state up — parent passes a callback function as a prop, child calls it.

Q2 ✅ — Double Curly Braces
Correct! Level it up:

"In JSX, a single {} is an expression slot — it means 'evaluate this JavaScript'. The style prop expects a JavaScript object, not a CSS string. So you write style={{ key: value }} — the outer braces are the JSX expression escape, the inner braces are the object literal. This is different from Angular or Vue which use their own template syntax."

🔥 Common mistake to mention: Beginners often write style="color: red" like HTML — this throws a React error because style must be an object, not a string.

Q3 ✅ — Single Responsibility
Correct! Executive-level answer:

"Component extraction follows the Single Responsibility Principle from SOLID. In practice this means: if a component needs to change for more than one reason, it should be split. App owns data and list logic; CategoryItem owns card rendering. This also enables reusability — CategoryItem can now be used anywhere in the app, and testability — I can unit test it in isolation with any props I want."

🔥 Bonus: Mention that this also maps to Atomic Design — the pattern Facebook/Airbnb use where UI is broken into atoms → molecules → organisms.

Q4 ✅ — Two-Step Destructuring
Correct! Add this nuance:

"Deep destructuring ({ category: { imageUrl, title } }) is syntactically valid JavaScript but has a real readability cost — especially in code reviews. The two-step approach also gives you access to the full category object if you need it later, without refactoring the signature. At FAANG, code is read far more than it's written, so clarity always wins over cleverness."

🔥 Bonus: In TypeScript (used at most FAANG companies), the two-step approach also makes type annotations cleaner.

Q5 ❌ — What React Does Internally (MOST IMPORTANT!)
Your answer: Re-renders the entire page from scratch ❌
Correct answer: Runs the reconciliation algorithm — diffs the virtual DOM and patches only changed real DOM nodes ✅
This is the core of how React works. Here's exactly what happens step by step:
New prop received by CategoryItem
        ↓
React calls CategoryItem() function again
        ↓
Returns a NEW virtual DOM tree (plain JS objects)
        ↓
React DIFFS new vDOM vs previous vDOM  ← "Reconciliation"
        ↓
Finds ONLY what changed (e.g. backgroundImage URL)
        ↓
Patches ONLY that one attribute in the real DOM
        ↓
Browser repaints just that element

"React never touches the real DOM directly unless it has to. It maintains a virtual DOM — a lightweight JS object tree — and on every re-render it diffs the new virtual DOM against the previous snapshot using the reconciliation algorithm. Only the minimal set of real DOM mutations are applied. This is why React is fast — real DOM operations are expensive, virtual DOM diffing is cheap."

🔥 Why "entire page from scratch" is wrong: That's what old jQuery apps did — and it caused massive performance problems. React was invented specifically to solve this.
🔥 Key term to use: "Fiber" — React 16+ uses the Fiber reconciler, which can pause, prioritize, and resume rendering work. Mentioning this at FAANG shows you know React's internals.

📊 Updated FAANG Readiness
TopicLevelProps & unidirectional flow🟢 StrongJSX syntax internals🟢 StrongComponent architecture / SOLID🟢 StrongDestructuring best practices🟢 StrongVirtual DOM & Reconciliation🔴 Must study — asked at every FAANG React interview

🎯 One Thing To Do Before Lesson 4
Memorize this sentence:

"React re-renders the component function, produces a new virtual DOM, diffs it against the previous one via reconciliation, and patches only the changed nodes in the real DOM."

That one sentence alone will impress interviewers
