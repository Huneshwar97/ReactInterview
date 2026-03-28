# 📘 CRWN Clothing v2 — Lesson 6 Notes

---

# 🔹 Lesson 6 — Nested Routes, Firebase & Google Auth

**Tag:** Routing + Auth | **Branch:** `lesson-6`

---

## 📌 What This Lesson Covers

Lesson 6 is the biggest lesson so far. It introduces **nested routes** with the Layout Route pattern, `<Outlet />` for child route injection, `<Link>` for client-side navigation, `<Fragment>` for DOM-clean wrappers, SVG imported as a React component, and **Firebase** for Google OAuth authentication. `App.js` now has multiple routes and a persistent navbar across all pages.

---

## 📁 Folder Structure

```
src/
  assets/
    crown.svg                               ← NEW — brand logo as SVG
  routes/
    home/
      home.component.jsx                    ← now includes <Outlet />
    navigation/
      navigation.component.jsx              ← NEW — layout route with navbar
      navigation.styles.scss                ← NEW
    sign-in/
      sign-in.component.jsx                 ← NEW — Google sign-in button
  components/
    category-item/                          ← unchanged
    directory/                              ← unchanged
  utils/
    firebase/
      firebase.utils.js                     ← NEW — Firebase init & auth helpers
  App.js                                    ← now has nested routes
  index.js                                  ← unchanged
  index.scss                                ← adds global a{} and box-sizing
```

---

## 📦 New Dependency: `firebase`

```json
{
  "dependencies": {
    "firebase": "9.6.6"    ← NEW
  }
}
```

Firebase v9 uses a **modular (tree-shakeable) API** — you import only the functions you need rather than the entire SDK. This significantly reduces bundle size.

```js
// v9 modular — only imports what's used
import { initializeApp } from 'firebase/app';
import { getAuth, signInWithPopup } from 'firebase/auth';
```

---

## 🔗 Full Route & Component Tree

```
BrowserRouter  (index.js)
  └── App
        └── Routes
              └── Route path="/"  →  Navigation   ← layout, always renders
                    ├── Route index  →  Home       ← renders at '/'
                    ├── Route 'shop' →  Shop       ← renders at '/shop'
                    └── Route 'sign-in' → SignIn   ← renders at '/sign-in'
                                   ↑
                    <Outlet /> in Navigation renders whichever child matches
```

---

## 🔑 Key Concepts

### 1. Nested Routes — The Layout Route Pattern

```jsx
// App.js
<Routes>
  <Route path='/' element={<Navigation />}>       {/* parent — always renders */}
    <Route index element={<Home />} />             {/* renders at '/' */}
    <Route path='shop' element={<Shop />} />       {/* renders at '/shop' */}
    <Route path='sign-in' element={<SignIn />} />  {/* renders at '/sign-in' */}
  </Route>
</Routes>
```

`Navigation` is the **layout route** — it wraps all child routes. It renders on **every page** because it's the parent. Child routes render inside it wherever `<Outlet />` is placed.

This is how you get a **persistent navbar** across all pages without copy-pasting it into every route component.

**Flat routes (Lesson 5) vs Nested routes (Lesson 6):**

```
Lesson 5 — flat:                    Lesson 6 — nested:
<Routes>                            <Routes>
  <Route path='/' …/>                 <Route path='/' element={<Nav />}>
</Routes>                               <Route index element={<Home />} />
                                        <Route path='shop' … />
                                      </Route>
                                    </Routes>
```

---

### 2. `<Outlet />` — Child Route Injection Point

```jsx
// navigation.component.jsx
const Navigation = () => (
  <Fragment>
    <div className='navigation'>
      {/* navbar — always visible */}
    </div>
    <Outlet />    {/* ← child route renders HERE */}
  </Fragment>
);
```

`<Outlet />` is a **placeholder** — React Router replaces it at render time with whichever child route matches the current URL:

