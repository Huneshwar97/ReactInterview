# 📘 CRWN Clothing v2 — Lesson 17 Notes

---

# 🔹 Lesson 17 — Firebase Firestore, writeBatch & Database Seeding

**Tag:** Firestore | **Branch:** `lesson-17`

---

## 📌 What This Lesson Covers

Lesson 17 introduces Firebase Firestore — the cloud database service. A new utility function `addCollectionAndDocuments` uses `writeBatch` to atomically write the entire shop catalog to Firestore in one network request. `shop-data.json` is converted to `shop-data.js` (a proper ES module). `ProductsProvider` temporarily uses a one-time seed `useEffect` (immediately commented out after use) to populate the database. Key concepts: Firestore data model (collections + documents), `writeBatch` atomicity, the one-time seed pattern, and why module-level constants are stable references in dependency arrays.

---

## 📁 New & Changed Files

```
src/
  utils/firebase/
    firebase.utils.js     ← UPDATED — new Firestore imports + addCollectionAndDocuments
  contexts/
    products.context.jsx  ← UPDATED — imports addCollectionAndDocuments + SHOP_DATA;
                                       commented-out seed useEffect
  shop-data.js            ← RENAMED from shop-data.json → proper JS module with export default
```

Everything else (CartContext, all components, all routes) is **identical to Lesson 16**.

---

## 🔗 Full Component/Provider Tree — Lesson 17

No structural changes from Lesson 16. Same provider stack, same routes, same components.

```
<BrowserRouter>
  <UserProvider>
    <ProductsProvider>       ← temporarily seeded Firestore via commented-out useEffect
      <CartProvider>
        <App />
      </CartProvider>
    </ProductsProvider>
  </UserProvider>
</BrowserRouter>
```

---

## 🔑 Key Concepts

### 1. Firebase Firestore vs Firebase Auth — Two Separate Services

You've used Firebase Auth since Lesson 9. Lesson 17 adds the second Firebase service: **Firestore**, a NoSQL cloud database.

```js
// Auth — user identity (Lessons 9–11)
import { getAuth } from 'firebase/auth';
export const auth = getAuth();

// Firestore — cloud database (Lesson 17+)
import { getFirestore } from 'firebase/firestore';
export const db = getFirestore();
```

| Service | Package | Purpose | First used |
|---------|---------|---------|-----------|
| Firebase Auth | `firebase/auth` | Authentication — sign in, sign out, user state | Lesson 9 |
| Firestore | `firebase/firestore` | Database — store and retrieve documents | Lesson 17 |

Both use the same `firebaseApp` instance initialised with `initializeApp(firebaseConfig)` — they share the same project credentials.

---

### 2. Firestore Data Model — Collections & Documents

Firestore is a **document database**, not SQL. No tables, no rows, no schema enforcement.

```
Firestore
  └── collections/              ← collection (like a table, but schema-free)
        ├── hats                ← document ID (key)
        │     title: "Hats"    ← fields (like columns, but flexible)
        │     items: [...]
        ├── sneakers
        │     title: "Sneakers"
        │     items: [...]
        └── jackets
              title: "Jackets"
              items: [...]
```

**Key terms:**

| Term | Firestore | SQL Equivalent |
|------|-----------|---------------|
| Collection | Group of documents | Table |
| Document | Single record with fields | Row |
| Document ID | Unique key within collection | Primary key |
| Field | Named value on a document | Column |

**Getting references (pointers — not data yet):**
```js
const collectionRef = collection(db, 'collections'); // ref to the collection
const docRef = doc(collectionRef, 'hats');            // ref to a specific document
```
References are just pointers — no data is fetched until you call `getDoc()`, `getDocs()`, or similar read functions.

---

### 3. `addCollectionAndDocuments` — The Seed Utility

```js
export const addCollectionAndDocuments = async (collectionKey, objectsToAdd) => {
  const batch = writeBatch(db);
  const collectionRef = collection(db, collectionKey);

  objectsToAdd.forEach((object) => {
    const docRef = doc(collectionRef, object.title.toLowerCase());
    batch.set(docRef, object);
  });

  await batch.commit();
  console.log('done');
};
```

**Step-by-step breakdown:**

