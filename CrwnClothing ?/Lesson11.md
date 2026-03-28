# 📘 CRWN Clothing v2 — Lesson 11 Notes

---

# 🔹 Lesson 11 — useEffect & onAuthStateChanged

**Tag:** useEffect + Auth Persistence | **Branch:** `lesson-11`

---

## 📌 What This Lesson Covers

Lesson 11 introduces `useEffect` — React's hook for side effects — and uses it to set up Firebase's `onAuthStateChanged` listener inside `UserProvider`. This one change solves two problems from Lesson 10: auth state was lost on page refresh, and `setCurrentUser` was scattered across multiple components. After this lesson, Firebase drives auth state automatically from a single place, and auth persists across page refreshes.

---

## 📁 Changed Files

```
src/
  contexts/
    user.context.jsx         ← UPDATED — useEffect + onAuthStateChangedListener
  utils/firebase/
    firebase.utils.js        ← UPDATED — onAuthStateChangedListener exported
  routes/navigation/
    navigation.component.jsx ← UPDATED — signOutHandler removed, onClick={signOutUser} direct
  components/sign-in-form/
    sign-in-form.component.jsx ← UPDATED — setCurrentUser calls removed
  components/sign-up-form/
    sign-up-form.component.jsx ← UPDATED — setCurrentUser calls removed
  App.js                     ← UPDATED — useContext + Navigate removed (simplified)
```

---

## 🔑 Key Concepts

### 1. `useEffect` — Side Effects in React

```js
useEffect(() => {
  const unsubscribe = onAuthStateChangedListener((user) => {
    if (user) createUserDocumentFromAuth(user);
    setCurrentUser(user);
  });

  return unsubscribe;   // ← cleanup function
}, []);                 // ← dependency array
```

React renders are supposed to be **pure functions** — same input → same output, no side effects. But real apps need subscriptions, API calls, timers, and DOM manipulation. `useEffect` is React's escape hatch for these.

**Timing:** `useEffect` runs **after** the browser has painted the screen — never blocks rendering.

| Hook | Timing | Use for |
|------|--------|---------|
| `useEffect` | After paint (async) | Subscriptions, API calls, timers — 99% of cases |
| `useLayoutEffect` | Before paint (sync) | DOM measurements to prevent flicker |

---

### 2. Dependency Array — Three Options

```js
useEffect(() => { ... });          // No array
useEffect(() => { ... }, []);      // Empty array
useEffect(() => { ... }, [value]); // With values
```

| Array | When effect runs |
|-------|----------------|
| No array | After **every** render |
| `[]` empty | **Once** — after initial mount only |
| `[value]` | After mount + whenever `value` changes (React uses `Object.is()` to compare) |

> ⚠️ **Infinite loop trap:** Effect with no array that updates state causes: effect → state update → re-render → effect → state update → ...
> ```js
> // ❌ INFINITE LOOP
> useEffect(() => {
>   setCount(count + 1); // causes re-render
> });                    // no array = runs after every render
> ```

> ⚠️ **Stale closure trap:** Missing a dependency means the effect captures an old (stale) value:
> ```js
> useEffect(() => {
>   const id = setInterval(() => {
>     console.log(count); // always logs 0 — stale closure!
>   }, 1000);
>   return () => clearInterval(id);
> }, []); // ← count is missing from deps
> ```
> 🔥 The ESLint `exhaustive-deps` rule catches this automatically — never ignore it.

---

### 3. Cleanup Function — Preventing Memory Leaks

```js
useEffect(() => {
  const unsubscribe = onAuthStateChangedListener((user) => {
    setCurrentUser(user);
  });

  return unsubscribe;  // ← React calls this on unmount
}, []);
```

React calls the cleanup function in two situations:
1. **Before running the effect again** (if dependencies changed)
2. **When the component unmounts**

**Without cleanup:** `onAuthStateChanged` creates a Firebase listener that holds a reference to `setCurrentUser`. When `UserProvider` unmounts, the listener is still alive — Firebase keeps firing callbacks on an unmounted component → memory leak → React warning:
> *"Can't perform a React state update on an unmounted component"*

**Every subscription needs cleanup:**
```js
// Timer
useEffect(() => {
  const id = setTimeout(() => doSomething(), 1000);
  return () => clearTimeout(id);         // cleanup
}, []);

// Event listener
useEffect(() => {
  window.addEventListener('resize', handler);
  return () => window.removeEventListener('resize', handler);  // cleanup
}, []);

// Firebase listener
useEffect(() => {
  const unsubscribe = onAuthStateChanged(auth, callback);
  return unsubscribe;                    // cleanup
}, []);
```

---

### 4. `onAuthStateChanged` — Firebase's Real-Time Observer

```js
// firebase.utils.js — new export
export const onAuthStateChangedListener = (callback) =>
  onAuthStateChanged(auth, callback);
```