| URL | What renders in `<Outlet />` |
|-----|------------------------------|
| `/` | `<Home />` |
| `/shop` | `<Shop />` |
| `/sign-in` | `<SignIn />` |

**Advanced:** You can pass context through Outlet:
```jsx
<Outlet context={{ user }} />  // parent passes data to child route
// child reads it with:
const { user } = useOutletContext();
```

**Deeply nested layouts** are also possible — a child route can itself be a layout with its own `<Outlet />`:
```
App Layout → Shop Layout → Product Detail
```

---

### 3. `<Link>` — Client-Side Navigation

```jsx
import { Link } from 'react-router-dom';

<Link className='logo-container' to='/'>
  <CrwnLogo className='logo' />
</Link>
<Link className='nav-link' to='/shop'>SHOP</Link>
<Link className='nav-link' to='/sign-in'>SIGN IN</Link>
```

**Never use `<a href>` in React Router apps:**

| | `<a href='/shop'>` | `<Link to='/shop'>` |
|--|-------|------|
| Network request? | ✅ Yes — full HTTP request | ❌ No |
| Page reload? | ✅ Yes — re-downloads JS bundle | ❌ No |
| React state destroyed? | ✅ Yes | ❌ No |
| Browser history? | ✅ Yes | ✅ Yes |
| DOM output | `<a>` tag | `<a>` tag (same!) |

`<Link>` renders as an `<a>` in the DOM — so it's still keyboard accessible and right-clickable — but it intercepts the click, calls `history.pushState()`, and tells React Router to re-render the matching route.

> 🔥 **`<NavLink>` bonus:** `<NavLink>` automatically adds an `active` CSS class when its `to` path matches the current URL — perfect for styling active nav items without manual logic.

---

### 4. `<Fragment>` — Invisible Wrapper

```jsx
import { Fragment } from 'react';

return (
  <Fragment>
    <div className='navigation'>...</div>
    <Outlet />
  </Fragment>
);

// Shorthand — identical except can't use key prop
return (
  <>
    <div className='navigation'>...</div>
    <Outlet />
  </>
);
```

React requires every component to return a **single root element** because JSX compiles to `React.createElement()` which returns one element. Fragment groups elements without adding any real DOM node.

**When you MUST use `<Fragment>` (not `<>`):**
```jsx
// Rendering lists — shorthand <> cannot take a key prop
{items.map(item => (
  <Fragment key={item.id}>
    <dt>{item.term}</dt>
    <dd>{item.description}</dd>
  </Fragment>
))}
```

---

### 5. SVG as React Component

```jsx
import { ReactComponent as CrwnLogo } from '../../assets/crown.svg';

<CrwnLogo className='logo' />
```

CRA's webpack config includes `@svgr/webpack` which transforms SVG files into React components at build time.

| | `<img src={logo}>` | `{ ReactComponent as Logo }` |
|--|----|----|
| Apply `className` | ✅ (to `<img>`) | ✅ (to SVG root element) |
| Style SVG internals | ❌ Opaque image | ✅ Via CSS cascade |
| Animate SVG paths | ❌ | ✅ |
| Add `onClick` | ✅ (to `<img>`) | ✅ (to SVG element) |
| Accessibility props | Limited | Full control |

> 🔥 **FAANG use case:** Icon systems use this pattern — one SVG component, different `className` props control size and color. No extra icon library needed.

---

### 6. 🔥 Firebase — Google Authentication

#### Setup

```js
// firebase.utils.js
import { initializeApp } from 'firebase/app';
import { getAuth, signInWithPopup, GoogleAuthProvider } from 'firebase/auth';

const firebaseConfig = {
  apiKey: '...',        // ← safe to be public — just identifies the project
  authDomain: '...',
  projectId: '...',
};

const firebaseApp = initializeApp(firebaseConfig);   // connect to Firebase project
const provider = new GoogleAuthProvider();           // Google as identity provider

provider.setCustomParameters({ prompt: 'select_account' }); // always show account chooser

export const auth = getAuth();
export const signInWithGooglePopup = () => signInWithPopup(auth, provider);
```

