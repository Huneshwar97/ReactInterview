# 📘 CRWN Clothing v2 — Lesson 9 Notes

---

# 🔹 Lesson 9 — Sign-In Form, Authentication Route & Email Sign-In

**Tag:** Auth Refactor | **Branch:** `lesson-9`

---

## 📌 What This Lesson Covers

Lesson 9 is an architectural refactor. The messy `/sign-in` route from Lesson 8 is replaced with a clean `/auth` route containing a new `Authentication` layout component. Sign-in logic is extracted into a proper `SignInForm` component. Email sign-in is added via Firebase. Error handling is upgraded to a `switch` statement. The result: both sign-in and sign-up live side-by-side on one page, and every piece has a single clear responsibility.

---

## 📁 New & Changed Files

```
src/
  routes/
    authentication/
      authentication.component.jsx    ← NEW — layout route, composes SignInForm + SignUpForm
      authentication.styles.scss      ← NEW — flexbox side-by-side layout
    navigation/
      navigation.component.jsx        ← UPDATED — /sign-in link → /auth
  components/
    sign-in-form/
      sign-in-form.component.jsx      ← NEW — extracted from old SignIn route
      sign-in-form.styles.scss        ← NEW
  utils/firebase/
    firebase.utils.js                 ← UPDATED — signInAuthUserWithEmailAndPassword added
  App.js                              ← UPDATED — 'sign-in' route → 'auth' route
```

**Removed:** `routes/sign-in/` folder entirely.

---

## 🔗 Full Component Tree — Lesson 9

```
BrowserRouter
  └── App
        └── Routes
              └── Route '/'  →  Navigation
                    ├── Route index     →  Home
                    ├── Route 'shop'    →  Shop
                    └── Route 'auth'    →  Authentication    ← NEW route
                                              ├── SignInForm  ← NEW component
                                              └── SignUpForm  ← from lesson 8
```

---

## 🔑 Key Concepts

### 1. `/auth` Route — Replacing `/sign-in`

```jsx
// App.js — route renamed and target changed
<Route path='auth' element={<Authentication />} />

// Navigation — link updated
<Link to='/auth'>SIGN IN</Link>
```

`/sign-in` was a poor name — the page now handles both sign-in AND sign-up. `/auth` is more accurate and professional. The route renders the new `Authentication` component which composes both forms side by side.

---

### 2. Authentication — Layout Route Component

```jsx
// authentication.component.jsx
const Authentication = () => (
  <div className='authentication-container'>
    <SignInForm />
    <SignUpForm />
  </div>
);
```

```scss
// authentication.styles.scss
.authentication-container {
  display: flex;
  width: 900px;
  justify-content: space-between;
  margin: 30px auto;
}
```

Authentication's job is **purely layout** — it:
- Maps to the `/auth` URL (lives in `routes/`)
- Composes `SignInForm` and `SignUpForm` side by side
- Contains zero state, zero Firebase calls, zero form logic

> 🔥 **This is the Container/Presentational pattern** — Authentication is the container (layout, composition); SignInForm/SignUpForm are the presentational components (UI + local logic).

---

### 3. SignInForm — Extracted into components/

```jsx
const defaultFormFields = { email: '', password: '' };

const SignInForm = () => {
  const [formFields, setFormFields] = useState(defaultFormFields);
  const { email, password } = formFields;

  const signInWithGoogle = async () => {
    const { user } = await signInWithGooglePopup();
    await createUserDocumentFromAuth(user);
  };

  const handleSubmit = async (event) => {
    event.preventDefault();
    try {
      const response = await signInAuthUserWithEmailAndPassword(email, password);
      resetFormFields();
    } catch (error) {
      switch (error.code) {
        case 'auth/wrong-password':
          alert('incorrect password for email');
          break;
        case 'auth/user-not-found':
          alert('no user associated with this email');
          break;
        default:
          console.log(error);
      }
    }
  };
  // ...
```

**Why `components/` not `routes/`?**

| | `routes/` | `components/` |
|--|-----------|---------------|
| Tied to a URL | ✅ Yes | ❌ No |
| Reusable anywhere | ❌ No | ✅ Yes |
| Example | Authentication | SignInForm |

`SignInForm` can now be used in a modal, checkout page, or any other context — without being tied to `/auth`.

---

### 4. Two Buttons — type='submit' vs type='button'

```jsx
<div className='buttons-container'>
  <Button type='submit'>Sign In</Button>
  <Button type='button' buttonType='google' onClick={signInWithGoogle}>
    Google sign in
  </Button>
</div>
```

