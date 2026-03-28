# 📘 CRWN Clothing v2 — Lesson 10 Notes

---

# 🔹 Lesson 10 — Context API, useContext & Protected Routes

**Tag:** Context API | **Branch:** `lesson-10`

---

## 📌 What This Lesson Covers

Lesson 10 is the most architecturally significant lesson so far. The **Context API** replaces prop drilling — `currentUser` is now globally accessible to any component without passing it through props. `useContext` reads context in any component. A **protected route** redirects logged-in users away from `/auth`. Sign-out is added. Navigation conditionally shows SIGN IN vs SIGN OUT. This lesson directly solves the prop drilling problem identified back in Lesson 4.

---

## 📁 New & Changed Files

```
src/
  contexts/
    user.context.jsx              ← NEW — UserContext + UserProvider
  index.js                        ← UPDATED — wraps app in UserProvider
  App.js                          ← UPDATED — useContext + Navigate for protected route
  routes/navigation/
    navigation.component.jsx      ← UPDATED — useContext + sign out + conditional nav
  components/sign-in-form/
    sign-in-form.component.jsx    ← UPDATED — setCurrentUser after sign in
  components/sign-up-form/
    sign-up-form.component.jsx    ← UPDATED — setCurrentUser after sign up
  utils/firebase/
    firebase.utils.js             ← UPDATED — signOutUser added
```

---

## 🔗 Provider Wrapping in index.js

```jsx
render(
  <React.StrictMode>
    <BrowserRouter>
      <UserProvider>       {/* ← any component can now read currentUser */}
        <App />
      </UserProvider>
    </BrowserRouter>
  </React.StrictMode>,
  rootElement
);
```

`UserProvider` wraps the entire app — so every component in the tree can access `currentUser` via `useContext`. Order matters: `BrowserRouter` must wrap `UserProvider` (not the other way) so routing hooks work inside the Provider.

---

## 🔑 Key Concepts

### 1. Context API — Three Parts Working Together

```js
// user.context.jsx
import { createContext, useState } from 'react';

// PART 1: createContext — creates the context object
export const UserContext = createContext({
  setCurrentUser: () => null,   // default no-op function
  currentUser: null,            // default value
});

// PART 2: Provider — wraps the tree, broadcasts value
export const UserProvider = ({ children }) => {
  const [currentUser, setCurrentUser] = useState(null);
  const value = { currentUser, setCurrentUser };

  return (
    <UserContext.Provider value={value}>
      {children}
    </UserContext.Provider>
  );
};
```

```js
// PART 3: useContext — reads the value in any component
const { currentUser } = useContext(UserContext);
```

| Part | Role |
|------|------|
| `createContext(default)` | Creates the context container + sets fallback default |
| `UserContext.Provider` | Wraps subtree, broadcasts `value` to all descendants |
| `useContext(UserContext)` | Subscribes to context — re-renders when value changes |

**Together they form a publish-subscribe system:**
- Provider **publishes** the value
- `useContext` **subscribes** to it
- React handles re-renders automatically when value changes

---

### 2. Default Value — What It's Really For

```js
export const UserContext = createContext({
  setCurrentUser: () => null,
  currentUser: null,
});
```

The default value is used **only** when a component calls `useContext(UserContext)` but has **no Provider anywhere above it** in the tree — almost never happens intentionally.

**Real purposes of the default value:**
1. **Documentation** — tells developers the shape of the context without reading the Provider
2. **IDE autocomplete** — editors use it for type inference
3. **Safety net** — `() => null` prevents crashes if someone accidentally uses Context outside the Provider

> ⚠️ **Common confusion:** The default value is NOT the initial state inside the Provider. The Provider's initial state comes from `useState(null)` inside `UserProvider`. These are completely separate.

---

### 3. useContext — Direct Access Anywhere

```js
// App.js — reads currentUser for protected route
const { currentUser } = useContext(UserContext);

// Navigation — reads AND writes (sign out)
const { currentUser, setCurrentUser } = useContext(UserContext);

// SignInForm — writes only (after sign in)
const { setCurrentUser } = useContext(UserContext);

// SignUpForm — writes only (after sign up)
const { setCurrentUser } = useContext(UserContext);
```

**Before Context (prop drilling):**
```
App → passes currentUser as prop
  → Navigation → passes it down
    → NavLinks → passes it down
      → UserDisplay → finally uses it
```

**After Context:**
```
UserDisplay → useContext(UserContext) → currentUser ✅
(zero props, zero drilling, works at any depth)
```

---

### 4. Protected Route — Navigate with `replace`

```jsx
// App.js
<Route
  path='auth'
  element={
    currentUser ? <Navigate to='/' replace /> : <Authentication />
  }
/>
```

- Logged in (`currentUser` truthy) → `<Navigate to='/'>` redirects immediately
- Not logged in → `<Authentication />` renders normally

**Why `replace` is critical:**

