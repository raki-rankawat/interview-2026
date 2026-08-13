[← Study Plan](../README.md)

# Module 9 — React Hooks

> **Difficulty:** ⭐⭐⭐⭐☆
> **Interview Frequency:** ⭐⭐⭐⭐⭐
> **Importance for React Developers:** ⭐⭐⭐⭐⭐

Hooks are the single most important React area in interviews. You need to know not just **what each Hook does**, but **when to use it, when not to, and what problem it solves**. The "when not to" half is what separates a mid-level answer from a senior one.

---

# Table of Contents

```text
1.  What Are Hooks?
2.  Rules of Hooks
3.  useState
4.  Functional State Updates
5.  Lazy Initialization
6.  Objects, Arrays & Immutable Updates
7.  useEffect
8.  The Dependency Array
9.  Cleanup Functions
10. Fetching Data & AbortController
11. When NOT to Use useEffect
12. Stale Closures
13. useRef
14. useRef vs useState
15. useMemo
16. useCallback
17. useCallback + React.memo
18. When NOT to Memoize
19. useContext
20. Context vs Redux
21. useReducer
22. useState vs useReducer
23. Custom Hooks
24. React 18 Behaviour You Must Know
25. Common Hook Mistakes
26. The Hook Mental Model
27. Common Interview Questions
28. Senior-Level Summary
29. Practice Questions
```

---

# What Are Hooks?

**Hooks are functions that let function components use React features — state, side effects, context, refs, and more.**

Before Hooks arrived in React 16.8, state and lifecycle belonged to class components.

The old way:

```jsx
class Counter extends React.Component {
    state = {
        count: 0
    };

    render() {
        return (
            <button onClick={() =>
                this.setState({
                    count: this.state.count + 1
                })
            }>
                {this.state.count}
            </button>
        );
    }
}
```

With Hooks:

```jsx
function Counter() {
    const [count, setCount] = useState(0);

    return (
        <button onClick={() => setCount(count + 1)}>
            {count}
        </button>
    );
}
```

## Why were Hooks introduced?

"Classes are verbose" is the weak answer. The real motivations:

1. **Stateful logic could not be reused.** Sharing behaviour between classes meant higher-order components and render props, which produced deeply nested "wrapper hell" trees. Custom Hooks solved this properly.
2. **Related logic was split across lifecycle methods.** A subscription's setup lived in `componentDidMount`, its update in `componentDidUpdate`, and its teardown in `componentWillUnmount` — three places, and unrelated concerns crammed together in each. One `useEffect` holds setup and teardown side by side.
3. **`this` was a persistent source of bugs.** Binding, arrow-function class fields, stale `this.state` — all of [Module 5](../javascript/this-prototypes.md)'s complexity, on every component.

That's the answer to give: **Hooks exist to make stateful logic reusable and to group code by concern rather than by lifecycle timing.**

## Important Interview Point

There is a very common misconception:

> "Hooks are basically lifecycle methods for function components."

**Do not say this in an interview.** It is an outdated model, and `useEffect` in particular is not "`componentDidMount` + `componentDidUpdate` + `componentWillUnmount` merged."

The better framing:

> "Hooks let function components use React's capabilities. `useEffect` specifically synchronizes a component with something outside React — it isn't a lifecycle callback, it's a way to keep an external system in sync with React state."

That distinction alone will make your Hook answers noticeably stronger.

---

# Rules of Hooks

Two rules. Both matter, and the "why" behind the first is a real interview question.

## Rule 1 — Only call Hooks at the top level

Never call a Hook inside `if`, `for`, `while`, a nested function, or a callback.

```jsx
function Component({ isLoggedIn }) {

    if (isLoggedIn) {
        const [user, setUser] = useState(null);   // ❌
    }

    return <div />;
}
```

Correct — the Hook runs unconditionally, and the *rendering* is what branches:

```jsx
function Component({ isLoggedIn }) {
    const [user, setUser] = useState(null);

    if (!isLoggedIn) {
        return <Login />;
    }

    return <Dashboard user={user} />;
}
```

Note the ordering: every Hook call comes **before** any early return. An early return above a Hook is the same bug in a different shape.

### Why?

React does not know your Hooks by name. It stores them per component instance as an ordered list, and matches them up **by call order**:

```text
First render                Second render
-----------------------     -----------------------
1. useState(null)     →     1. useState(null)      ✅ same slot
2. useState(0)        →     2. useState(0)         ✅ same slot
3. useEffect(...)     →     3. useEffect(...)      ✅ same slot
```

Put a Hook behind a condition and the list shifts:

```text
First render (logged in)    Second render (logged out)
-----------------------     --------------------------
1. useState(user)     →     1. useState(0)   ← wrong slot: the
2. useState(0)        →     2. useEffect(...)   count state is now
3. useEffect(...)     →     (missing)           read as the user state
```

React would hand slot 1's stored value to a different Hook. That is why the rule is absolute, and why React throws "Rendered fewer hooks than expected" when it detects the mismatch.

## Rule 2 — Only call Hooks from React functions

Hooks may be called from **function components** and **custom Hooks**. Nowhere else.

```javascript
function calculateSomething() {
    const [value, setValue] = useState(0);   // ❌ not a component or Hook
}
```

Make it a custom Hook by naming it with the `use` prefix:

```javascript
function useSomething() {
    const [value, setValue] = useState(0);

    return { value, setValue };
}
```

The `use` prefix is a convention, but it is a load-bearing one: the `eslint-plugin-react-hooks` linter uses it to decide which functions are allowed to call Hooks and which functions to check dependencies on. Install that plugin — it catches most of the mistakes in this module before you run the app.