> 🔐 **Security note:** The `firebaseConfig` object is **safe to be public** — it identifies the Firebase project, like a public address. Real security comes from Firebase Security Rules on the database, not from hiding the config.

#### Sign-In Flow — Step by Step

```
1. User clicks "Sign in with Google Popup" button
        ↓
2. logGoogleUser() called (async function)
        ↓
3. signInWithGooglePopup() → signInWithPopup(auth, provider)
        ↓
4. Google OAuth 2.0 popup window opens
        ↓
5. User selects Google account
        ↓
6. Google issues OAuth token to Firebase
        ↓
7. Firebase validates token with Google's servers
        ↓
8. Firebase returns UserCredential object:
   {
     user: {
       uid: 'abc123',
       email: 'user@gmail.com',
       displayName: 'John Doe',
       photoURL: 'https://...',
       accessToken: '...'
     }
   }
        ↓
9. createUserProfileDocument(response) — currently just console.log()
   (stub — later lessons write this to Firestore)
```

#### SignIn Component

```jsx
const SignIn = () => {
  const logGoogleUser = async () => {
    const response = await signInWithGooglePopup();  // await the Promise
    createUserProfileDocument(response);             // stub for now
  };

  return (
    <div>
      <h1>Sign In Page</h1>
      <button onClick={logGoogleUser}>Sign in with Google Popup</button>
    </div>
  );
};
```

`logGoogleUser` is `async` because `signInWithPopup` returns a **Promise** — it must wait for the user to complete the Google flow before continuing. `await` pauses execution until the Promise resolves.

---

### 7. Global SCSS Updates — `index.scss`

```scss
a {
  text-decoration: none;   /* removes underline from ALL <a> tags globally */
  color: black;
}

* {
  box-sizing: border-box;  /* padding included in element width — prevents layout bugs */
}
```

`box-sizing: border-box` is the single most impactful global CSS rule — without it, adding padding to an element makes it wider than expected, breaking layouts. This is a standard reset applied in every production app.

---

## 🔄 What Changed from Lesson 5 → Lesson 6

| Area | Lesson 5 | Lesson 6 |
|------|----------|----------|
| Route structure | Flat — one Route | Nested — Navigation wraps all |
| Persistent navbar | ❌ None | ✅ Navigation layout route |
| Child rendering | Direct | Via `<Outlet />` |
| Navigation links | None | `<Link>` components |
| Auth | None | Firebase Google Sign-In |
| New dependency | react-router-dom | firebase |
| SVG usage | None | Imported as React component |
| New folders | routes/ | navigation/, sign-in/, utils/firebase/ |
| Global CSS | Basic body styles | Adds `a {}` and `box-sizing` reset |

---

## ✅ Lesson 6 Checklist

- [ ] `yarn` — installs new `firebase` dependency
- [ ] `yarn start` → navbar appears on every page
- [ ] Navigate between `/`, `/shop`, `/sign-in` — navbar stays, content swaps
- [ ] Click "Sign in with Google Popup" → Google account chooser appears
- [ ] Check browser console — UserCredential object logged after sign-in
- [ ] Trace: button click → async fn → signInWithPopup → Google popup → UserCredential
- [ ] Explain `<Outlet />` to yourself — what replaces it and when
- [ ] Explain why `<Link>` not `<a href>`

---

## 💡 Why the Layout Route Pattern?

Without the layout route pattern, you'd have to include `<Navigation />` manually in every route component — duplicating it in Home, Shop, SignIn, and every future page. If the navbar changes, you'd update every file.

With the layout route pattern:
- Navbar defined **once** in Navigation
- All child routes automatically get it
- Adding a new page = add one `<Route>` in App.js, no navbar work needed

