# 📘 CRWN Clothing v2 — Lesson 8 Notes

---

# 🔹 Lesson 8 — useState, Controlled Forms & Email Auth

**Tag:** Hooks + Forms | **Branch:** `lesson-8`

---

## 📌 What This Lesson Covers

Lesson 8 introduces the most important React hook — **`useState`** — through a real-world sign-up form. You learn controlled components, the spread + computed property key pattern for form handling, reusable `FormInput` and `Button` components with the rest props pattern, email/password authentication via Firebase, and why `event.preventDefault()` is essential in React forms.

---

## 📁 New Files in Folder Structure

```
src/
  components/
    button/
      button.component.jsx          ← NEW — reusable button with variants
      button.styles.scss            ← NEW — default, google, inverted styles
    form-input/
      form-input.component.jsx      ← NEW — reusable floating label input
      form-input.styles.scss        ← NEW — animated floating label styles
    sign-up-form/
      sign-up-form.component.jsx    ← NEW — full sign-up form with useState
      sign-up-form.styles.scss      ← NEW
  utils/firebase/
    firebase.utils.js               ← UPDATED — email auth + additionalInformation
  routes/sign-in/
    sign-in.component.jsx           ← UPDATED — includes <SignUpForm />
```

---

## 📦 New Firebase Import

```js
import { createUserWithEmailAndPassword } from 'firebase/auth';
```

Adds email/password account creation alongside existing Google OAuth.

---

## 🔑 Key Concepts

### 1. `useState` — React's Memory Hook

```js
import { useState } from 'react';

const defaultFormFields = {
  displayName: '',
  email: '',
  password: '',
  confirmPassword: '',
};

const [formFields, setFormFields] = useState(defaultFormFields);
```

`useState` lets a component **remember data between re-renders**.

```js
const [value, setValue] = useState(initialValue);
//     ↑         ↑              ↑
//  current    setter        starting
//  state      function       value
```

| | Regular Variable | useState |
|--|-----------------|---------|
| Survives re-renders | ❌ Resets every time | ✅ Persists |
| Triggers re-render on change | ❌ Never | ✅ Always |
| Where stored | Component stack frame | React Fiber node |
| How to update | Direct assignment | Setter function only |

**How React detects changes:** Uses `Object.is()` comparison. If the reference is the same object, React sees no change and skips re-rendering.

> ⚠️ **Never mutate state directly:**
> ```js
> formFields.email = value;      // ❌ same reference → no re-render → UI stuck
> setFormFields({ ...formFields, email: value }); // ✅ new object → re-render
> ```

> 🔥 **FAANG follow-up:** *"What's the difference between useState and a regular variable?"* — A regular variable resets to its initial value every re-render. `useState` persists because React stores it in the **Fiber node** outside the component function.

---

### 2. Controlled Components — React Owns the Form

A **controlled component** means React state is the **single source of truth** for every input value:

```jsx
<input
  value={email}           // ← value comes FROM state (React controls it)
  onChange={handleChange} // ← every keystroke updates state
  name='email'
/>
```

**Uncontrolled** (bad): Input manages its own value in the DOM.
**Controlled** (correct): React state drives the input value — they stay in sync.

---

### 3. handleChange — One Function for All Fields

```js
const handleChange = (event) => {
  const { name, value } = event.target;
  //        ↑      ↑
  //  "email"   "john@..."   (from the input element)

  setFormFields({ ...formFields, [name]: value });
  //              ↑               ↑
  //  copy all fields      override just this one
};
```

**Breaking down `{ ...formFields, [name]: value }`:**

```js
// If name='email' and value='john@gmail.com':
{ ...formFields, [name]: value }
// becomes:
{
  displayName: '',           // copied from formFields
  email: 'john@gmail.com',  // overridden (computed key [name])
  password: '',              // copied from formFields
  confirmPassword: '',       // copied from formFields
}
```

| Part | What it does |
|------|-------------|
| `...formFields` | Shallow copies all existing fields into new object |
| `[name]` | **Computed property key** — uses variable value as key name |
| `value` | The new value to set for that field |