---

# `useState`

The Hook you will use most.

```jsx
const [count, setCount] = useState(0);
```

```text
count            setCount
   ↓                ↓
current value    updater — schedules a re-render
```

The argument is the **initial** value, used only on the first render. On every later render, `useState` ignores it and returns the stored value.

```jsx
function Counter() {
    const [count, setCount] = useState(0);

    return (
        <div>
            <p>Count: {count}</p>

            <button onClick={() => setCount(count + 1)}>
                Increment
            </button>
        </div>
    );
}
```

What happens on click:

```text
setCount(1)
      ↓
React marks the component as needing an update
      ↓
Component function runs again
      ↓
useState returns 1
      ↓
React updates the DOM
```

## Important Interview Point — the setter does not assign

`count` is a `const` belonging to **this render**. `setCount` does not change it:

```jsx
function handleClick() {
    console.log(count);   // 0
    setCount(5);
    console.log(count);   // still 0 — not 5
}
```

Output:

```text
0
0
```

The new value only exists in the *next* render. Each render is a separate call of your function with its own `count` variable. Internalize this and half of React's confusing behaviour stops being confusing.

---

# Functional State Updates

An important interview detail.

Say you want to increment three times:

```jsx
setCount(count + 1);
setCount(count + 1);
setCount(count + 1);
```

You might expect `0 → 1 → 2 → 3`. You get **1**.

All three calls read the same `count` from the same render — `0` — so all three compute `0 + 1`. React batches them and renders once.

Use the **updater function** form, which receives the latest pending value:

```jsx
setCount(prev => prev + 1);
setCount(prev => prev + 1);
setCount(prev => prev + 1);
```

```text
queue: [prev => prev+1, prev => prev+1, prev => prev+1]

0 → 1 → 2 → 3
```

## Rule of thumb

If the next state depends on the previous state, pass a function:

```jsx
setCount(prev => prev + 1);      // ✅
setCount(count + 1);             // ❌ when it depends on the previous value
```

If the next state is independent, the direct form is fine:

```jsx
setName(event.target.value);     // ✅ nothing to derive
```

## Important Interview Point

The updater must be **pure** — compute and return the next state, nothing else. React may call it more than once (it does in Strict Mode development, deliberately), so side effects inside it will happen twice.

---

# Lazy Initialization

When the initial value is expensive to compute:

```jsx
const [data, setData] = useState(expensiveCalculation());   // ❌
```

`expensiveCalculation()` runs on **every render**. React throws the result away after the first, but you paid for it every time.

Pass a function instead:

```jsx
const [data, setData] = useState(() => expensiveCalculation());   // ✅
```

React calls the initializer only on the first render. This is **lazy initialization**.

```jsx
const [form, setForm] = useState(() =>
    JSON.parse(localStorage.getItem("draft")) ?? {}
);
```

Watch the difference carefully — this is the classic mistake:

```jsx
useState(getInitial())     // calls getInitial() on every render, uses the first result
useState(getInitial)       // calls getInitial() once — pass the function, don't invoke it
```

In Strict Mode development React calls the initializer twice, on purpose, to surface impure initializers. Keep it pure.

---

# Objects, Arrays & Immutable Updates

You must never mutate state. Write these patterns until they come out without thinking.

```jsx
user.name = "John";      // ❌ same reference — React sees no change
setUser(user);
```

```jsx
setUser(prev => ({       // ✅ new object
    ...prev,
    name: "John"
}));
```

React compares the previous and next state with `Object.is`. A mutated object is *identical* to itself, so React can skip the re-render entirely. This is [Module 2 — Data Types & Memory](../javascript/data-types.md) and [Module 4 — Objects & Arrays](../javascript/objects-arrays.md) cashing out in React.

## Nested objects

Every level you change must be copied — spread is shallow.

```jsx
setUser(prev => ({
    ...prev,
    address: {
        ...prev.address,
        city: "Milan"
    }
}));
```

Three levels deep:

```jsx
setState(prev => ({
    ...prev,
    user: {
        ...prev.user,
        address: {
            ...prev.user.address,
            city: "Milan"
        }
    }
}));
```

Untouched branches keep their old references — that is **structural sharing**, and it is a feature: `React.memo` children reading those branches skip re-rendering.

## Arrays

Use the non-mutating operations:

```jsx
setItems(prev => [...prev, item]);                        // add to end
setItems(prev => [item, ...prev]);                        // add to front
setItems(prev => prev.filter(i => i.id !== id));          // remove
setItems(prev => prev.map(i =>                            // update one
    i.id === id ? { ...i, done: true } : i
));
setItems(prev => [...prev].sort(byName));                 // sort a copy
```

`push`, `pop`, `splice`, `sort`, and `reverse` all mutate in place — never call them on state.

## Array of objects, nested field

The pattern that trips people up in live coding:

```jsx
setUsers(prev => prev.map(user =>
    user.id === id
        ? { ...user, address: { ...user.address, city: "Milan" } }
        : user
));
```

Read it as: map to a new array, replace only the matching item, and rebuild only the path down to the field you changed.

## Important Interview Point

If you find yourself writing four levels of spread, the real answer is usually **flatten the state shape** — or reach for Immer (built into Redux Toolkit), which lets you write mutating syntax against a draft and produces an immutable result.

Do not "solve" it with `JSON.parse(JSON.stringify(state))`. That drops `undefined`, functions, `Map`, `Set`, and `Date` becomes a string; it also copies untouched branches, destroying structural sharing and defeating every memoized child. `structuredClone` fixes the data-loss problem but not the wasted copying.

> This was called out in your interview feedback — being precise about immutable updates and nested copy patterns. Write these from memory before your next interview.