> 🔥 **This pattern scales to multiple layout levels:** A top-level layout (navbar + footer) can contain an authenticated layout (sidebar) which contains a settings layout (tabs) — each level uses `<Outlet />`.

---

## 🧠 FAANG Interview Q&A — Lesson 6

### Q1. What does `<Outlet />` do and what renders there at '/shop'?

**Answer:** `<Outlet />` is React Router's **child route injection point** — a placeholder that gets replaced at render time with whichever child route matches the current URL. When the URL is `/shop`, React Router matches `<Route path='shop' element={<Shop />}>` and renders `<Shop />` in place of `<Outlet />`. The parent `Navigation` component always renders — giving a persistent navbar — while only the Outlet content swaps.

```
URL: /shop
  Navigation renders  ← always (layout route)
    <Outlet />        ← replaced by <Shop />
```

> 🔥 **Bonus:** `<Outlet context={value} />` passes data to child routes; children read it with `useOutletContext()`.

---

### Q2. Why use `<Link>` instead of `<a href>`?

**Answer:** A plain `<a href='/shop'>` makes the browser do a full HTTP request — re-downloads the HTML, re-executes the entire JS bundle, reinitialises React, and destroys all in-memory state. `<Link to='/shop'>` intercepts the click event, calls `history.pushState()` to update the URL, and tells React Router to re-render the matching route — no network request, no reload, no state loss.

In the DOM, `<Link>` still renders as an `<a>` tag — so it's keyboard accessible and right-clickable — but the click is handled by JavaScript.

> 🔥 **`<NavLink>`:** Automatically adds an `active` CSS class when its `to` matches the current URL — perfect for highlighted nav items without manual logic.

---

### Q3. What problem does Fragment solve and what's the shorthand?

**Answer:** React requires every component to return a **single root element** because JSX compiles to `React.createElement()` which returns one value. Before Fragment, developers wrapped everything in an extra `<div>` — polluting the DOM, potentially breaking flexbox/grid layouts, and adding unnecessary nodes. Fragment groups elements without writing any real DOM node.

```jsx
// Full — can accept key prop (required for lists)
<Fragment key={item.id}>...</Fragment>

// Shorthand — identical but cannot accept key prop
<>...</>
```

> 🔥 **Performance:** In large lists where each item renders multiple elements, removing wrapper divs via Fragment meaningfully reduces DOM node count.

---

### Q4. What does `{ ReactComponent as CrwnLogo }` give you over `<img src>`?

**Answer:** CRA's webpack includes `@svgr/webpack` which transforms SVG files into React components at build time. This gives you a real React component — you can pass `className`, `style`, `onClick`, `aria-label`, and any prop. The SVG's internal paths inherit styles via CSS cascade, so you can change fill color with just CSS.

With `<img src={logo}>`, the SVG is an opaque image — you can't style its internals, animate its paths, or control accessibility attributes on SVG elements.

> 🔥 **FAANG pattern:** Icon systems use this — one SVG component, `className` controls size and color. No extra icon library needed.

---

### Q5. ⭐ Hard: Walk through the complete Google Sign-In flow

**Answer:**

```
1. Button click → logGoogleUser() (async)
2. signInWithGooglePopup() → signInWithPopup(auth, provider)
3. Google OAuth 2.0 popup opens
4. User selects Google account
5. Google issues OAuth token to Firebase
6. Firebase validates token with Google's servers
7. Firebase returns UserCredential:
   { user: { uid, email, displayName, photoURL, accessToken } }
8. createUserProfileDocument(response) — currently console.log() stub
   (later lessons: writes user to Firestore)
```

`logGoogleUser` is `async` because `signInWithPopup` returns a **Promise** — must await the user completing the Google flow before continuing.

`firebaseConfig` (apiKey etc.) is **safe to be public** — it just identifies the Firebase project. Real security comes from Firebase Security Rules on the database.