> 🔥 **Why spread?** State must never be mutated directly. `{ ...formFields }` creates a new object reference — React sees a change and re-renders.
> 🔥 **Why `[name]`?** Without computed keys you'd need a switch statement with a case for every field. One `handleChange` handles all 4 inputs because of this pattern.
> ⚠️ **Shallow copy caveat:** `{ ...formFields }` only copies one level deep. Nested objects share the same reference. For deeply nested state, use deep cloning or `useReducer`.

---

### 4. handleSubmit — Form Submission

```js
const handleSubmit = async (event) => {
  event.preventDefault();  // ← CRITICAL — stops page reload

  if (password !== confirmPassword) {
    alert('passwords do not match');
    return;
  }

  try {
    const { user } = await createAuthUserWithEmailAndPassword(email, password);
    await createUserDocumentFromAuth(user, { displayName });
    resetFormFields();
  } catch (error) {
    if (error.code === 'auth/email-already-in-use') {
      alert('Cannot create user, email already in use');
    } else {
      console.log('user creation encountered an error', error);
    }
  }
};
```

**`event.preventDefault()` — why it's essential:**

HTML forms have a default browser behaviour: on submit, they make an HTTP request causing a **full page reload**. Without `preventDefault()`:
1. Page reloads entirely
2. React app re-initialises from scratch
3. All state is lost — form fields, auth state, cart, everything
4. URL may change, breaking routing

> 🔥 **`stopPropagation` vs `preventDefault`:** Common confusion. `preventDefault()` stops the browser's default action. `stopPropagation()` stops the event from bubbling up the DOM tree. They solve different problems.
> 🔥 **Why `onSubmit` on `<form>` not `onClick` on button?** `onSubmit` catches BOTH mouse clicks AND keyboard Enter key — `onClick` only catches mouse clicks.

---

### 5. resetFormFields

```js
const resetFormFields = () => {
  setFormFields(defaultFormFields);
};
```

Called after successful sign-up — resets all inputs back to empty strings. Uses the same `defaultFormFields` object defined outside the component so it's never recreated.

---

### 6. Firebase — Email Auth + additionalInformation

**New Firebase function:**
```js
export const createAuthUserWithEmailAndPassword = async (email, password) => {
  if (!email || !password) return;  // guard clause
  return await createUserWithEmailAndPassword(auth, email, password);
};
```

**Updated `createUserDocumentFromAuth` — now accepts `additionalInformation`:**

```js
// Lesson 7
export const createUserDocumentFromAuth = async (userAuth) => {

// Lesson 8
export const createUserDocumentFromAuth = async (
  userAuth,
  additionalInformation = {}   // ← default empty object = backward compatible
) => {
  // ...
  await setDoc(userDocRef, {
    displayName,
    email,
    createdAt,
    ...additionalInformation,  // ← spread LAST so it wins
  });
```

**Why `additionalInformation` is needed:**

| Sign-in method | `userAuth.displayName` | Solution |
|---------------|----------------------|---------|
| Google OAuth | ✅ From Google profile | Pass nothing — `additionalInformation = {}` |
| Email/Password | ❌ `null` — Firebase doesn't know it | Pass `{ displayName }` from form state |

**Spread order matters:**
```js
{
  displayName,              // null (from userAuth for email signup)
  email,
  createdAt,
  ...additionalInformation, // { displayName: 'John' } — overwrites null above ✅
}
```
`...additionalInformation` comes LAST so it overrides. If it came first, `displayName: null` would win.

**Error handling:**
```js
if (error.code === 'auth/email-already-in-use') {
  alert('Cannot create user, email already in use');
}
```
Firebase returns specific error codes — always check `error.code`, not `error.message`, for programmatic error handling.

---

### 7. FormInput — Rest Props Pattern

```jsx
const FormInput = ({ label, ...otherProps }) => {
  return (
    <div className='group'>
      <input className='form-input' {...otherProps} />
      {label && (
        <label className={`${otherProps.value.length ? 'shrink' : ''} form-input-label`}>
          {label}
        </label>
      )}
    </div>
  );
};
```

**The rest props pattern (`...otherProps`):**
- `label` is destructured separately — used for the floating label UI
- Everything else (`type`, `name`, `value`, `onChange`, `required`) collected into `otherProps`
- `{...otherProps}` spreads all of them onto `<input>` automatically
- One component handles every input type — text, email, password — without listing each prop