---

# `useEffect`

The most misunderstood Hook.

## Definition

**`useEffect` synchronizes a component with a system outside React.**

External systems include:

* network requests
* subscriptions and WebSockets
* timers
* browser APIs (`document.title`, `localStorage`, `matchMedia`)
* non-React third-party libraries (a chart, a map)

```jsx
useEffect(() => {
    document.title = `Count: ${count}`;
}, [count]);
```

`document.title` is not React state. Keeping it in step with `count` is synchronization — a legitimate effect.

## When effects run

```text
State / props change
        ↓
     Render          ← pure: calculate the UI, no side effects
        ↓
   Commit to DOM
        ↓
   Browser paints
        ↓
  useEffect runs     ← after the user can already see the frame
```

The mental model that matters:

> **Rendering calculates the UI. Effects synchronize with things outside React.**

`useEffect` runs *after* paint, so it never blocks the browser from showing the frame. Its rare sibling `useLayoutEffect` runs synchronously after DOM mutations but **before** paint — use it only when you must measure the DOM and adjust before the user sees anything (a tooltip that would otherwise visibly jump).

---

# The Dependency Array

Three forms, three behaviours.

## No array — after every render

```jsx
useEffect(() => {
    console.log("Effect");
});
```

```text
Render → Effect
Render → Effect
Render → Effect
```

Rarely what you want, and a fast route to an infinite loop if the effect sets state.

## Empty array — once, on mount

```jsx
useEffect(() => {
    console.log("Effect");
}, []);
```

Runs after the first render only — in production.

**In development with Strict Mode, React deliberately runs setup → cleanup → setup.** This is not a bug and not something to work around; it surfaces effects that aren't safe to re-run. Know this for interviews (see [React 18 Behaviour](#react-18-behaviour-you-must-know) below).

## With dependencies — when a dependency changes

```jsx
useEffect(() => {
    console.log(count);
}, [count]);
```

Runs after mount, then after any render where a dependency changed. React compares each entry with `Object.is` — the same reference check as state.

## Important Interview Point — the dependency array is not a trigger list

The array does not mean "run when I want." It means: *these are all the reactive values from the component this effect reads.* React uses it to decide when to re-synchronize.

That is why **lying about dependencies is a bug, not an optimization**:

```jsx
useEffect(() => {
    fetchUser(userId).then(setUser);
}, []);              // ❌ reads userId but doesn't declare it
```

Change `userId` and the effect never re-runs — the screen shows the wrong user. The lint rule flags exactly this. If a dependency causes a loop, the fix is to change the code (move the value out, use an updater, memoize the object), never to delete the entry.

Note that objects, arrays, and functions declared inside the component are **new references on every render**, so listing one makes the effect run every render:

```jsx
const options = { id };                       // new object each render

useEffect(() => {
    connect(options);
}, [options]);                                // ❌ runs every render
```

Fix by depending on the primitive, or by moving the object inside the effect:

```jsx
useEffect(() => {
    connect({ id });
}, [id]);                                     // ✅
```

Setter functions from `useState` and `dispatch` from `useReducer` have stable identities — React guarantees it, so they never need to be listed.

---

# Cleanup Functions

An effect may return a function. React calls it to tear the effect down.

```jsx
useEffect(() => {
    const interval = setInterval(() => {
        console.log("Running");
    }, 1000);

    return () => {
        clearInterval(interval);
    };
}, []);
```

Cleanup runs:

* **before the effect re-runs**, when a dependency changed, and
* **when the component unmounts**.

Ordering when `roomId` changes from `"a"` to `"b"`:

```text
cleanup("a")  →  setup("b")
```

Always the old teardown first, then the new setup. Never two live subscriptions at once.

Cleanup is essential for:

* timers — `clearInterval`, `clearTimeout`
* event listeners — `removeEventListener`
* subscriptions and WebSockets — `unsubscribe`, `close`
* in-flight requests — `AbortController.abort()`

Skip it and you get memory leaks, duplicate listeners stacking up on every render, and state updates fired at components that no longer exist.

```jsx
useEffect(() => {
    function onResize() {
        setWidth(window.innerWidth);
    }

    window.addEventListener("resize", onResize);

    return () => window.removeEventListener("resize", onResize);
}, []);
```

Note that `onResize` must be the **same reference** in both calls — defining it inside the effect guarantees that. `addEventListener("resize", () => ...)` followed by `removeEventListener("resize", () => ...)` removes nothing, because the two arrow functions are different objects.

---

# Fetching Data & AbortController

A production-shaped pattern:

```jsx
useEffect(() => {
    const controller = new AbortController();

    async function fetchUsers() {
        try {
            setStatus("loading");

            const response = await fetch("/api/users", {
                signal: controller.signal
            });

            if (!response.ok) {
                throw new Error(`HTTP ${response.status}`);
            }

            const data = await response.json();

            setUsers(data);
            setStatus("success");
        } catch (error) {
            if (error.name !== "AbortError") {
                setError(error);
                setStatus("error");
            }
        }
    }

    fetchUsers();

    return () => {
        controller.abort();
    };
}, []);
```

```text
Component mounts
      ↓
request starts
      ↓
component unmounts / deps change
      ↓
cleanup runs
      ↓
controller.abort()
      ↓
request cancelled, catch sees AbortError and ignores it
```

Three details that make this a senior answer:

1. **The `AbortError` check.** Aborting rejects the promise. Without the guard you would set an error state for a cancellation you caused deliberately.
2. **`response.ok`.** `fetch` does not reject on 404 or 500 — see [Module 6](../javascript/async-event-loop.md). Without this check you would `.json()` an error page.
3. **The race condition it fixes.** With `[userId]` as the dependency, switching users quickly can let a slower earlier response land after a faster later one, leaving the wrong data on screen. Aborting the previous request removes the race. (An `ignore` boolean flipped in cleanup solves the same problem without cancelling the network call.)

That is a far stronger answer than "I use `useEffect` to fetch data."

## Important Interview Point

Strict Mode's double-invoke hits this pattern in development: mount → abort → mount again, so you will see a cancelled request in the network tab. That is the intended behaviour proving your cleanup works.

Also worth saying out loud: **in production applications, data fetching usually shouldn't live in `useEffect` at all.** TanStack Query, SWR, or a framework loader give you caching, deduplication, retries, and race handling for free. Being able to write the raw effect *and* knowing you would reach for a library is exactly the balance interviewers look for.

---

# When NOT to Use `useEffect`

A major modern React interview topic — and the source of most bad effect code.

## Don't use an effect for derived values

```jsx
const [fullName, setFullName] = useState("");          // ❌

useEffect(() => {
    setFullName(`${firstName} ${lastName}`);
}, [firstName, lastName]);
```

This costs an extra render, and `fullName` can be momentarily stale. Just derive it:

```jsx
const fullName = `${firstName} ${lastName}`;           // ✅
```

Same for filtering, sorting, and totals — compute during render, and wrap in `useMemo` only if profiling says it is slow.

## Don't use an effect to respond to a user event

```jsx
useEffect(() => {                                       // ❌
    if (submitted) {
        sendAnalytics("form_submitted");
    }
}, [submitted]);
```

Nothing outside React needs synchronizing — a specific thing happened. Put it in the handler:

```jsx
function handleSubmit() {                               // ✅
    sendAnalytics("form_submitted");
    submitForm();
}
```

## Don't use an effect to reset state on a prop change

```jsx
useEffect(() => {                                       // ❌
    setDraft("");
}, [userId]);
```

Give the component a `key` and let React remount it with fresh state:

```jsx
<Editor key={userId} userId={userId} />                 // ✅
```

## The test

> **"Am I synchronizing React with something outside React?"**

**Yes** → an effect may be right.
**No** → derive it during render, or do it in the event handler.

---

# Stale Closures

The bug behind a large share of "my Hook doesn't work" questions, and interview question 16.

Every render creates new functions that close over **that render's** values. If a function outlives the render that created it, it keeps seeing old values. This is [Module 3 — Functions & Closures](../javascript/functions-closures.md) with React's render model on top.

```jsx
function Timer() {
    const [count, setCount] = useState(0);

    useEffect(() => {
        const id = setInterval(() => {
            setCount(count + 1);        // ❌ count is frozen at 0
        }, 1000);

        return () => clearInterval(id);
    }, []);                             // set up once, never re-created

    return <p>{count}</p>;
}
```

Output:

```text
1
1
1
1   ← stuck forever
```

The interval callback was created during the first render, where `count` is `0`. It computes `0 + 1` every second, forever. The empty dependency array is what keeps that first closure alive.

## Three fixes, in order of preference

**1. Remove the dependency — use the updater form.** The callback no longer reads `count` at all:

```jsx
useEffect(() => {
    const id = setInterval(() => {
        setCount(prev => prev + 1);     // ✅
    }, 1000);

    return () => clearInterval(id);
}, []);
```

**2. Declare the dependency honestly.** The effect tears down and re-creates with a fresh closure each time `count` changes — correct, but it restarts the interval every second:

```jsx
useEffect(() => {
    const id = setInterval(() => setCount(count + 1), 1000);
    return () => clearInterval(id);
}, [count]);
```

**3. Keep the latest value in a ref.** For non-reactive values an effect needs to read without re-subscribing:

```jsx
const onTickRef = useRef(onTick);

useEffect(() => {
    onTickRef.current = onTick;         // updated after every render
});

useEffect(() => {
    const id = setInterval(() => onTickRef.current(), 1000);
    return () => clearInterval(id);
}, []);                                 // never re-subscribes
```

## Important Interview Point

Say this explicitly: **a stale closure is not a React bug — it is ordinary JavaScript closure behaviour meeting React's "every render is a snapshot" model.** Naming the cause that precisely reads as real experience.

---

# `useRef`

`useRef` stores a mutable value that **persists across renders without triggering one**.

```jsx
const countRef = useRef(0);

countRef.current;        // read
countRef.current = 5;    // write — no re-render
```

`useRef` returns the *same object* on every render; only `.current` changes.

## Use 1 — DOM access

```jsx
function Input() {
    const inputRef = useRef(null);

    const handleClick = () => {
        inputRef.current.focus();
    };

    return (
        <>
            <input ref={inputRef} />

            <button onClick={handleClick}>
                Focus
            </button>
        </>
    );
}
```

React sets `inputRef.current` when it commits the DOM, so it is `null` during the first render body and available by the time effects and handlers run.

## Use 2 — instance values that shouldn't render

```jsx
const intervalRef = useRef(null);

function start() {
    intervalRef.current = setInterval(tick, 1000);
}

function stop() {
    clearInterval(intervalRef.current);
}
```

Timer ids, previous values, "has this already submitted" flags, WebSocket instances, third-party library handles — none of them belong on screen, so none of them belong in state.

## Important Interview Point

Do not read or write `.current` **during render**. Rendering must be pure, and React makes no guarantee about when a render's work is kept. Confine ref access to event handlers and effects.

In React 19, `ref` is an ordinary prop for function components — `forwardRef` is no longer required to pass one down.

---

# `useRef` vs `useState`

A very common interview question.