| Button type | Behaviour |
|-------------|-----------|
| `type='submit'` | Dispatches `submit` event on the form → triggers `onSubmit` / `handleSubmit` |
| `type='button'` | Does nothing by default — only fires its own `onClick` |
| `type='reset'` | Clears all form inputs to default values |

> ⚠️ **`type='button'` is critical here.** Without it, clicking "Google sign in" would ALSO trigger `handleSubmit` — which calls `signInAuthUserWithEmailAndPassword` with empty fields, causing a Firebase error.

> 🔥 **FAANG trap:** The default `<button>` type is `type='submit'` — this surprises developers and causes accidental form submissions. Always set `type='button'` explicitly on non-submit buttons inside forms.

---

### 5. switch Statement — Clean Error Handling

```js
catch (error) {
  switch (error.code) {
    case 'auth/wrong-password':
      alert('incorrect password for email');
      break;
    case 'auth/user-not-found':
      alert('no user associated with this email');
      break;
    default:
      console.log(error);
  }
}
```

**switch vs if/else — when to use each:**

| | `switch` | `if/else if` |
|--|---------|-------------|
| Best for | Exact value matching on single expression | Range checks, complex conditions |
| Readability | ✅ Cleaner for many cases | ❌ Repetitive for many cases |
| Equality | Strict (`===`) | Any expression |
| Fall-through | ✅ Possible (omit `break`) | ❌ Not applicable |

**`break` is critical** — without it, JavaScript falls through to the next case and executes it too.

**Intentional fall-through** — multiple cases sharing one handler:
```js
case 'auth/wrong-password':
case 'auth/invalid-password':
  alert('incorrect password'); // handles both cases
  break;
```

**Common Firebase error codes:**

| Error code | Meaning |
|-----------|---------|
| `auth/wrong-password` | Correct email, wrong password |
| `auth/user-not-found` | No account with this email |
| `auth/email-already-in-use` | Sign-up with existing email |
| `auth/too-many-requests` | Account temporarily locked |
| `auth/network-request-failed` | No internet connection |

> 🔥 **Always use `error.code` not `error.message` for programmatic handling** — `error.message` is a human-readable string that can change; `error.code` is a stable contract.

---

### 6. New Firebase Function — Email Sign-In

```js
// New import
import { signInWithEmailAndPassword } from 'firebase/auth';

// New wrapper — same Facade pattern
export const signInAuthUserWithEmailAndPassword = async (email, password) => {
  if (!email || !password) return;
  return await signInWithEmailAndPassword(auth, email, password);
};
```

**What `signInWithEmailAndPassword` does:**
1. Sends credentials to Firebase Auth servers
2. Firebase verifies password hash against stored hash
3. **Success** → returns `UserCredential { user: { uid, email, displayName, ... } }` + sets `auth.currentUser`
4. **Failure** → throws error with specific `error.code`

> 🔥 **Sign-in does NOT write to Firestore** — the user document was created at sign-up. Sign-in only authenticates. Passwords are never stored in Firestore — Firebase Auth handles all hashing securely on Google's servers.

---

## 🔄 What Changed from Lesson 8 → Lesson 9

| Area | Lesson 8 | Lesson 9 |
|------|----------|----------|
| Route | `/sign-in` | `/auth` |
| Page component | `SignIn` (mixed concerns) | `Authentication` (layout only) |
| Sign-in UI | Inside `SignIn` route | `SignInForm` in `components/` |
| Page layout | Sign-in only | Sign-in + Sign-up side by side |
| Email sign-in | ❌ Missing | ✅ `signInAuthUserWithEmailAndPassword` |
| Error handling | `if/else` | `switch(error.code)` |
| Google sign-in | In route | In `SignInForm` component |

---

## ✅ Lesson 9 Checklist

- [ ] `yarn start` → navigate to `/auth` → see both forms side by side
- [ ] Sign up with email → confirm in Firebase Console Auth tab
- [ ] Sign in with same email/password → confirm `response` logged in console
- [ ] Enter wrong password → confirm "incorrect password" alert
- [ ] Enter unregistered email → confirm "no user associated" alert
- [ ] Click Google sign in button → confirm Google popup opens (not form submit)
- [ ] Explain why Google button needs `type='button'`
- [ ] Explain why SignInForm lives in `components/` not `routes/`

---

## 💡 Why Extract SignInForm into components/?

**Problem with keeping it in the route:**
- Sign-in logic was tied to a URL — unusable elsewhere without importing a route component
- The route had too many responsibilities: layout + state + validation + Firebase calls
- To reuse sign-in in a checkout modal or promotional popup → you'd duplicate the code