**Floating label logic:**
```js
className={`${otherProps.value.length ? 'shrink' : ''} form-input-label`}
```
- If input has a value → adds `shrink` class → label moves up (CSS animation)
- If input is empty → no `shrink` → label sits inside the input like a placeholder

**`{label && <label>}` — conditional rendering:**
- If `label` prop is passed → render the `<label>` element
- If no `label` → render nothing (short-circuit evaluation)

> 🔥 **Warning:** Spreading props onto DOM elements can cause React warnings if custom props leak through. Always destructure component-specific props before spreading `otherProps` onto native elements.

---

### 8. Button — Variant Pattern with Type Map

```jsx
const BUTTON_TYPE_CLASSES = {
  google: 'google-sign-in',
  inverted: 'inverted',
};

const Button = ({ children, buttonType, ...otherProps }) => (
  <button
    className={`button-container ${BUTTON_TYPE_CLASSES[buttonType]}`}
    {...otherProps}
  >
    {children}
  </button>
);
```

| `buttonType` prop | CSS class added | Visual |
|-------------------|----------------|--------|
| `'google'` | `google-sign-in` | Blue Google button |
| `'inverted'` | `inverted` | White button, black border |
| `undefined` (none) | `undefined` → ignored | Default black button |

**`children` prop:** Whatever is placed between `<Button>` and `</Button>` tags becomes `children`:
```jsx
<Button type='submit'>Sign Up</Button>
// children = "Sign Up"
```

---

## 🔄 Complete Sign-Up Flow

```
User types in form
  → each keystroke → handleChange(event)
  → { ...formFields, [name]: value }
  → setFormFields(newObject)
  → React re-renders → input shows new value

User clicks "Sign Up"
  → handleSubmit(event)
  → event.preventDefault()            ← stops page reload
  → password === confirmPassword?      ← client validation
  → createAuthUserWithEmailAndPassword(email, password)
  → Firebase creates Auth user
  → Returns { user } (displayName is null!)
  → createUserDocumentFromAuth(user, { displayName })
  → Writes to Firestore:
     { displayName: 'John', email, createdAt }
                ↑ from form, not null
  → resetFormFields()                  ← clears all inputs
```

---

## 🔄 What Changed from Lesson 7 → Lesson 8

| Area | Lesson 7 | Lesson 8 |
|------|----------|----------|
| Auth methods | Google only | Google + Email/Password |
| State management | None | `useState` for form fields |
| Form handling | None | Controlled components + handleChange |
| `createUserDocumentFromAuth` | No extra params | `additionalInformation = {}` |
| Reusable components | None new | `FormInput`, `Button` |
| Error handling | Basic | Firebase error codes |
| New React concept | — | `useState`, controlled components |

---

## ✅ Lesson 8 Checklist

- [ ] `yarn start` → navigate to `/sign-in` → see sign-up form
- [ ] Type in all fields — confirm inputs update in real time
- [ ] Submit with mismatched passwords → see alert
- [ ] Submit with valid data → check Firebase Console Auth tab → user created
- [ ] Check Firestore → users collection → displayName saved correctly
- [ ] Submit same email again → see "email already in use" alert
- [ ] Explain controlled component pattern out loud
- [ ] Explain `{ ...formFields, [name]: value }` without looking at notes

---

## 💡 Why Controlled Components Over Uncontrolled?

Uncontrolled components (using `ref`) let the DOM manage input state — React can only read the value when needed. Controlled components make React state the single source of truth:

- **Validation on every keystroke** — can show errors as user types
- **Conditional submit** — disable button if form is invalid
- **Reset form** — `setFormFields(defaultFormFields)` clears everything instantly
- **Derived state** — compute character count, password strength etc. from state
- **Testing** — state is in JavaScript, easy to unit test without DOM

> 🔥 **At FAANG:** React Hook Form is preferred in large codebases — it uses uncontrolled inputs under the hood for performance, but gives you a controlled-like API. Understanding controlled components is the prerequisite for understanding why React Hook Form exists.

---

## 🧠 FAANG Interview Q&A — Lesson 8

### Q1. What is the only correct way to update state?