| `useState` | `useRef` |
| ---------- | -------- |
| Stores rendered data | Stores a mutable value |
| Updating triggers a re-render | Updating triggers nothing |
| Value is a snapshot of the render | `.current` is always the latest value |
| Updated via the setter, asynchronously | Assigned directly, synchronously |
| For anything the user sees | For DOM nodes and behind-the-scenes values |

```jsx
const [count, setCount] = useState(0);
setCount(1);              // schedules a render

const countRef = useRef(0);
countRef.current++;       // nothing happens on screen
```

## The decision rule

> **Does the UI need to change when this value changes?**
> Yes → state. No → ref.

The classic follow-up: *"What if I use a ref for something displayed?"* The value updates but the screen doesn't, until some other state change happens to trigger a render — and then a stale-looking number suddenly jumps. Being able to describe that failure shows you understand the difference rather than having memorized the table.

---

# `useMemo`

`useMemo` memoizes **the result of a calculation** between renders.

```jsx
const filteredUsers = useMemo(() => {
    return users.filter(user =>
        user.name.includes(search)
    );
}, [users, search]);
```

While `users` and `search` keep the same references, React reuses the previous array instead of recomputing.

## Two legitimate reasons to use it

**1. The calculation is genuinely expensive.** Sorting or filtering thousands of rows, parsing a large payload, heavy derived data. Profile first — "expensive" means milliseconds you can see, not a string concatenation.

**2. You need a stable reference.** Often the more important one:

```jsx
const config = useMemo(() => ({ id, mode }), [id, mode]);

useEffect(() => {
    connect(config);
}, [config]);                       // now only re-runs when id or mode change
```

Without the memo, `config` is a new object every render, so the effect re-runs every render — and any `React.memo` child receiving it re-renders every time too.

## Important Interview Point

`useMemo` is a **performance hint, not a guarantee**. React is allowed to throw a cached value away (it does, for example, to free memory for offscreen content). So your code must still be correct if the calculation runs on every render — never put a side effect inside `useMemo`.

---

# `useCallback`

`useCallback` memoizes **a function reference**.

```jsx
const handleDelete = useCallback((id) => {
    deleteUser(id);
}, [deleteUser]);
```

It is literally `useMemo` for functions:

```jsx
useCallback(fn, deps)  ===  useMemo(() => fn, deps)
```

## The distinction interviewers ask for

```text
useMemo      → memoizes a VALUE  (the result of calling the function)
useCallback  → memoizes a FUNCTION (the function itself, uncalled)
```

## Why it matters

Every render creates a new function object:

```jsx
const handleDelete = () => deleteUser();
```

`handleDelete` from render 1 and render 2 are two different objects, even though the code is identical. Pass it to a memoized child and:

```text
Parent renders
      ↓
new function reference
      ↓
Child's props changed (by reference)
      ↓
React.memo can't bail out — Child re-renders
```

`useCallback` keeps the reference stable so the child's props really are unchanged.

---

# `useCallback` + `React.memo`

These only work as a pair.

```jsx
const Child = React.memo(function Child({ onClick }) {
    console.log("Child rendered");

    return <button onClick={onClick}>Click</button>;
});
```

```jsx
function Parent() {
    const [count, setCount] = useState(0);

    const handleClick = useCallback(() => {
        console.log("clicked");
    }, []);

    return (
        <>
            <button onClick={() => setCount(count + 1)}>{count}</button>
            <Child onClick={handleClick} />
        </>
    );
}
```

Output when clicking the counter, with `useCallback`:

```text
(nothing — Child does not re-render)
```

Without `useCallback`:

```text
Child rendered
Child rendered
Child rendered
```

`React.memo` skips re-rendering when props are shallowly equal. A fresh function reference breaks that equality, which is why the two are almost always discussed together.

## Important Interview Point

**`useCallback` alone does nothing for performance.** If the child is not wrapped in `React.memo`, and the function is not a dependency of another Hook, memoizing it just adds an allocation and a dependency array. Interviewers ask about `useCallback` specifically to find out whether you know that.

---

# When NOT to Memoize

Over-memoization is a classic senior-interview trap.

```jsx
const fullName = useMemo(
    () => `${firstName} ${lastName}`,
    [firstName, lastName]
);
```

A string concatenation is faster than the memo machinery around it. You have paid for:

* extra complexity in every future read of this component
* a dependency array to keep correct
* memory held for the cached value and the deps
* a comparison on every render

for no measurable gain.

The honest position to state in an interview:

> "Memoization is not free. It costs memory and a comparison on every render, and wrong dependencies turn it into a correctness bug. I memoize when profiling shows an actual problem, or when a reference has to stay stable for `React.memo` or a Hook dependency."

## Important Interview Point

React 19's **React Compiler** automatically inserts this memoization at build time, which removes most of the need to write `useMemo` and `useCallback` by hand. Mentioning it shows you follow the ecosystem — but keep the fundamentals, because the compiler is opt-in, most codebases haven't adopted it, and you still need to explain *what* it is doing on your behalf.

---

# `useContext`

`useContext` reads a value from a Context provider, without threading props through every intermediate component.

```jsx
const ThemeContext = createContext(null);
```

Provider:

```jsx
function App() {
    return (
        <ThemeContext.Provider value="dark">
            <Dashboard />
        </ThemeContext.Provider>
    );
}
```

Consumer, at any depth:

```jsx
function Button() {
    const theme = useContext(ThemeContext);

    return (
        <button className={theme}>
            Click
        </button>
    );
}
```

`Dashboard` never sees `theme`. That is the point — Context solves **prop drilling**.

In React 19 you can render the context itself as the provider: `<ThemeContext value="dark">`.