| Step | Code | Network? | What happens |
|------|------|---------|-------------|
| 1 | `writeBatch(db)` | ❌ | Creates a local batch queue in memory |
| 2 | `collection(db, collectionKey)` | ❌ | Gets a reference to the target collection |
| 3 | `doc(collectionRef, object.title.toLowerCase())` | ❌ | Creates a doc reference using title as ID |
| 4 | `batch.set(docRef, object)` | ❌ | Queues the write — adds to local batch |
| 5 | `await batch.commit()` | ✅ | Sends ALL queued writes atomically |

**Document ID derivation:**
```js
{ title: 'Hats',     items: [...] } → doc ID: 'hats'
{ title: 'Sneakers', items: [...] } → doc ID: 'sneakers'
{ title: 'Jackets',  items: [...] } → doc ID: 'jackets'
{ title: 'Womens',   items: [...] } → doc ID: 'womens'
{ title: 'Mens',     items: [...] } → doc ID: 'mens'
```
`object.title.toLowerCase()` → consistent, lowercase, predictable document IDs.

---

### 4. `writeBatch` — Atomicity

**Why batch instead of individual `setDoc` calls?**

```js
// ❌ Individual setDoc — 5 separate network requests, each can fail independently
objectsToAdd.forEach(async (object) => {
  await setDoc(doc(collectionRef, object.title.toLowerCase()), object);
  // If #3 fails: documents 1 and 2 are written, 3-5 are missing → corrupted state
});

// ✅ writeBatch — one atomic network request
const batch = writeBatch(db);
objectsToAdd.forEach((object) => {
  batch.set(doc(collectionRef, object.title.toLowerCase()), object);
});
await batch.commit();
// Either ALL 5 write successfully, or NONE do → no partial states
```

**Atomicity** = all-or-nothing. If the network fails during `batch.commit()`, Firestore rolls back — no documents are written. Your database is never left in a partially-seeded state.

> 🔥 **Senior phrasing:** *"Batched writes are atomic — they behave as a single unit of work. Individual writes are independent — each can succeed or fail without affecting the others. For seeding operations where all-or-nothing correctness matters, always batch."*

**`batch.set()` is synchronous** — it just queues a write locally. This is why `forEach` is fine here. If you were making async calls inside the loop (like individual `setDoc`), you'd need `Promise.all` with `.map()` instead.

---

### 5. `shop-data.js` — From JSON to ES Module

```js
// Before: shop-data.json (static JSON, no exports)
// After: shop-data.js — proper JavaScript module

const SHOP_DATA = [
  { title: 'Hats',     items: [{ id: 1, name: 'Brown Brim', price: 25, imageUrl: '...' }, ...] },
  { title: 'Sneakers', items: [...] },
  { title: 'Jackets',  items: [...] },
  { title: 'Womens',   items: [...] },
  { title: 'Mens',     items: [...] },
];

export default SHOP_DATA;
```

**Why rename to `.js`?**
- `.js` files support `export default` — making this a proper ES module
- Can be imported with `import SHOP_DATA from '../shop-data.js'`
- The data structure mirrors the Firestore shape: each object has `title` (used as doc ID) and `items` array

---

### 6. One-Time Seed Pattern — The Commented-Out `useEffect`

```js
export const ProductsProvider = ({ children }) => {
  const [products, setProducts] = useState([]);

  // useEffect(() => {
  //   addCollectionAndDocuments('collections', SHOP_DATA);
  // }, []);

  const value = { products };
  ...
};
```

**The workflow:**
```
1. Uncomment the useEffect
2. Run the app → useEffect fires once on mount
3. addCollectionAndDocuments sends batch to Firestore
4. console.log('done') confirms success
5. Check Firestore console → data is there
6. Re-comment the useEffect immediately
7. Never uncomment again (unless re-seeding intentionally)
```

**Why `[]` dependency array?**
Run once on mount — you only need to seed once. If left active with `[]`, it still runs every time the app starts in a browser tab, overwriting the entire collection each time.

**Why comment out instead of delete?**
It's a **migration record** — documents what was done and provides a reference if you ever need to re-seed from scratch (e.g., after accidentally deleting Firestore data).

> 🔥 **Senior note:** In production, database seeding belongs in dedicated migration scripts or admin CLI tools — not React component lifecycle hooks. The course uses `useEffect` as a pragmatic learning shortcut that teaches the async pattern without introducing a separate toolchain.

---

### 7. Module-Level Constants in Dependency Arrays

```js
// SHOP_DATA is defined at module scope — outside any component
const SHOP_DATA = [...];
export default SHOP_DATA;

// Inside ProductsProvider:
useEffect(() => {
  addCollectionAndDocuments('collections', SHOP_DATA);
}, []); // ← correct
// }, [SHOP_DATA]); ← behaves identically to [] — but dangerous for different reasons
```

