# 📘 CRWN Clothing v2 — Lesson 7 Notes

---

# 🔹 Lesson 7 — Firestore Database Integration

**Tag:** Database | **Branch:** `lesson-7`

---

## 📌 What This Lesson Covers

Lesson 7 completes the authentication flow by replacing the `console.log` stub from Lesson 6 with **real Firestore database reads and writes**. The UI is identical — same routes, same navbar, same sign-in button. The entire lesson lives in `firebase.utils.js`. After this lesson, signing in with Google creates a permanent user record in Firestore that persists across sessions.

---

## 📁 What Changed in the Folder Structure

```
src/
  utils/
    firebase/
      firebase.utils.js    ← UPDATED — stub replaced with real Firestore logic
  routes/
    sign-in/
      sign-in.component.jsx ← UPDATED — now destructures user, awaits DB call
```

Everything else is **identical to Lesson 6**.

---

## 📦 New Firestore Imports

```js
import { getFirestore, doc, getDoc, setDoc } from 'firebase/firestore';
```

| Import | Purpose |
|--------|---------|
| `getFirestore` | Gets the Firestore database instance (like `getAuth` for auth) |
| `doc` | Creates a **reference** — a pointer to where a document lives |
| `getDoc` | **Reads** a document from Firestore (one-time) |
| `setDoc` | **Writes** a document to Firestore (overwrites entirely) |

---

## 🗄️ Firestore Data Model

Firestore is a **NoSQL document database** — not tables and rows like MySQL.

```
Firestore
  └── users/                         ← Collection (like a folder)
        ├── uid_abc123/              ← Document (like a JSON file)
        │     ├── displayName: "John Doe"
        │     ├── email: "john@gmail.com"
        │     └── createdAt: Timestamp
        └── uid_xyz789/              ← Another Document
              ├── displayName: "Jane Smith"
              ├── email: "jane@gmail.com"
              └── createdAt: Timestamp
```

**Firestore vs SQL side by side:**

| MySQL (relational) | Firestore (document) |
|--------------------|----------------------|
| Tables | Collections |
| Rows | Documents |
| Columns (fixed schema) | Fields (flexible, per document) |
| JOIN | Sub-collections or denormalised data |
| Foreign keys | Document references |
| Fixed schema | No schema — each document can differ |

> 🔥 **When to choose SQL over Firestore:** SQL for complex multi-table joins, strict ACID transactions, fixed schemas. Firestore for real-time listeners, horizontal scaling, flexible document shapes.

---

## 🔑 Key Concepts

### 1. Database Instance

```js
export const db = getFirestore();
```

- Creates a connection to Firestore — the NoSQL database
- `db` is a handle to the entire database — like `auth` is a handle to Firebase Auth
- Exported so other files can use it directly for reads/writes if needed
- Only one Firestore instance exists per Firebase app — `getFirestore()` always returns the same one

---

### 2. `doc()` — Creating a Reference (No Network Call!)

```js
const userDocRef = doc(db, 'users', userAuth.uid);
```

- Creates a `DocumentReference` — a **lightweight pointer** describing the path to a document
- Arguments: `doc(database, collectionName, documentId)`
- Path: `database/users/abc123uid`
- **No network call** — like writing a file path on paper without opening the file
- The reference can point to a document that **doesn't exist yet** — `doc()` doesn't care

> 🔥 **This separation is intentional** — build a reference once, reuse it for multiple operations (read, write, listen, delete).

---

### 3. `getDoc()` — Reading from Firestore

```js
const userSnapshot = await getDoc(userDocRef);
```

- **NOW** a network call happens — reads the document from Firestore
- Returns a **`DocumentSnapshot`** — NOT a plain JS object

```js
// DocumentSnapshot API — three things you need:
userSnapshot.exists()      // → boolean: does the document exist?
userSnapshot.data()        // → plain JS object: { displayName, email, createdAt }
userSnapshot.id            // → string: the document ID (e.g. 'abc123uid')
```

> ⚠️ **Common bug:** `userSnapshot.displayName` → `undefined`. You MUST call `.data()` first:
> ```js
> userSnapshot.data().displayName  // ✅ correct
> userSnapshot.displayName         // ❌ always undefined
> ```

---

### 4. The `exists()` Check — Idempotency

```js
if (!userSnapshot.exists()) {
  // only runs on first sign-in
  await setDoc(userDocRef, { displayName, email, createdAt });
}
```

Without this check, every sign-in would call `setDoc()` — completely **overwriting** the user document with just 3 fields, destroying:
- Cart items
- Order history
- Saved addresses
- Preferences

The `exists()` check makes the function **idempotent** — can be called many times, only creates data once.