## Important Interview Point — the re-render trap

**Every consumer re-renders when the context value changes** — and "changes" means by reference.

```jsx
<AuthContext.Provider value={{ user, login, logout }}>   // ❌ new object every render
```

That object literal is new on every render of `App`, so every consumer in the tree re-renders every time `App` renders, even if `user` never changed.

```jsx
const value = useMemo(                                    // ✅
    () => ({ user, login, logout }),
    [user, login, logout]
);

<AuthContext.Provider value={value}>
```

The other standard fix is **splitting contexts** — put rarely-changing data and frequently-changing data in separate providers so a fast-changing value doesn't re-render consumers that only need the slow one.

---

# Context vs Redux

Flagged as an improvement area in your interview feedback, so this answer needs to be sharp.

Do **not** say:

> "Context is for small apps, Redux is for large apps."

Size is not the variable, and interviewers hear that answer constantly.

## The framing that lands

**Context is a dependency-injection mechanism, not a state manager.** It transports a value down the tree. It does not store state (you still need `useState` or `useReducer`), does not batch, has no selectors, no middleware, and no devtools. Every consumer re-renders whenever the value's reference changes.

**Redux Toolkit is a state-management library.** Centralized store, reducers describing transitions, selective subscriptions via selectors, middleware, and time-travel devtools.

## Context fits

* theme, locale, feature flags
* the current authenticated user
* configuration and injected services

The pattern: **read often, change rarely, few distinct pieces.**

## Redux Toolkit fits

* complex shared state with many consumers
* frequent updates where re-rendering every consumer would be too expensive
* multi-step state transitions worth expressing as reducers
* a need for middleware, devtools, or an auditable action log

## The question to actually ask

> "What kind of state is this, how often does it change, how complex are the transitions, and how many parts of the app read it?"

## Important Interview Point

The strongest addition — and the most common real-world answer today:

> "A lot of what teams used to put in Redux was server state, not client state. Cached API data with loading, error, refetch and invalidation belongs in TanStack Query. Once server state moves there, the genuinely global client state left over is often small enough that Context plus `useReducer` handles it."

That reframing — **server state vs client state** — is what a senior answer sounds like in 2026.

---

# `useReducer`

Useful when state transitions get complex.

Instead of scattering related updates:

```jsx
setStatus("loading");
setError(null);
setData(null);
```

centralize them.

```jsx
const initialState = {
    status: "idle",
    data: null,
    error: null
};
```

```jsx
function reducer(state, action) {
    switch (action.type) {
        case "FETCH_START":
            return {
                ...state,
                status: "loading",
                error: null
            };

        case "FETCH_SUCCESS":
            return {
                status: "success",
                data: action.payload,
                error: null
            };

        case "FETCH_ERROR":
            return {
                status: "error",
                data: null,
                error: action.payload
            };

        default:
            return state;
    }
}
```

```jsx
function Users() {
    const [state, dispatch] = useReducer(reducer, initialState);

    // ...
}
```

```jsx
dispatch({ type: "FETCH_START" });
```

The gain is that **impossible states become impossible**. With three separate `useState` calls you can end up with `status: "success"` and `error` still set from last time. The reducer makes each transition produce one complete, valid state.

## Rules

* The reducer must be **pure** — no fetches, no mutation, no `Math.random()`. Given the same state and action, always the same result. (Strict Mode double-invokes reducers in development to catch impurity.)
* **`dispatch` has a stable identity.** It never changes between renders, so it never needs to appear in a dependency array — which makes it much easier to pass down than a pile of callbacks.

---

# `useState` vs `useReducer`

```text
useState                       useReducer
────────                       ──────────
one independent value          several related values
simple set                     multiple named transitions
boolean / string / number      state machine-ish shape
next state is obvious          next state depends on the action
```

Reach for `useReducer` when:

* several state values always change together
* the next state depends on the previous state in non-trivial ways
* the same transitions are triggered from many places
* you find yourself calling three setters in a row, repeatedly

A useful signal: if you can name your updates — `submitted`, `retried`, `cancelled` — a reducer is probably the better fit. `useReducer` is also easier to test, because the reducer is a plain function you can call without rendering anything.

---

# Custom Hooks

Custom Hooks let you **reuse stateful logic**.

Say several components need online/offline status.

```jsx
function useOnlineStatus() {
    const [isOnline, setIsOnline] = useState(navigator.onLine);

    useEffect(() => {
        const handleOnline = () => setIsOnline(true);
        const handleOffline = () => setIsOnline(false);

        window.addEventListener("online", handleOnline);
        window.addEventListener("offline", handleOffline);

        return () => {
            window.removeEventListener("online", handleOnline);
            window.removeEventListener("offline", handleOffline);
        };
    }, []);

    return isOnline;
}
```

```jsx
function Checkout() {
    const isOnline = useOnlineStatus();

    if (!isOnline) {
        return <p>No internet connection</p>;
    }

    return <PaymentForm />;
}
```

```jsx
function Header() {
    const isOnline = useOnlineStatus();

    return (
        <header>
            {isOnline ? "Online" : "Offline"}
        </header>
    );
}
```

Different UI, shared logic. The subscription, the cleanup, and the state all live in one place.

## Important Interview Point — Hooks share logic, not state

This is interview question 31, and the answer surprises people:

> **Every component that calls a custom Hook gets its own independent state.**

`Checkout` and `Header` above each hold a separate `isOnline`. A custom Hook is just a function — calling it runs `useState` again, in a different component, producing different state.

To share the *value*, you need Context (or a store). Custom Hooks are a code-reuse mechanism, not a state-sharing one. The common combination is a Context provider paired with a `useAuth()` custom Hook that reads it — the Hook hides the Context, and the Context is what makes the state shared.