`onAuthStateChanged` is Firebase's **persistent auth state observer**. It fires automatically:

| Event | When it fires |
|-------|--------------|
| App first loads | Immediately — checks persisted token in localStorage |
| User signs in | After successful sign-in |
| User signs out | After sign-out |

**The page refresh problem — solved:**

```
Lesson 10 — on page refresh:
  useState resets → currentUser = null
  User appears logged out even if they were logged in ❌

Lesson 11 — on page refresh:
  UserProvider mounts → useEffect runs → onAuthStateChanged fires
  Firebase reads JWT token from localStorage
  If valid → callback fires with user object → setCurrentUser(user) ✅
  Auth state restored transparently
```

**Token lifecycle:** Firebase stores a JWT (JSON Web Token) in localStorage automatically. `onAuthStateChanged` reads this token, validates with Google's servers if needed, and fires with the user or `null` if expired/invalid.

---

### 5. The Big Architectural Shift — Single Source of Truth

**Lesson 10 — scattered, imperative state management:**
```js
// SignInForm — had to call setCurrentUser manually
const { user } = await signInWithGooglePopup();
setCurrentUser(user);                             // ← manual

const { user } = await signInAuthUserWithEmailAndPassword(email, password);
setCurrentUser(user);                             // ← manual again

// Navigation — had to call setCurrentUser manually
const signOutHandler = async () => {
  await signOutUser();
  setCurrentUser(null);                           // ← manual
};
```

**Lesson 11 — single listener, automatic:**
```js
// UserProvider — ONE place handles ALL auth state changes
useEffect(() => {
  const unsubscribe = onAuthStateChangedListener((user) => {
    setCurrentUser(user);   // ← fires for sign-in, sign-out, AND refresh
  });
  return unsubscribe;
}, []);

// SignInForm — just calls Firebase, no state management
await signInWithGooglePopup();       // Firebase fires → listener → setCurrentUser ✅

// Navigation — direct, no wrapper needed
<span onClick={signOutUser}>SIGN OUT</span>  // Firebase fires → listener → setCurrentUser ✅
```

**Design patterns this implements:**

| Pattern | Description |
|---------|-------------|
| **Single Source of Truth** | Auth state lives in exactly one place — `UserProvider`'s listener |
| **Observer Pattern** | Firebase is the observable; `UserProvider` is the observer |
| **Inversion of Control** | Components don't control when state updates — Firebase does |
| **Event-Driven Architecture** | Auth events flow from Firebase → listener → Context → UI |

**Why this is more maintainable:**
- Adding a new sign-in method → just call the Firebase function → listener handles the rest
- Zero risk of forgetting to call `setCurrentUser` in a new component
- Single place to add logic (logging, analytics) that runs on every auth change
- Components are fully decoupled from auth state management

---

## 🔄 Complete Auth Flow — Lesson 11

```
App loads (any page)
  → UserProvider mounts
  → useEffect([]) runs once
  → onAuthStateChangedListener subscribes
  → Firebase checks localStorage for JWT
  → Token found? → callback(user) → setCurrentUser(user) → UI: logged in
  → No token?   → callback(null) → setCurrentUser(null) → UI: logged out

User signs in (any method)
  → Firebase Auth succeeds
  → onAuthStateChanged fires with user
  → setCurrentUser(user) in UserProvider
  → All Context consumers re-render

User signs out
  → signOutUser() called
  → Firebase clears session
  → onAuthStateChanged fires with null
  → setCurrentUser(null) in UserProvider
  → All Context consumers re-render

UserProvider unmounts
  → React calls cleanup: unsubscribe()
  → Firebase listener removed
  → No memory leak ✅
```

---

## 🔄 What Changed from Lesson 10 → Lesson 11

| Area | Lesson 10 | Lesson 11 |
|------|----------|-----------|
| Where `setCurrentUser` is called | SignInForm, SignUpForm, Navigation | **UserProvider only** |
| Auth on page refresh | ❌ Lost — useState resets | ✅ Persists — listener restores it |
| Firebase state sync | Manual in each component | Automatic via listener |
| Navigation sign-out | `signOutHandler` wrapper function | `onClick={signOutUser}` direct |
| New hook introduced | useContext | **useEffect** |
| New Firebase export | — | `onAuthStateChangedListener` |
| Components' auth responsibility | Call `setCurrentUser` | Just call Firebase functions |

---

## ✅ Lesson 11 Checklist

- [ ] `yarn start` → sign in → refresh page → confirm still signed in ✅
- [ ] Sign out → refresh → confirm stays signed out ✅
- [ ] Confirm SIGN OUT button calls `signOutUser` directly (no wrapper handler)
- [ ] Explain useEffect dependency array — all 3 options
- [ ] Explain what the cleanup function prevents
- [ ] Explain why `onAuthStateChanged` fires on page load (localStorage token)
- [ ] Explain why `setCurrentUser` now lives in only one place