```
Without replace:
  History stack: [/, /auth, /]
  User presses back → goes to /auth → immediately redirected to / again
  Back button is broken — user is stuck ❌

With replace:
  History stack: [/, /]    ← /auth entry replaced, not pushed
  User presses back → goes to / naturally ✅
```

> 🔥 **Always use `replace` for auth redirects** — it's the correct UX pattern for all protected routes. Without it, the back button creates redirect loops.

---

### 5. Sign Out — Two Systems, Two Updates

```js
// firebase.utils.js — new function
export const signOutUser = async () => await signOut(auth);

// Navigation — sign out handler
const signOutHandler = async () => {
  await signOutUser();        // ← clears Firebase Auth session
  setCurrentUser(null);       // ← clears React Context
};
```

**Why BOTH are necessary:**

| | `signOutUser()` | `setCurrentUser(null)` |
|--|-----------------|----------------------|
| What it updates | Firebase Auth session (Google's servers) | React Context (in-memory UI state) |
| Effect | Invalidates token, clears local storage, `auth.currentUser → null` | Triggers re-render in all consumers |
| Without it | Firebase thinks user is signed out but UI still shows SIGN OUT | UI is correct but Firebase session still active (security risk) |

> 🔥 **The fundamental split:** Firebase manages **authentication state** (server-side security). React Context manages **UI state** (what components render). They are separate systems and both must be updated.

---

### 6. Conditional Navigation — SIGN IN vs SIGN OUT

```jsx
{currentUser ? (
  <span className='nav-link' onClick={signOutHandler}>
    SIGN OUT
  </span>
) : (
  <Link className='nav-link' to='/auth'>
    SIGN IN
  </Link>
)}
```

- `currentUser` is truthy (user logged in) → SIGN OUT button renders
- `currentUser` is null → SIGN IN link renders
- No props needed — Navigation reads `currentUser` directly from Context
- React re-renders Navigation automatically when `currentUser` changes

---

### 7. Context Limitations — The Full Picture

Context API is powerful but has real trade-offs:

| Limitation | Detail |
|-----------|--------|
| **Performance** | Every `useContext` consumer re-renders when value changes — even if the data they use didn't change |
| **New object on every render** | `const value = { currentUser, setCurrentUser }` creates a new object reference each render — all consumers re-render unnecessarily |
| **Not for high-frequency updates** | Fine for auth (changes rarely). Terrible for mouse position (changes hundreds of times/second) |
| **No DevTools** | Unlike Redux, no time-travel debugging or action history |
| **Scale complexity** | Many contexts in large apps becomes hard to manage |

**Performance fix with useMemo:**
```js
const value = useMemo(
  () => ({ currentUser, setCurrentUser }),
  [currentUser]  // only creates new object when currentUser changes
);
```

**When to use Context vs Redux/Zustand:**

| Use Context for | Use Redux/Zustand for |
|-----------------|----------------------|
| Auth state | Shopping cart |
| Theme (light/dark) | Complex async operations |
| Locale/language | High-frequency updates |
| Feature flags | Large shared state with many writers |

---

## 🔄 Complete Auth State Flow — Lesson 10

```
User signs in (any method)
  → Firebase returns { user }
  → setCurrentUser(user)              ← updates UserContext
  → All consumers re-render:
      Navigation: shows SIGN OUT
      App.js /auth: redirects to /    (protected route)

User visits /auth while logged in
  → currentUser is truthy
  → <Navigate to='/' replace />       ← redirect, history entry replaced
  → Back button works naturally

User signs out
  → signOutUser()                     ← Firebase session cleared
  → setCurrentUser(null)              ← Context cleared
  → All consumers re-render:
      Navigation: shows SIGN IN
      App.js /auth: shows Authentication
```

---

## 🔄 What Changed from Lesson 9 → Lesson 10

| Area | Lesson 9 | Lesson 10 |
|------|----------|-----------|
| User state location | Local to each component | Global UserContext |
| Sharing currentUser | Not shared | `useContext` anywhere |
| After sign-in | No global state update | `setCurrentUser(user)` |
| /auth route | Always accessible | Protected — redirects if logged in |
| Navigation | Always shows SIGN IN | Conditional: SIGN IN or SIGN OUT |
| Sign out | Not implemented | `signOutUser` + `setCurrentUser(null)` |
| New folder | — | `contexts/` |
| New hook used | useState only | useState + useContext |

---

## ✅ Lesson 10 Checklist

- [ ] `yarn start` → sign in → navbar shows SIGN OUT
- [ ] While logged in, navigate to `/auth` → confirm redirect to `/`
- [ ] Click back button → confirm it goes to a sensible page (not a redirect loop)
- [ ] Click SIGN OUT → navbar shows SIGN IN
- [ ] Navigate to `/auth` after sign out → Authentication page shows normally
- [ ] Explain the 3 parts of Context API without notes
- [ ] Explain why `replace` is needed on Navigate
- [ ] Explain why both `signOutUser()` AND `setCurrentUser(null)` are needed

---

## 💡 Why Context, Not Props?

**Prop drilling for `currentUser` would look like this:**
```jsx
// Without Context — painful
<App currentUser={currentUser}>
  <Navigation currentUser={currentUser}>
    <NavLinks currentUser={currentUser}>
      <UserDisplay currentUser={currentUser} />  {/* finally used here */}
    </NavLinks>
  </Navigation>
</App>
```

Every intermediate component carries a prop it doesn't use — just to pass it down. If `currentUser` shape changes, every file must be updated.

**With Context:**
```jsx
const { currentUser } = useContext(UserContext);  // anywhere, directly
```

Zero props, zero coupling, zero pain.

> 🔥 **This is exactly what Context was designed for:** global data that many components need at different levels — auth state, theme, locale. For anything more complex (cart, orders, async operations), Redux or Zustand is the right tool.

---

## 🧠 FAANG Interview Q&A — Lesson 10

### Q1. Explain the three parts of Context API

**Answer:** `createContext(defaultValue)` creates the context object — a container with a `Provider` component and a default fallback value. `UserContext.Provider` wraps a subtree and broadcasts a `value` prop to all descendants. `useContext(UserContext)` subscribes a component to the context — returning the current value and triggering a re-render whenever it changes.

Together they form a **publish-subscribe system**: Provider publishes, useContext subscribes, React handles re-renders automatically.

> 🔥 **Performance:** Every `useContext` consumer re-renders when context value changes — even if the specific data they use didn't change. Split large contexts into smaller focused ones (UserContext, CartContext, ThemeContext).

---

### Q2. When is the `createContext` default value actually used?

**Answer:** Only when a component calls `useContext(UserContext)` but has **no Provider anywhere above it** in the tree. In practice this almost never happens intentionally. The real purpose is documentation (shape of the context), IDE autocomplete, and safety (`() => null` prevents crashes outside the Provider).

> ⚠️ **Common confusion:** The default value is NOT the Provider's initial state. The Provider's initial state comes from `useState(null)` inside `UserProvider`. Completely separate concepts.

---

### Q3. What does `replace` do on `<Navigate>` and why does it matter?

**Answer:** Without `replace`, `<Navigate to='/'>` **pushes** a new entry onto the browser history stack — making the stack `[/, /auth, /]`. Clicking back goes to `/auth`, which immediately redirects to `/` again — the back button is broken in a redirect loop. With `replace`, the `/auth` entry is **replaced** by `/` — stack becomes `[/, /]`. Clicking back goes to the page before `/auth`, working naturally.

```
Without replace → history: [/, /auth, /] → back button loops ❌
With replace    → history: [/, /]        → back button works ✅
```

> 🔥 **Always use `replace` for all auth redirects** — it's the standard pattern.

---

### Q4. Why are both `signOutUser()` AND `setCurrentUser(null)` needed?

**Answer:** They update two different systems. `signOutUser()` calls Firebase's `signOut(auth)` — invalidates the server session, clears the token from local storage, sets `auth.currentUser` to null. But React Context still holds the old user — UI still shows SIGN OUT and protected routes still think someone is logged in. `setCurrentUser(null)` clears the Context — triggering re-renders in Navigation (shows SIGN IN) and App.js (unblocks `/auth`).

Firebase manages **authentication security**. Context manages **UI state**. Both must be updated independently.

---

### Q5. ⭐ Hard: What problem does Context solve and what are its limitations?

**Answer:** Without Context, `currentUser` would need prop drilling — passed from App through Navigation through NavLinks to UserDisplay, with every intermediate component carrying a prop it doesn't use. Any shape change requires updating every file in the chain.

Context solves this by making `currentUser` globally accessible — any component calls `useContext(UserContext)` for direct access, zero drilling, zero coupling.

**Limitations:**
1. **Performance** — every consumer re-renders when the context value changes. `const value = { currentUser, setCurrentUser }` creates a new object every render, causing unnecessary re-renders. Fix: `useMemo`.
2. **Not for high-frequency updates** — fine for auth (rare changes), terrible for mouse position (hundreds per second).
3. **No DevTools** — no time-travel debugging or action history like Redux.
4. **Scale** — managing many contexts in large apps gets complex.

Use Context for slow-changing global data (auth, theme, locale). Use Redux/Zustand for complex state, async operations, or high-frequency updates.

---

## 📊 Quiz Results

| # | Question | Score | Gap to Fix |
|---|----------|-------|------------|
| Q1 | Three parts of Context API | ✅ Correct | Bonus: performance — all consumers re-render on value change |
| Q2 | When default value is used | ✅ Correct | Remember: default ≠ Provider's initial state |
| Q3 | `replace` on Navigate | ❌ Missed | `replace` prevents redirect loops by replacing history entry not pushing |
| Q4 | Why both signOutUser + setCurrentUser(null) | ✅ Correct | — |
| Q5 | Context limitations | ❌ Partial | Context re-renders all consumers; not for high-frequency updates; no DevTools; use Redux for complex state |

**Total: 3/5** — Good foundation. Two key gaps: `replace` behaviour and Context performance limitations. Both are senior-level interview topics — review before next lesson.