> 🔥 **`createUserProfileDocument` is a stub** — in later lessons it uses Firestore's `doc()` and `setDoc()` to persist user data beyond what Firebase Auth stores (e.g. cart, orders, preferences).

---

## 🔬 firebase.utils.js — Line by Line Deep Dive

### Line 1: Import `initializeApp`

```js
import { initializeApp } from 'firebase/app';
```

- Imports the `initializeApp` function from Firebase's **core package**
- `firebase/app` is the foundation — every Firebase service builds on top of it
- `initializeApp` takes your config object and creates a **connection** to your Firebase project in the cloud
- Without this, nothing else in Firebase works — must run before `getAuth()`, `getFirestore()`, etc.
- Calling `initializeApp` twice with the same config throws an error — runs **once at module load time**

---

### Lines 2-7: Import Auth Functions

```js
import {
  getAuth,
  signInWithRedirect,
  signInWithPopup,
  GoogleAuthProvider,
} from 'firebase/auth';
```

| Import | Purpose |
|--------|---------|
| `getAuth` | Gets the Firebase Auth instance — the object that manages all auth state |
| `signInWithRedirect` | Sign in by **redirecting** the whole page to Google, then back |
| `signInWithPopup` | Sign in via a **popup window** — page stays open |
| `GoogleAuthProvider` | Tells Firebase *which* identity provider to use — Google in this case |

> ⚠️ `signInWithRedirect` is imported but **never used** in this lesson — it's there for reference. On mobile, popups are often blocked by browsers, so redirect is preferred for mobile apps.

---

### Lines 9-16: Firebase Config Object

```js
const firebaseConfig = {
  apiKey: 'AIzaSyDDU4V-_QV3M8GyhC9SVieRTDM4dbiT0Yk',
  authDomain: 'crwn-clothing-db-98d4d.firebaseapp.com',
  projectId: 'crwn-clothing-db-98d4d',
  storageBucket: 'crwn-clothing-db-98d4d.appspot.com',
  messagingSenderId: '626766232035',
  appId: '1:626766232035:web:506621582dab103a4d08d6',
};
```

| Key | Meaning |
|-----|---------|
| `apiKey` | Identifies your app to Google's APIs — **not a secret** |
| `authDomain` | Domain Firebase uses for OAuth redirects |
| `projectId` | Your unique Firebase project name |
| `storageBucket` | Where Firebase Storage files live |
| `messagingSenderId` | Used for Firebase Cloud Messaging (push notifications) |
| `appId` | Unique ID for this specific web app within the project |

> 🔐 **Safe to commit to GitHub.** Firebase security comes from **Firestore Security Rules**, not from hiding this config. Think of it like a public address — knowing the address doesn't give access to what's inside.

---

### Line 18: Initialize the App

```js
const firebaseApp = initializeApp(firebaseConfig);
```

- Connects your React app to the Firebase project in the cloud
- Returns a `FirebaseApp` instance — the root object representing your connection
- Must happen **before** calling `getAuth()`, `getFirestore()`, or any other Firebase service
- Should only run **once** — at module load time when the file is first imported

---

### Line 20: Create the Google Auth Provider

```js
const provider = new GoogleAuthProvider();
```

- Creates an instance of Google's identity provider
- Tells Firebase Auth: *"Use Google as the sign-in method"*
- Firebase supports many providers — swap this line to change identity provider:

```js
// Other providers — same pattern
new FacebookAuthProvider();
new GithubAuthProvider();
new TwitterAuthProvider();
```

---

### Lines 22-24: Custom Parameters

```js
provider.setCustomParameters({
  prompt: 'select_account',
});
```

- Passes extra options to Google's OAuth consent screen
- `prompt: 'select_account'` forces Google to **always show the account chooser** — even if the user is already signed in

| `prompt` value | Behaviour |
|----------------|-----------|
| `'select_account'` | Always show account picker |
| `'consent'` | Always show permission consent screen |
| `'none'` | Never show UI — fails if not already signed in |