---

## 💡 Why This Pattern Scales

Imagine adding 5 new sign-in methods: Apple, Facebook, GitHub, Twitter, phone number.

**Lesson 10 approach:** Every new sign-in component must remember to call `setCurrentUser` — 5 new places to get it wrong.

**Lesson 11 approach:** Add the Firebase function, call it in the component, done. `onAuthStateChanged` handles the state update automatically — 0 new state management code.

This is how production codebases at FAANG scale auth — a single authoritative listener, decoupled components.

> 🔥 **Real-world extension:** The same `onAuthStateChanged` listener is where you'd add: analytics tracking (log sign-in events), session monitoring (refresh expired tokens), and permission checking (load user roles from Firestore on sign-in).

---

## 🧠 FAANG Interview Q&A — Lesson 11

### Q1. What is `useEffect` for and when does it run?

**Answer:** `useEffect` is React's escape hatch for **side effects** — anything that reaches outside React's pure render cycle: subscriptions, API calls, timers, DOM manipulation. React renders are supposed to be pure functions (same input → same output). `useEffect` runs **after** the browser has painted the screen, so it never blocks rendering.

> 🔥 **useEffect vs useLayoutEffect:** `useEffect` runs after paint (async, non-blocking) — use 99% of the time. `useLayoutEffect` runs before paint (sync, blocking) — only for DOM measurements to prevent visual flicker.

---

### Q2. Explain the three dependency array options

**Answer:**
- **No array** — runs after every render. Dangerous if the effect causes a state update (infinite loop).
- **`[]` empty** — runs once after initial mount. Equivalent to `componentDidMount`.
- **`[value]`** — runs after mount + whenever `value` changes. React uses `Object.is()` to compare.

> ⚠️ **Stale closure trap:** Missing a dependency means the effect captures a stale value. The ESLint `exhaustive-deps` rule catches this — never ignore it.
> ⚠️ **Infinite loop:** Effect with no array that updates state → effect → re-render → effect → ...

---

### Q3. What does the cleanup function do and what happens without it?

**Answer:** React calls the cleanup function before running the effect again (if deps changed) OR when the component unmounts. Without cleanup, `onAuthStateChanged` creates a Firebase listener that keeps running after `UserProvider` unmounts — calling `setCurrentUser` on an unmounted component → memory leak → React warning: *"Can't perform a React state update on an unmounted component."*

Every subscription, event listener, timer, and WebSocket must return a cleanup function.

---

### Q4. What does `onAuthStateChanged` solve that Lesson 10 couldn't?

**Answer:** `onAuthStateChanged` is a **persistent observer** that fires immediately on app load — Firebase reads its persisted JWT token from localStorage and fires the callback with the stored user or `null`. In Lesson 10, `currentUser` only lived in React's `useState` — refreshing the page wiped it entirely because `useState` doesn't persist across sessions. With `onAuthStateChanged` in `useEffect([])`, the listener starts on mount and Firebase restores auth state transparently on every page load.

---

### Q5. ⭐ Hard: What architectural pattern does the Lesson 11 refactor implement?

**Answer:** It implements the **Observer Pattern** combined with **Single Source of Truth** and **Inversion of Control**. In Lesson 10, state was managed imperatively — every component that triggered auth had to manually call `setCurrentUser` — a fragile contract that breaks as the codebase grows. In Lesson 11, Firebase is the **event emitter**, `UserProvider` is the **single observer**, and components are fully decoupled. Components just call Firebase functions; Firebase emits `onAuthStateChanged` events; `UserProvider` reacts and updates Context. Adding a new sign-in method requires zero state management changes — just call the Firebase function.

> 🔥 **Pattern names:** Single Source of Truth, Observer Pattern, Inversion of Control, Event-Driven Architecture.
> 🔥 **Scale benefit:** One place to add logging, analytics, permission checking, and token refresh on every auth change.

---

## 📊 Quiz Results

| # | Question | Score | Gap to Fix |
|---|----------|-------|------------|
| Q1 | useEffect purpose & timing | ✅ Correct | Bonus: useEffect vs useLayoutEffect |
| Q2 | Three dependency array options | ✅ Correct | Know: infinite loop trap + stale closure trap |
| Q3 | Cleanup function — what & why | ✅ Correct | Know all cleanup examples: timers, event listeners, WebSockets |
| Q4 | onAuthStateChanged vs page refresh | ✅ Correct | Bonus: Firebase stores JWT in localStorage automatically |
| Q5 | Architectural pattern — Single Source of Truth | ✅ Correct | Bonus: Observer Pattern, Inversion of Control, Event-Driven Architecture |

**Total: 5/5** — Perfect score. Back on the streak. useEffect fully understood at senior level.