**Why `[SHOP_DATA]` behaves identically to `[]`:**

`SHOP_DATA` is a module-level constant. It's created once when the JavaScript module is first loaded. Its reference never changes between renders — it's always the same array in memory.

```js
// Between every render:
previousSHOP_DATA === currentSHOP_DATA  // true — always same reference
// → React's === comparison: no change detected → effect never re-runs
// → [SHOP_DATA] is functionally identical to []
```

**The deeper danger of the junior's suggestion:**
Even IF `SHOP_DATA` could change (imagine it was state), re-seeding on change would be wrong — it would overwrite Firestore with the local snapshot, destroying any data that exists only in the database (user orders, updated prices, etc.). Seeds are **one-way migrations**, not continuous sync jobs.

**The three things to know about dependency arrays and constants:**

| Value type | Reactive? | Changes between renders? | Safe in deps array? |
|-----------|----------|------------------------|-------------------|
| `useState` value | ✅ Yes | ✅ Can change | ✅ Yes — triggers correctly |
| `useContext` value | ✅ Yes | ✅ Can change | ✅ Yes — triggers correctly |
| Module-level constant | ❌ No | ❌ Never | ⚠️ Harmless but misleading |
| Props | ✅ Yes | ✅ Can change | ✅ Yes — triggers correctly |

---

### 8. `async/await` in Firebase Utilities

```js
export const addCollectionAndDocuments = async (collectionKey, objectsToAdd) => {
  // ...
  await batch.commit();   // ← waits for Firestore to confirm
  console.log('done');    // ← only runs after commit succeeds
};
```

`batch.commit()` returns a Promise — it's an async network operation. `await` pauses execution in the function until the Promise resolves (success) or rejects (error). The `async` keyword on the function signature is required whenever you use `await` inside.

**The forEach + async note:**
```js
// batch.set() is SYNCHRONOUS — just queues locally
// forEach is fine — no Promises involved here
objectsToAdd.forEach((object) => {
  batch.set(docRef, object);  // sync — no await needed
});

// If using individual async setDoc instead — you'd need:
await Promise.all(
  objectsToAdd.map((object) => setDoc(doc(collectionRef, object.title.toLowerCase()), object))
);
```

---

## 🔄 What Changed from Lesson 16 → Lesson 17

| Area | Lesson 16 | Lesson 17 |
|------|-----------|-----------|
| Firebase services used | Auth only | Auth + Firestore |
| Firestore imports | None | `getFirestore`, `collection`, `doc`, `writeBatch`, `setDoc` |
| `firebase.utils.js` | Auth utils only | + `addCollectionAndDocuments` |
| Product data source | `shop-data.json` (static) | `shop-data.js` (ES module, same data) |
| `ProductsProvider` | Empty `products` state | + commented-out seed `useEffect` |
| Database | Auth only in Firebase | + Firestore `collections` collection seeded |

---

## ✅ Checklist

- [ ] Uncomment seed `useEffect` → run app → check Firestore console for 5 documents
- [ ] Re-comment `useEffect` immediately after confirming data exists
- [ ] Explain what `writeBatch` does before `commit()` (local queue — no network)
- [ ] Explain why `writeBatch` is preferred over individual `setDoc` calls (atomicity)
- [ ] Explain why `SHOP_DATA` in `[SHOP_DATA]` dependency behaves like `[]`
- [ ] Explain why the seed effect should never be left active (overwrites on every app start)
- [ ] Trace the document ID derivation: `'Hats'` → `'hats'`

---

## 💡 Why Section — Architectural Reasoning

**Why `writeBatch` instead of `setDoc` in a loop?**
Atomicity — all documents write or none do. Individual `setDoc` calls each make their own network request and can fail independently, leaving the database in a partial state. For a seeding operation, partial state means some categories show in the shop and some don't — a silent data bug that's hard to diagnose.