## What makes a good custom Hook

* Name starts with `use` — required for the linter, and it signals that Hook rules apply.
* It encapsulates a concern (`useDebounce`, `useLocalStorage`, `useFetch`, `useMediaQuery`), not just a bundle of unrelated `useState` calls.
* It returns the minimum needed — a value, a tuple, or a small object.
* It handles its own cleanup.

For subscribing to genuinely external stores, `useSyncExternalStore` is the purpose-built Hook. It handles server rendering and concurrent tearing correctly — both of which the `useOnlineStatus` version above gets wrong (`navigator` doesn't exist on a server). Knowing when to reach for it is a strong signal.

---

# React 18 Behaviour You Must Know

## Automatic batching

Before React 18, only updates inside React event handlers were batched. Updates in promises, `setTimeout`, and native event handlers each caused their own render.

```jsx
setTimeout(() => {
    setCount(c => c + 1);
    setFlag(f => !f);
}, 0);
```

```text
React 17 → two renders
React 18 → one render (automatic batching)
```

Batching is now automatic everywhere. `flushSync` opts out for the rare case where you need the DOM updated synchronously.

## Strict Mode double-invoking

In **development only**, `<StrictMode>` deliberately:

* renders components twice — to surface impure render logic
* runs state initializers and reducers twice — same reason
* mounts effects, unmounts them, and mounts again — setup → cleanup → setup

```text
Development, Strict Mode:      Production:

setup                          setup
cleanup                        (no cleanup until unmount)
setup
```

None of this happens in production. It exists to catch effects that break when re-run — a subscription without an `unsubscribe`, an event listener never removed, a fetch never aborted. If double-mounting breaks your component, the cleanup is missing.

## Concurrent rendering

React 18 can interrupt, pause, and restart a render. Two consequences worth knowing:

* **Rendering must be pure.** No mutation of props/state, no side effects during render — a render that gets thrown away must leave nothing behind.
* Two Hooks let you mark work as low priority: `useTransition` for updates that may lag (filtering a huge list) and `useDeferredValue` for a value that may trail the input behind it. Both keep typing responsive while expensive rendering catches up.

---

# Common Hook Mistakes

A checklist worth re-reading the night before an interview.

1. **Calling a Hook conditionally** or after an early return — breaks the call-order contract.
2. **Lying about dependencies** to stop a loop — hides the bug instead of fixing it.
3. **Mutating state** and calling the setter with the same reference — no re-render.
4. **`setCount(count + 1)` repeatedly** in one handler — all reads see the same snapshot; use the updater.
5. **Missing cleanup** — leaked timers, stacked listeners, un-aborted requests.
6. **`useEffect` for derived state** — an extra render and a chance to be stale; compute during render.
7. **Setting state in an effect that depends on that same state** — infinite loop.
8. **Object or function literals in dependency arrays** — new reference every render, so the effect never settles.
9. **Memoizing everything** — cost with no benefit, and `useCallback` without `React.memo` does nothing.
10. **Passing a raw object as a Context value** — every consumer re-renders on every provider render.
11. **Index keys in a list of stateful rows** — covered in [Module 8](react-fundamentals.md).
12. **Reading `ref.current` during render** — rendering must be pure.
13. **Treating Strict Mode's double effect as a bug** — it's a diagnostic; fix the cleanup.

---

# The Hook Mental Model

```text
useState
   ↓
Store changing data that the UI displays

useEffect
   ↓
Synchronize with a system outside React

useRef
   ↓
Persist a value without triggering a render

useMemo
   ↓
Memoize a calculated value

useCallback
   ↓
Memoize a function reference

useContext
   ↓
Read a shared value without prop drilling

useReducer
   ↓
Manage complex, related state transitions

Custom Hook
   ↓
Reuse stateful logic (not the state itself)
```

---

# Common Interview Questions

Aim to answer each out loud in 1–3 minutes.

### Fundamentals

* What are React Hooks?
* Why were Hooks introduced?
* What are the Rules of Hooks?
* Why can't Hooks be called conditionally?
* Why must custom Hooks start with `use`?

### `useState`

* How does `useState` work?
* Why does the state variable not update immediately after calling the setter?
* Why use functional state updates?
* What is lazy initialization?
* Why shouldn't state be mutated directly?
* How do you update a nested object in state?

### `useEffect`

* What is `useEffect` actually for?
* What does the dependency array do?
* What happens with no dependency array? With `[]`?
* What is an effect cleanup, and when does it run?
* How would you cancel an API request?
* When should you NOT use `useEffect`?
* What is a stale closure, and how do you fix one?
* What is the difference between `useEffect` and `useLayoutEffect`?

### `useRef`

* `useRef` vs `useState`?
* Why doesn't updating `ref.current` cause a render?
* What are practical uses of `useRef`?
* Why shouldn't you read a ref during render?

### Performance

* `useMemo` vs `useCallback`?
* When should you use each?
* Why shouldn't you memoize everything?
* How do `React.memo` and `useCallback` work together?
* Does `useCallback` help if the child isn't wrapped in `React.memo`?

### State management

* `useState` vs `useReducer`?
* Context vs Redux?
* When would you use Context, and when would Redux be more appropriate?
* Why does every consumer re-render when a context value changes?
* What's the difference between client state and server state?

### Architecture

* What is a custom Hook, and why would you write one?
* Can two components share state through a custom Hook?
* What does Strict Mode's double effect invocation tell you?

---

# Senior-Level Summary

By the end of this module, you should be able to explain:

* Hooks exist to make **stateful logic reusable** and to group code by concern instead of by lifecycle timing — not to replace lifecycle methods.
* Hooks are matched **by call order**, which is why they must be called unconditionally at the top level of a component or custom Hook.
* Each render is a **snapshot**: state variables are constants for that render, and the setter schedules the next one.
* Use the **updater form** whenever the next state depends on the previous, and pass a function to `useState` for expensive initial values.
* State must be **replaced, not mutated**; nested updates copy every level down to the changed field, and structural sharing keeps memoized children cheap.
* `useEffect` **synchronizes with external systems** and runs after paint; the dependency array is a description of what the effect reads, not a trigger list.
* **Cleanup runs before every re-run and on unmount** — required for timers, listeners, subscriptions, and in-flight requests.
* Cancel requests with `AbortController`, ignore `AbortError`, and check `response.ok`; in real apps prefer a query library over hand-rolled effects.
* Don't use effects for **derived values, event responses, or resetting state on a prop change** — derive during render, handle in the handler, or remount with a `key`.
* A **stale closure** is plain JavaScript closure behaviour over a render snapshot; fix it with the updater form, honest dependencies, or a ref.
* `useRef` persists a mutable value **without re-rendering**; state is for what the user sees, refs for everything else.
* `useMemo` memoizes a **value**, `useCallback` a **function**; both are hints, not guarantees, and `useCallback` is pointless without `React.memo` or a Hook dependency consuming it.
* Context is **dependency injection, not state management** — every consumer re-renders on a reference change, so memoize the value and split contexts by update frequency.
* Redux Toolkit earns its place with complex transitions, frequent updates, and selective subscriptions — but most "global state" is **server state** and belongs in a query library.
* `useReducer` centralizes related transitions, must be pure, and gives you a stable `dispatch`.
* Custom Hooks share **logic, not state** — each caller gets its own; Context is what makes state shared.
* React 18 batches updates everywhere, and Strict Mode's double-invoking in development is a **diagnostic for missing cleanup**, not a bug.

---

# Practice Questions

Answer these without running them.

### 1. What is logged, and what is `count` after one click?

```jsx
function Counter() {
    const [count, setCount] = useState(0);

    function handleClick() {
        setCount(count + 1);
        console.log(count);
        setCount(count + 1);
    }

    return <button onClick={handleClick}>{count}</button>;
}
```

Rewrite `handleClick` so the count increases by 2.

---

### 2. Why does this render an infinite loop, and what are two ways to fix it?

```jsx
function UserList() {
    const [users, setUsers] = useState([]);

    useEffect(() => {
        fetch("/api/users")
            .then(res => res.json())
            .then(setUsers);
    }, [users]);

    return <ul>{users.map(u => <li key={u.id}>{u.name}</li>)}</ul>;
}
```

---

### 3. What does the counter display after 5 seconds, and why?

```jsx
function Timer() {
    const [count, setCount] = useState(0);

    useEffect(() => {
        const id = setInterval(() => setCount(count + 1), 1000);
        return () => clearInterval(id);
    }, []);

    return <p>{count}</p>;
}
```

---

### 4. Why does the listener never get removed?

```jsx
useEffect(() => {
    window.addEventListener("resize", () => setWidth(window.innerWidth));

    return () => {
        window.removeEventListener("resize", () => setWidth(window.innerWidth));
    };
}, []);
```

---

### 5. `Child` is wrapped in `React.memo`. Does it re-render when the parent's `count` changes? Why?

```jsx
function Parent() {
    const [count, setCount] = useState(0);

    const user = { name: "Rakesh" };

    const handleSave = useCallback(() => save(user), [user]);

    return (
        <>
            <button onClick={() => setCount(count + 1)}>{count}</button>
            <Child user={user} onSave={handleSave} />
        </>
    );
}
```

---

### 6. Two components call `useCounter()`. Clicking the first component's button — what happens in the second?

```jsx
function useCounter() {
    const [count, setCount] = useState(0);
    return { count, increment: () => setCount(c => c + 1) };
}
```

---

### 7. In development with Strict Mode, how many times does each log appear on mount, and how many in production?

```jsx
function Widget() {
    console.log("render");

    const [value] = useState(() => {
        console.log("init");
        return 0;
    });

    useEffect(() => {
        console.log("effect");
        return () => console.log("cleanup");
    }, []);

    return <p>{value}</p>;
}
```

---

### 8. Rewrite without the effect

```jsx
function Cart({ items }) {
    const [total, setTotal] = useState(0);

    useEffect(() => {
        setTotal(items.reduce((sum, i) => sum + i.price, 0));
    }, [items]);

    return <p>Total: {total}</p>;
}
```

---

# Coding Practice

Build these small, without libraries.

1. **`useToggle`** — returns `[value, toggle]`. Make sure `toggle` is stable across renders.
2. **`useDebounce(value, delay)`** — returns the value after it has stopped changing for `delay` ms. Cleanup must cancel the pending timer.
3. **`useLocalStorage(key, initial)`** — `useState` that persists. Use lazy initialization for the read.
4. **`useFetch(url)`** — returns `{ data, error, status }`, aborts on unmount and on `url` change, and survives Strict Mode's double mount without a duplicate request landing.
5. **`usePrevious(value)`** — returns the value from the previous render. A ref, updated in an effect.
6. **Stopwatch** — start, stop, reset, with a `useRef` for the interval id. Then rewrite the state as a `useReducer` and compare which reads better.

---

| ← Previous | Index | Next → |
| ---------- | ----- | ------ |
| [Module 8 — React Fundamentals](react-fundamentals.md) | [Study Plan](../README.md) | [Module 10 — React Performance](react-performance.md) |