**After extraction:**
- `SignInForm` is URL-agnostic — import it anywhere
- Independently testable — pass mock Firebase functions as props
- `Authentication` route stays thin — just layout
- Future `/checkout` page can embed `<SignInForm />` with zero duplication

> 🔥 **The rule:** Routes answer *"what page is this?"* — Components answer *"what does this UI do?"* — Never mix them.

---

## 🧠 FAANG Interview Q&A — Lesson 9

### Q1. Difference between `type='submit'` and `type='button'`?

**Answer:** `type='submit'` is the **default** button type inside a `<form>` — clicking it dispatches a `submit` event on the form, triggering the `onSubmit` handler. `type='button'` tells the browser to do nothing by default — only fire the element's own `onClick`.

Without `type='button'` on the Google sign-in button, clicking it would ALSO trigger `handleSubmit` — calling `signInAuthUserWithEmailAndPassword` with empty fields and causing a Firebase error.

> 🔥 **Trap:** The default `<button>` type is `type='submit'` — always set `type='button'` explicitly on non-submit buttons inside forms.
> 🔥 **Third type:** `type='reset'` — clears all form inputs to their default values.

---

### Q2. Why `switch(error.code)` instead of `if/else if`?

**Answer:** `switch` is the right tool when branching on the **exact value** of a single expression. It reads like a lookup table — each `case` maps a specific Firebase error code to its handler. Chained `if/else if (error.code === '...')` is repetitive and harder to scan. `break` after each case prevents fall-through. `default` catches any unhandled error code.

**Intentional fall-through** — multiple cases sharing one handler (omit `break`):
```js
case 'auth/wrong-password':
case 'auth/invalid-password':
  alert('incorrect password');
  break;
```

> 🔥 **Always use `error.code` not `error.message`** — code is a stable contract; message is a human string that can change between Firebase versions.

---

### Q3. Role of Authentication component and why it lives in routes/?

**Answer:** `Authentication` maps 1:1 to the `/auth` URL and is rendered directly by React Router — that's why it lives in `routes/`. Its job is purely layout: composing `SignInForm` and `SignUpForm` side by side with flexbox. It has zero state, zero Firebase calls, zero form logic. `SignInForm` and `SignUpForm` live in `components/` because they're reusable UI units with no URL awareness — they could be embedded in a modal, sidebar, or any page.

> 🔥 **Container/Presentational pattern:** Authentication = container (layout, composition). SignInForm/SignUpForm = presentational (UI + local logic).

---

### Q4. What does `signInWithEmailAndPassword` return on success vs failure?

**Answer:** On success it returns a `UserCredential` — `{ user: { uid, email, displayName, ... } }` — and sets `auth.currentUser` so the app knows who is logged in. On failure it throws an error with a specific `error.code`: `'auth/wrong-password'` for incorrect password, `'auth/user-not-found'` for no account with that email, `'auth/too-many-requests'` if temporarily locked.

Sign-in does **not** write to Firestore — the user document was created at sign-up. Passwords are never stored in Firestore — Firebase Auth handles all hashing securely on Google's servers.

---

### Q5. ⭐ Hard: What architectural problem did extracting SignInForm solve?

**Answer:** The core problem was **mixing concerns** — a route should only handle page composition and URL structure, not form state, validation, and Firebase calls. Keeping sign-in logic in `SignIn` route meant: (1) logic was tied to a URL — unusable elsewhere without importing a route component, violating the routes/components contract; (2) the route had too many responsibilities; (3) adding a sign-in prompt to `/checkout` would require duplicating the code.

Extracting to `components/` makes the logic URL-agnostic, independently testable, and reusable anywhere. `Authentication` becomes thin — just layout. This is exactly how FAANG codebases are structured.

> 🔥 **The rule:** Routes answer *"what page is this?"* — Components answer *"what does this UI do?"* — Never mix them.

---

## 📊 Quiz Results

| # | Question | Score | Gap to Fix |
|---|----------|-------|------------|
| Q1 | type='submit' vs type='button' | ✅ Correct | Bonus: default button type is `submit` — always explicit |
| Q2 | switch vs if/else for error codes | ✅ Correct | Bonus: intentional fall-through by omitting `break` |
| Q3 | Authentication role — routes/ vs components/ | ✅ Correct | — |
| Q4 | signInWithEmailAndPassword — success vs failure | ✅ Correct | Bonus: sign-in never writes to Firestore |
| Q5 | Architectural problem solved by extracting SignInForm | ✅ Correct | — |

**Total: 5/5** — Perfect score. Five perfect scores in a row. Architecture and Firebase auth patterns fully understood.