**Answer:** The only correct way is through the **setter function** — `setFormFields()`. React uses `Object.is()` comparison to detect state changes. Calling the setter with a new value schedules a re-render and the component function runs again with the updated state. Direct mutation — `formFields.email = value` — keeps the same object reference, `Object.is()` sees no change, React never re-renders, and the UI is stuck. Direct mutation also breaks time-travel debugging and makes bugs unreproducible.

> 🔥 **Key difference from regular variable:** A regular variable resets every render. `useState` persists because React stores it in the **Fiber node** outside the component function.

---

### Q2. Explain `{ ...formFields, [name]: value }` in detail

**Answer:** `{ ...formFields }` creates a **shallow copy** of the state object — necessary because you must never mutate state directly. `[name]` is a **computed property key** — square brackets tell JavaScript to evaluate `name` as a variable and use its value as the object key. So if `name` is `'email'`, it becomes `{ ...formFields, email: value }`. This pattern lets one `handleChange` function handle all 4 inputs without a switch statement.

> ⚠️ **Shallow copy caveat:** Only copies one level deep. Nested objects inside still share references — use deep cloning or `useReducer` for nested state.

---

### Q3. Why does email sign-up need `additionalInformation` but Google sign-in doesn't?

**Answer:** Google OAuth returns `userAuth.displayName` directly from the user's Google profile — it's never null. Email/password auth creates the account but sets `displayName` to `null` — Firebase has no way to know it. The displayName only exists in React form state. We pass it as `additionalInformation: { displayName }` and spread it LAST in `setDoc` so it overwrites the null from `userAuth`. Spread order is critical — `...additionalInformation` must come after `displayName` to win.

---

### Q4. What problem does the rest props pattern solve in FormInput?

**Answer:** The **rest props pattern** (`...otherProps`) solves the prop forwarding problem. `label` is destructured separately for FormInput's own floating label UI. Everything else — `type`, `name`, `value`, `onChange`, `required`, `minLength`, any future HTML attribute — is collected into `otherProps` and spread onto `<input>`. Without this, you'd have to explicitly list every possible HTML input attribute as a prop, making the component fragile and incomplete. With `{...otherProps}`, FormInput supports every native input attribute automatically.

> ⚠️ **Warning:** Always destructure component-specific props before spreading to avoid forwarding custom props to DOM elements (causes React warnings).

---

### Q5. ⭐ Hard: Why is `event.preventDefault()` essential in React forms?

**Answer:** HTML forms have a default browser behaviour — on submit, they make an HTTP GET or POST request to the `action` URL (or current URL), causing a full page reload. Without `preventDefault()` in a React SPA: (1) the page reloads, (2) the entire React app re-initialises, (3) all state is destroyed — form fields, auth state, cart, everything, (4) the URL may change, breaking React Router. In React, form submission is always handled in JavaScript.

```js
const handleSubmit = async (event) => {
  event.preventDefault(); // ← cancel browser default FIRST, before any async work
  // ... rest of logic
};
```

> 🔥 **`stopPropagation` vs `preventDefault`:** `preventDefault` stops the browser's default action. `stopPropagation` stops the event from bubbling up the DOM. Different problems.
> 🔥 **Use `onSubmit` on `<form>`, not `onClick` on button:** `onSubmit` catches both mouse clicks AND keyboard Enter. `onClick` only catches mouse clicks.

---

## 📊 Quiz Results

| # | Question | Score | Gap to Fix |
|---|----------|-------|------------|
| Q1 | Only correct way to update state | ✅ Correct | Bonus: know React uses `Object.is()` comparison, state lives in Fiber node |
| Q2 | Spread + computed property key `[name]` | ✅ Correct | Remember: shallow copy only — nested objects still shared |
| Q3 | Why `additionalInformation` for email but not Google | ✅ Correct | Bonus: spread order matters — `...additionalInformation` must come LAST |
| Q4 | Rest props pattern in FormInput | ✅ Correct | Bonus: always destructure component props before spreading to DOM elements |
| Q5 | `event.preventDefault()` — why essential | ✅ Correct | Bonus: use `onSubmit` on form not `onClick` on button — catches keyboard Enter too |

**Total: 5/5** — Perfect score. Four perfect scores in a row. useState and controlled components fully understood.