**Why convert `shop-data.json` to `shop-data.js`?**
JSON files can be imported in React (via Webpack's JSON loader), but `.js` modules are the native format for ES module exports. More importantly, a `.js` file can be evolved — you can add helper functions, transformations, or comments alongside the data. JSON is a transport format; `.js` is a code module.

**Why is the seed effect commented out instead of using a flag or environment variable?**
Pragmatism at the learning stage. In production you'd use a proper migration system (Firestore admin SDK, a CLI script, or a one-time Cloud Function). The comment-out pattern is honest — it shows that this code is a one-shot tool, not ongoing logic. The comment is the documentation.

**Why does module-level `SHOP_DATA` never change its reference?**
JavaScript modules are singletons — they're evaluated once when first imported and cached. `SHOP_DATA` is assigned once at module load time and that assignment never changes. Only `useState`, `useReducer`, and `useContext` values are reactive in React. Everything else is static from React's perspective.

---

## 🧠 FAANG Interview Q&A — Lesson 17

### Q1. What is `writeBatch` and what does `batch.commit()` do?

**Answer:** `writeBatch(db)` creates a local batch object — a queue of write operations held in memory with no network activity. You add operations to it with `batch.set()`, `batch.update()`, or `batch.delete()` — all synchronous, all local. `batch.commit()` sends the entire queue to Firestore in a single atomic network request. Either all operations succeed, or none do.

---

### Q2. How is the Firestore document ID derived for `{ title: 'Hats', items: [...] }`?

**Answer:** `object.title.toLowerCase()` — `'Hats'.toLowerCase()` produces `'hats'`, which becomes the document ID within the `collections` collection. Lowercasing ensures consistent, predictable keys regardless of how the title is capitalised in the source data. The resulting Firestore path is `collections/hats`.

---

### Q3. Why is the seed `useEffect` commented out after the first run?

**Answer:** `useEffect` with `[]` runs once per app load in a browser tab — not truly "once ever." Left active, every time a user opens the app, it would call `addCollectionAndDocuments`, overwriting the entire `collections` collection in Firestore with the local snapshot. Any data that exists only in Firestore (updated prices, new items) would be destroyed. It runs once manually to seed the database, then gets commented out permanently as a migration record.

---

### Q4 ⭐ What is the key advantage of `writeBatch` over individual `setDoc` calls in a loop?

**Answer:** Atomicity. `writeBatch` groups all writes into a single unit of work — either all documents are written successfully, or none are. Individual `setDoc` calls each make independent network requests. If the network drops after 3 of 5 `setDoc` calls complete, the database is left in a partial state: 3 categories exist, 2 don't. With `writeBatch`, Firestore either commits everything or rolls back entirely. No partial states, no silent data corruption.

---

### Q5 ⭐⭐ A junior engineer suggests `[SHOP_DATA]` as the dependency array. What is wrong, and what would actually happen?

**Answer:** Two problems. First, `SHOP_DATA` is a module-level constant — created once when the module loads, its reference never changes between renders. React's `===` comparison always sees the same reference, so `[SHOP_DATA]` behaves identically to `[]` — the effect still only runs on mount. The junior's assumption that it's reactive is wrong; only `useState`, `useReducer`, and `useContext` values are reactive in React.

Second, and more critically: even if `SHOP_DATA` could change, re-running the seed would overwrite Firestore's `collections` with a local snapshot — destroying any data that exists only in the database. Database seeds are one-way migrations, not continuous sync mechanisms. The correct mental model is: seed once, then Firestore is the source of truth.

---

## 📊 Quiz Results

| # | Question | Score | Gap to Fix |
|---|----------|-------|------------|
| Q1 | `writeBatch` purpose + `commit()` | ✅ Correct | — |
| Q2 | Document ID from `title.toLowerCase()` | ✅ Correct | — |
| Q3 | Why seed `useEffect` is commented out | ✅ Correct | — |
| Q4 | Atomicity — `writeBatch` vs individual `setDoc` | ✅ Correct | — |
| Q5 | Module-level constants + deps array + re-seed danger | ✅ Correct | — |

**Lesson 17 Score: 5/5 = 100%** 🎉
**Running Total: 65/85 = 76%**

### 🔁 Weak Areas to Carry Forward (from previous lessons)

| Gap | Correct Mental Model |
|-----|---------------------|
| `useEffect` runs AFTER render | Always after render + paint. Never before. `useLayoutEffect` is the before-paint variant. |
| Multiple `useEffect` setState = multiple renders | Each `setState` inside `useEffect` triggers its own render. Not batched. Two effects = 3 renders total. |
| `useNavigate` vs `<Link>` | `<Link>` doesn't throw on buttons. Use `useNavigate` for imperative control + state cleanup before navigating. |
| JSX cannot render objects | Must destructure to primitives. "Objects are not valid as a React child." Not a hooks issue. |