> Without this, Google might silently sign in with the last used account — bad for users with multiple Google accounts (personal vs work).

---

### Line 26-28: createUserProfileDocument (Stub)

```js
export const createUserProfileDocument = async (userAuth, additionalData) => {
```
- `export` — makes it importable in SignIn component
- `async` — will eventually do async Firestore database writes
- `userAuth` — the Firebase `UserCredential` object returned after sign-in
- `additionalData` — extra fields to save beyond what Firebase Auth stores

```js
  if (!userAuth) return;
```
- **Guard clause** — if `userAuth` is null/undefined, exit immediately
- Prevents errors if called before authentication completes
- Pattern: **fail fast, fail early**

```js
  console.log(userAuth);
```
- Currently just logs the `UserCredential` to the browser console
- **This is a stub** — in later lessons becomes:

```js
  // Future Firestore write:
  const userRef = doc(db, 'users', userAuth.uid);
  await setDoc(userRef, { email, displayName, createdAt, ...additionalData });
```

---

### Lines 30-31: Export Auth & Sign-In Function

```js
export const auth = getAuth();
```
- `getAuth()` returns the **Auth instance** for this Firebase app
- Tracks sign-in state across the app
- Exported so other files can call `onAuthStateChanged(auth, callback)` in later lessons
- There's only ever **one** Auth instance per Firebase app — `getAuth()` always returns the same one

```js
export const signInWithGooglePopup = () => signInWithPopup(auth, provider);
```
- A **wrapper function** — hides Firebase implementation details from components
- `signInWithPopup(auth, provider)` does the actual work:
  - `auth` — the Auth instance (knows the Firebase project)
  - `provider` — the GoogleAuthProvider (knows to open Google's OAuth)
- Returns a **Promise** → resolves with `UserCredential` when user completes sign-in
- The SignIn component **never needs to import** `auth` or `provider` directly — clean separation

> 🔥 **This is the Facade Pattern** — complex Firebase setup hidden behind simple exported functions. If you ever switch from Firebase to AWS Cognito, you only change this one file, not every component. This file is a **service layer**.

---

### Complete Flow — Code Connected to Execution

```
firebase.utils.js loads (once, at import time)
  → initializeApp(firebaseConfig)     line 18 — connects to Firebase cloud
  → new GoogleAuthProvider()          line 20 — creates Google provider
  → provider.setCustomParameters()    line 22 — always show account chooser
  → getAuth()                         line 30 — gets auth instance

User clicks "Sign in with Google"
  → logGoogleUser() in SignIn         async function
  → signInWithGooglePopup()           line 31 — our wrapper
  → signInWithPopup(auth, provider)   Firebase function
  → Google OAuth popup opens
  → User selects account
  → Google → Firebase: OAuth token
  → Firebase validates with Google
  → Returns UserCredential {
      user: { uid, email, displayName, photoURL, accessToken }
    }
  → createUserProfileDocument()       line 26 — stub, console.logs for now
                                       Later: writes to Firestore
```

---



| # | Question | Score | Gap to Fix |
|---|----------|-------|------------|
| Q1 | `<Outlet />` — what it does and what renders at '/shop' | ✅ Correct | Bonus: learn `useOutletContext()` |
| Q2 | `<Link>` vs `<a href>` | ✅ Correct | Bonus: know `<NavLink>` adds `active` class |
| Q3 | Fragment — problem solved & shorthand | ✅ Correct | Remember: `<>` can't take `key` prop — use `<Fragment key>` in lists |
| Q4 | SVG as React Component — what it gives you | ✅ Correct | — |
| Q5 | Full Google Sign-In flow | ✅ Correct | Bonus: mention firebaseConfig is safe to be public + createUserProfileDocument is a stub |

**Total: 5/5** — Perfect score. Excellent grasp of nested routing, Firebase Auth, and React fundamentals.