> 🔥 **Cleaner alternative:** `setDoc(ref, data, { merge: true })` — only writes fields that don't exist, preserves everything else. No manual `exists()` check needed for simple cases.

---

### 5. `setDoc()` — Writing to Firestore

```js
try {
  await setDoc(userDocRef, {
    displayName,
    email,
    createdAt,
  });
} catch (error) {
  console.log('error creating the user', error.message);
}
```

- **Completely overwrites** the document with the provided data
- If document doesn't exist → creates it
- If document exists → **replaces it entirely** (fields not in the object are deleted)
- `try/catch` handles network failures gracefully — production apps must never crash silently

**getDoc vs setDoc vs updateDoc:**

| Function | Operation | Document must exist? | Existing fields |
|----------|-----------|---------------------|-----------------|
| `getDoc(ref)` | Read once | N/A | N/A |
| `setDoc(ref, data)` | Write/overwrite | No — creates if missing | ❌ Deleted if not in new data |
| `setDoc(ref, data, { merge: true })` | Merge write | No — creates if missing | ✅ Preserved |
| `updateDoc(ref, partialData)` | Partial update | ✅ Yes — throws if missing | ✅ Preserved |

> 🔥 **Trick:** `updateDoc` on a non-existent document throws an error. `setDoc` with `{ merge: true }` is safer when unsure if the document exists.

---

### 6. The Complete `createUserDocumentFromAuth` Function

```js
export const createUserDocumentFromAuth = async (userAuth) => {
  // 1. Create reference — no network call
  const userDocRef = doc(db, 'users', userAuth.uid);

  // 2. Read from Firestore — network call
  const userSnapshot = await getDoc(userDocRef);

  // 3. Only write if user doesn't exist yet
  if (!userSnapshot.exists()) {
    const { displayName, email } = userAuth;  // pull fields from Firebase Auth user
    const createdAt = new Date();              // timestamp for when account was created

    try {
      // 4. Write to Firestore
      await setDoc(userDocRef, {
        displayName,
        email,
        createdAt,
      });
    } catch (error) {
      console.log('error creating the user', error.message);
    }
  }

  // 5. Return reference (not data) — caller can use for further reads/listens
  return userDocRef;
};
```

---

### 7. Updated SignIn Component

```js
// Lesson 6 — passed entire UserCredential, didn't await DB
const response = await signInWithGooglePopup();
createUserProfileDocument(response);

// Lesson 7 — destructures user, properly awaits DB operation
const { user } = await signInWithGooglePopup();
const userDocRef = await createUserDocumentFromAuth(user);
```

Two key improvements:
- **Destructures `{ user }`** from `UserCredential` — only passes what's needed to the DB function
- **`await`s** the Firestore function — because it now does real async database work

---

## 🔄 Complete Flow — Lesson 7

```
User clicks "Sign in with Google"
        ↓
signInWithGooglePopup()
        ↓
Google OAuth popup → user selects account
        ↓
Firebase returns UserCredential
  { user: { uid, email, displayName, photoURL } }
        ↓
const { user } = UserCredential   ← destructure
        ↓
createUserDocumentFromAuth(user)
        ↓
doc(db, 'users', uid)             ← create reference (no network)
        ↓
getDoc(ref)                       ← read from Firestore (network call)
        ↓
    exists?  ──YES──→  return ref  (user already in DB, skip write)
      │
      NO
      ↓
setDoc(ref, { displayName, email, createdAt })  ← write to Firestore
      ↓
return ref
```

---

## 🔄 What Changed from Lesson 6 → Lesson 7

| Area | Lesson 6 | Lesson 7 |
|------|----------|----------|
| After sign-in | `console.log(userAuth)` stub | Real Firestore read + write |
| Firestore imports | None | `getFirestore, doc, getDoc, setDoc` |
| DB instance | None | `export const db = getFirestore()` |
| User creation | Not implemented | `createUserDocumentFromAuth()` |
| exists() check | None | ✅ Prevents overwriting existing users |
| SignIn component | Passes full response | Destructures `{ user }`, awaits DB |
| Data persistence | None | User saved to Firestore permanently |

---

## ✅ Lesson 7 Checklist

- [ ] `yarn start` → sign in with Google
- [ ] Open Firebase Console → Firestore → users collection → confirm your user document was created
- [ ] Sign in again → confirm document was NOT overwritten (check `createdAt` stays the same)
- [ ] Understand: `doc()` = reference, `getDoc()` = read, `setDoc()` = write
- [ ] Understand: `snapshot.data()` not `snapshot.fieldName`
- [ ] Explain `exists()` check purpose out loud

---

## 💡 Why Return the Reference, Not the Data?

```js
return userDocRef;  // returns DocumentReference
// not:
return userSnapshot.data();  // would return plain object
```

Returning the **reference** is more powerful — the caller can:
- Read the document: `getDoc(userDocRef)`
- Listen for real-time updates: `onSnapshot(userDocRef, callback)`
- Update it: `updateDoc(userDocRef, { cart: [...] })`
- Delete it: `deleteDoc(userDocRef)`

Returning data would be a dead end — a frozen snapshot with no way to interact with Firestore further.

---

## 🧠 FAANG Interview Q&A — Lesson 7

### Q1. What does `doc()` do and does it make a network call?

**Answer:** `doc(db, 'users', userAuth.uid)` creates a `DocumentReference` — a lightweight pointer object describing the path to a document in Firestore. It's like writing a file path on paper — you haven't opened the file yet. **No network call happens.** The network call only happens when you pass this reference to `getDoc()`, `setDoc()`, `updateDoc()`, or `onSnapshot()`.

> 🔥 **Bonus:** References can point to documents that don't exist yet — `doc()` doesn't care. This is exactly what happens the first time a new user signs in.

---

### Q2. Why check `userSnapshot.exists()` before `setDoc()`?

**Answer:** Without the check, every sign-in would call `setDoc()` — completely overwriting the user document with just 3 fields, destroying cart items, order history, preferences and any other data saved after account creation. The `exists()` check makes the function **idempotent** — it can run many times but only creates data once.

> 🔥 **Cleaner alternative:** `setDoc(ref, data, { merge: true })` — only writes fields that don't exist, preserves everything else. No manual exists() check needed.
> 🔥 **FAANG term:** This is the **check-then-act** pattern, also called **idempotency** in distributed systems.

---

### Q3. Difference between `getDoc`, `setDoc`, and `updateDoc`?

**Answer:**
- `getDoc(ref)` — one-time read, returns a `DocumentSnapshot`
- `setDoc(ref, data)` — complete overwrite; creates document if it doesn't exist; deletes any fields not in the new data
- `updateDoc(ref, partialData)` — partial merge; only changes specified fields; **throws if document doesn't exist**

Use `getDoc` to read. Use `setDoc` for creating or full replacement. Use `updateDoc` for partial updates like changing a username. Use `setDoc` with `{ merge: true }` when unsure if the document exists.

---

### Q4. ⭐ What does `getDoc()` return?

**Answer:** `getDoc()` returns a Promise that resolves to a **`DocumentSnapshot`** — NOT a plain JS object. The snapshot has:

```js
snapshot.exists()   // boolean — does the document exist?
snapshot.data()     // plain JS object — the actual fields (undefined if doesn't exist)
snapshot.id         // string — the document's ID
```

> ⚠️ **Common mistake:** `snapshot.displayName` → always `undefined`. You MUST call `.data()` first: `snapshot.data().displayName`.

The reason it's a snapshot and not raw data: Firestore needs to communicate **metadata** (does it exist? what's the ID?) separately from the data itself.

---

### Q5. ⭐ Hard: Firestore data model vs SQL

**Answer:** Firestore is a NoSQL document database. Data is organised in **Collections** (like folders) containing **Documents** (like JSON files), each containing **fields**. Documents can also have **sub-collections** for hierarchical data. No fixed schema — each document can have completely different fields.

MySQL is relational — fixed schema tables, rows match column definitions, JOIN to relate data across tables. Firestore avoids JOINs by **denormalising** — storing related data together in the same document or sub-collection.

```
MySQL              →   Firestore
Tables             →   Collections
Rows               →   Documents
Columns            →   Fields
JOIN               →   Sub-collections / denormalised data
Foreign keys       →   DocumentReferences
```

> 🔥 **Choose SQL when:** complex joins, strict ACID transactions, fixed schemas needed.
> 🔥 **Choose Firestore when:** real-time listeners, horizontal scaling, flexible document shapes.

---

## 📊 Quiz Results

| # | Question | Score | Gap to Fix |
|---|----------|-------|------------|
| Q1 | `doc()` — reference, no network call | ✅ Correct | — |
| Q2 | `exists()` check — prevent overwrite, idempotency | ✅ Correct | Bonus: know `setDoc` with `{ merge: true }` as alternative |
| Q3 | `getDoc` vs `setDoc` vs `updateDoc` | ✅ Correct | Remember: `updateDoc` throws if document doesn't exist |
| Q4 | What `getDoc()` returns | ❌ Missed | Returns `DocumentSnapshot` not plain object — must call `.data()` to get fields |
| Q5 | Firestore data model vs SQL | ✅ Correct | — |

**Total: 4/5** — Strong. Key gap: **DocumentSnapshot API** — `snapshot.data()` is required, never access fields directly on the snapshot.
