[← Study Plan](../README.md)

# Module 10 — React Performance

> **Difficulty:** ⭐⭐⭐⭐☆
> **Interview Frequency:** ⭐⭐⭐⭐⭐
> **Importance for React Developers:** ⭐⭐⭐⭐⭐

Performance is the area where you already got positive interview feedback — `React.memo`, `useMemo`, `useCallback` and React 18 rendering were noted as strengths. So this module is not about learning the tools. It is about the layer above them: **the reasoning, the trade-offs, and the production scenarios** that separate "I know the three memoization APIs" from "I know how to make a slow app fast."

The single most valuable thing in this note is the *order of operations*: measure, find the bottleneck, fix the right layer, measure again. Candidates who lead with `useMemo` sound junior. Candidates who lead with the profiler sound senior.

---

# Table of Contents

```text
1.  What "React Performance" Actually Means
2.  How React Rendering Works
3.  What Causes a Re-render
4.  Re-render ≠ DOM Update
5.  The Default Rule — Parents Re-render Children
6.  Referential Equality
7.  React.memo
8.  useMemo
9.  useCallback
10. All Three Together
11. When NOT to Memoize
12. What Is Actually Worth Optimizing
13. State Colocation
14. Passing Children as Props
15. Context Performance
16. Debouncing in React
17. Throttling
18. Debounce vs Throttle
19. useDeferredValue and useTransition
20. Code Splitting
21. React.lazy
22. Suspense
23. List Virtualization
24. Expensive Calculations
25. Images and Assets
26. Network Performance
27. Bundle Optimization
28. Measuring Performance
29. Core Web Vitals
30. The Senior-Level Optimization Strategy
31. Common Performance Mistakes
32. The Performance Mental Model
33. Common Interview Questions
34. Senior-Level Summary
35. Practice Questions
36. Coding Practice
```

---

# What "React Performance" Actually Means

The narrow, junior definition:

> "React performance means using `useMemo` and `useCallback`."

That is far too small. It describes one fix for one class of problem, and in most slow applications it is **not** the class of problem you have.

Performance is really about how much work happens between a user's intent and the pixels that answer it:

* How much JavaScript the browser has to **download**
* How much JavaScript it has to **parse and execute**
* How **often** components render
* How **expensive** each render is
* How much **DOM** exists and how much of it changes
* How many **network requests** you make and how long they take
* How much **memory** you hold

Those are six different layers, and memoization only touches one of them. A component tree that renders perfectly can still feel broken if the initial bundle is 3 MB or the dashboard endpoint takes four seconds.

The process that matters:

```text
Measure
   ↓
Find the bottleneck
   ↓
Understand the cause
   ↓
Fix the right layer
   ↓
Measure again
```

## Important Interview Point

If you are asked "how do you optimize React performance?" and you answer with API names, you have answered a different question. The question is really **"do you know how to find a bottleneck?"** — the APIs are the easy part.

---

# How React Rendering Works

Everything in this module rests on this pipeline. Know it precisely.

```text
State update / props change / context change
                  ↓
        ┌───────────────────┐
        │   RENDER PHASE    │   React calls your component function.
        │                   │   It returns a tree of React elements.
        │   pure, no DOM    │   Nothing has touched the DOM yet.
        └───────────────────┘
                  ↓
        ┌───────────────────┐
        │  RECONCILIATION   │   React diffs the new element tree
        │                   │   against the previous one and builds
        │                   │   a list of the minimum DOM changes.
        └───────────────────┘
                  ↓
        ┌───────────────────┐
        │   COMMIT PHASE    │   React applies those changes to the
        │                   │   real DOM, then runs layout effects,
        │   synchronous     │   paints, then passive effects.
        └───────────────────┘
```

Three vocabulary items interviewers listen for:

* **Render** — React *calling your function*. It is a computation, not a DOM write.
* **Reconciliation** — the diffing algorithm that decides what actually changed.
* **Commit** — the moment the DOM is mutated.

## Reconciliation, briefly

React compares the two element trees with a heuristic O(n) algorithm rather than a general tree-diff (which would be O(n³)). Two assumptions make that possible:

1. **Different element types produce different trees.** `<div>` → `<span>` means React tears the old subtree down and builds a new one; it does not try to match children across the change.
2. **Keys tell React which children are the same across renders.** This is the whole reason `key` exists, and why an index key on a reorderable list is a bug rather than a style preference (see [Module 8](react-fundamentals.md)).

## Important Interview Point

"Virtual DOM" is not what makes React fast — a hand-written direct DOM update is always faster than a diff. What the Virtual DOM buys you is a **declarative programming model with predictable, batched DOM writes**. The honest framing:

> "React isn't fast *because* of the Virtual DOM. The Virtual DOM is what lets me write declarative UI without hand-managing DOM mutations, and it makes the updates it does perform reasonably minimal and batched."

That answer stands out, because most candidates repeat "the Virtual DOM makes React fast" without ever questioning it.

---

# What Causes a Re-render

Exactly four things.

## 1. Its own state changed

```jsx
const [count, setCount] = useState(0);

setCount(1);
```

One nuance: if you set state to a value that is `Object.is`-equal to the current one, React may bail out and skip re-rendering the children.

```jsx
const [count, setCount] = useState(0);

setCount(0);        // same value — React can bail out
```

React may still call the component once more before deciding to bail — which is why this is a bail-out optimization, not a guarantee you can build logic on.

## 2. Its parent re-rendered

```jsx
function Parent() {
    const [count, setCount] = useState(0);

    return (
        <>
            <button onClick={() => setCount(count + 1)}>
                {count}
            </button>

            <Child />
        </>
    );
}
```

`Child` takes no props at all and still re-renders when `Parent` does. **This is the default and it is not a bug.** React re-renders the whole subtree below the component that updated, unless something explicitly stops it.

## 3. A context it consumes changed

A component calling `useContext(SomeContext)` re-renders when that provider's `value` changes — compared with `Object.is`.

## 4. An external store it subscribes to changed

Redux, Zustand, a router, a media-query subscription. Under the hood modern libraries use `useSyncExternalStore`, which is React 18's official way for outside state to trigger a render safely under concurrent rendering.

## Important Interview Point — what is *not* on this list

**Props changing does not cause a re-render.** This sounds wrong until you think about it: props change *because* the parent re-rendered, and the parent re-rendering is what re-renders the child. There is no mechanism by which a prop value notifies a component.

That is precisely why `React.memo` works the way it does — it can only *stop* an already-triggered parent-driven render by inspecting the props afterwards.

---

# Re-render ≠ DOM Update

This distinction is worth a lot of interview credit, because most candidates blur it.

```jsx
function Counter() {
    const [count, setCount] = useState(0);

    console.log("render");

    return <h1>Hello</h1>;
}
```

Click a button that calls `setCount(c => c + 1)` five times:

Output:

```text
render
render
render
render
render
```

Five renders. **Zero DOM updates** — the output is `<h1>Hello</h1>` every single time, so reconciliation finds nothing to commit.

The flow again, with the important word bolded:

```text
State update
     ↓
React calls your component  ← this is the "render"
     ↓
New element tree
     ↓
Reconciliation (diff)
     ↓
DOM commit  ← only if something actually differs
```

Do not say:

> "React only re-renders the DOM element that changed."

Two words are wrong in that sentence. React doesn't "re-render the DOM"; it renders *components* and commits *DOM changes*. A component can render many times and change no DOM at all.

Say instead:

> "A re-render means React called the component function again. Whether that produces a DOM change depends on what reconciliation finds. Renders are usually cheap; the expensive parts are big subtrees, heavy computation during render, and large DOM commits."

---

# The Default Rule — Parents Re-render Children

Internalize this, because most "unnecessary render" bugs come from forgetting it:

> **When a component re-renders, React re-renders every component it returned, recursively — regardless of whether their props changed.**

```jsx
function App() {
    const [count, setCount] = useState(0);

    return (
        <div>
            <button onClick={() => setCount(count + 1)}>{count}</button>
            <Header />
            <Sidebar />
            <Dashboard />
        </div>
    );
}
```

One click re-renders `Header`, `Sidebar`, and `Dashboard` and everything beneath them. None of them received a single prop.

There are exactly three ways out:

1. **`React.memo`** — compare props and bail out.
2. **Move the state down** so the expensive subtree isn't under the component that updates ([State Colocation](#state-colocation)).
3. **Pass the subtree in as `children` or a prop element** so its element object is created by a parent that *didn't* re-render ([Passing Children as Props](#passing-children-as-props)).

Options 2 and 3 are architectural, cost nothing at runtime, and are the ones juniors never mention. Reach for them first.

---

# Referential Equality

Everything about memoization in React reduces to this one JavaScript fact.

Primitives compare **by value**:

```javascript
1 === 1;              // true
"hello" === "hello";  // true
true === true;        // true
```

Objects, arrays, and functions compare **by reference**:

```javascript
const a = {};
const b = {};

console.log(a === b);   // false
console.log(a === a);   // true
```

Output:

```text
false
true
```

```javascript
const arr1 = [1, 2];
const arr2 = [1, 2];

const fn1 = () => {};
const fn2 = () => {};

console.log(arr1 === arr2);   // false
console.log(fn1 === fn2);     // false
```

Output:

```text
false
false
```

> Note: write these with variables, not as bare `{} === {}`. At the start of a statement, JavaScript parses `{` as a **block**, not an object literal, so `{} === {}` is a syntax error rather than `false`. Some consoles paper over it; the correct expression form is `({}) === ({})`. The concept is right, the one-liner isn't.

Every evaluation of an object literal, array literal, or function expression **creates a new reference**. So in a component body:

```jsx
function Parent() {
    const user = { name: "Rakesh" };     // new object every render
    const items = [];                    // new array every render
    const onClick = () => save();        // new function every render

    return <Child user={user} items={items} onClick={onClick} />;
}
```

Every render hands `Child` three props that are "the same data" and "different values". React's shallow comparison uses `Object.is`, so all three look changed.

## Important Interview Point

This is why the answer to "why does my memoized component still re-render?" is almost always **"an inline object, array, or function prop"** — and why the fix is `useMemo` / `useCallback` rather than a second `React.memo`. Referential equality is the mechanism; the Hooks are just how you preserve a reference across renders.

The background theory is [Module 2 — Data Types & Memory](../javascript/data-types.md).

---

# `React.memo`

`React.memo` wraps a component and lets React **skip re-rendering it when its props are shallowly equal to the previous props**.

```jsx
const UserCard = React.memo(function UserCard({ name }) {
    console.log("UserCard rendered");

    return <div>{name}</div>;
});
```

```jsx
function App() {
    const [count, setCount] = useState(0);

    return (
        <>
            <button onClick={() => setCount(count + 1)}>{count}</button>
            <UserCard name="Rakesh" />
        </>
    );
}
```

Output on three clicks:

```text
UserCard rendered
```

Rendered once, on mount. `name` is a string, so it compares equal every time and React reuses the previous result for the whole `UserCard` subtree.

## What "shallow comparison" means

React compares each prop with `Object.is`, one level deep:

```text
prevProps.name === nextProps.name       ✅ compared
prevProps.user === nextProps.user       ✅ compared (reference only)
prevProps.user.name                     ❌ never looked at
```

So this defeats it completely:

```jsx
<UserCard user={{ name: "Rakesh" }} />
```

A new object literal every render → new reference → props "changed" → re-render. `React.memo` did nothing except add a comparison you now pay for.

## The custom comparator

```jsx
const UserCard = React.memo(
    function UserCard({ user }) {
        return <div>{user.name}</div>;
    },
    (prevProps, nextProps) => prevProps.user.id === nextProps.user.id
);
```

The comparator returns **`true` when the props are equal, meaning "skip the render."** That is the **opposite** of the class-component `shouldComponentUpdate`, which returns `true` to *allow* the update. Getting this backwards is a classic interview slip — and a nasty production bug, because a component that never updates looks frozen.

Use it sparingly. A deep comparison over a big object can cost more than the render it prevents.

## When `React.memo` does nothing

`React.memo` only blocks re-renders that come **from the parent**. It cannot stop a re-render caused by:

* the component's **own state** changing
* a **context** the component consumes changing
* an **external store** it subscribes to changing

That last two are worth saying out loud in an interview, because "wrap the consumer in `React.memo`" is the standard wrong answer to context performance problems.

---

# `useMemo`

`useMemo` memoizes **a computed value** between renders.

```jsx
const filteredUsers = useMemo(() => {
    return users.filter(user =>
        user.name.toLowerCase().includes(search.toLowerCase())
    );
}, [users, search]);
```

While `users` keeps the same reference and `search` the same value, React returns the previously computed array instead of filtering again — and, just as importantly, returns the **same array reference**, so a memoized child receiving it can bail out.

Two legitimate reasons to use it, and they are different:

**1. The computation is genuinely expensive.** Sorting or filtering thousands of rows, parsing a large payload, running a date library over a long list. "Expensive" means milliseconds you can see in the profiler — not a template string.

**2. You need a stable reference.** Often the more important one:

```jsx
const config = useMemo(() => ({ id, mode }), [id, mode]);

useEffect(() => {
    connect(config);
}, [config]);
```

Without the memo, `config` is a fresh object every render, so the effect re-runs every render and any memoized child receiving it re-renders too.

## A third use worth knowing

You can memoize **JSX** — element objects are just objects:

```jsx
const expensiveTree = useMemo(
    () => <ExpensiveChart data={data} />,
    [data]
);

return <Layout>{expensiveTree}</Layout>;
```

Same element reference → React bails out on that subtree, with no `React.memo` wrapper anywhere. This is a useful trick when you can't modify the child component.

## Important Interview Point

`useMemo` is a **performance hint, not a semantic guarantee**. React reserves the right to throw a cached value away (it does exactly this to free memory for offscreen content). Your code must still be correct if the function runs on every render — so never put a side effect, a subscription, or a mutation inside `useMemo`.

---

# `useCallback`

`useCallback` memoizes **a function reference**.

```jsx
const handleClick = useCallback(() => {
    console.log("clicked");
}, []);
```

It is literally `useMemo` for functions:

```jsx
useCallback(fn, deps)  ===  useMemo(() => fn, deps)
```

The purpose is **not**:

> "It makes the function faster."

That's wrong, and interviewers listen for it. The function body is identical either way. The purpose is:

> **Keep the same function reference across renders, so that something comparing references can bail out.**

The "something" is one of exactly two things:

1. A child wrapped in `React.memo`.
2. A dependency array — `useEffect`, `useMemo`, or another `useCallback` that lists this function.

```jsx
const Child = React.memo(function Child({ onClick }) {
    console.log("Child render");

    return <button onClick={onClick}>Click</button>;
});
```

Without `useCallback`, clicking the parent's counter three times:

Output:

```text
Child render
Child render
Child render
```

With `useCallback(fn, [])`:

Output:

```text
(nothing — Child does not re-render)
```

## Important Interview Point

**`useCallback` on its own does nothing for performance.** If the child isn't memoized and the function isn't a Hook dependency, you have added an allocation, a dependency array to keep correct, and a comparison every render — in exchange for nothing. Interviewers ask about `useCallback` specifically to see whether you know that it only pays off as half of a pair.

---

# All Three Together

This is the canonical example. Know it cold.

```jsx
const UserList = React.memo(function UserList({ users, onSelect }) {
    console.log("UserList render");

    return (
        <ul>
            {users.map(user => (
                <li key={user.id}>
                    <button onClick={() => onSelect(user.id)}>
                        {user.name}
                    </button>
                </li>
            ))}
        </ul>
    );
});
```

```jsx
function App({ users }) {
    const [search, setSearch] = useState("");
    const [theme, setTheme] = useState("light");

    const filteredUsers = useMemo(
        () => users.filter(user => user.name.includes(search)),
        [users, search]
    );

    const handleSelect = useCallback((id) => {
        console.log("selected", id);
    }, []);

    return (
        <>
            <button onClick={() => setTheme(t => t === "light" ? "dark" : "light")}>
                {theme}
            </button>

            <input value={search} onChange={e => setSearch(e.target.value)} />

            <UserList users={filteredUsers} onSelect={handleSelect} />
        </>
    );
}
```

Toggling the theme re-renders `App`, but:

* `useMemo` returns the **same** `filteredUsers` array (`users` and `search` are unchanged)
* `useCallback` returns the **same** `handleSelect`
* `React.memo` sees both props equal by reference and skips `UserList`

Output when toggling the theme:

```text
(nothing)
```

Remove any *one* of the three and `UserList` renders on every theme toggle. That's the point of the example: they are a chain, and the chain is only as strong as its weakest link. A single unmemoized inline prop — `style={{ margin: 8 }}`, `items={[]}`, `onClose={() => setOpen(false)}` — breaks the whole thing.

## The comparison table

| Tool          | Memoizes                       | Prevents                              | Useless without                         |
| ------------- | ------------------------------ | ------------------------------------- | --------------------------------------- |
| `React.memo`  | A component's rendered output  | Re-render caused by the parent        | Stable props                             |
| `useMemo`     | A **value** (the result)       | Recomputation + reference churn       | A consumer that cares about the value    |
| `useCallback` | A **function** (uncalled)      | Reference churn                       | `React.memo` or a dependency array       |

Compressed:

```text
React.memo    →  Component
useMemo       →  Value
useCallback   →  Function
```

And the one-liner interviewers like: **`useCallback(fn, deps)` is `useMemo(() => fn, deps)`.**

---

# When NOT to Memoize

This is the senior half of the topic, and it is where the positive feedback you already have turns into a genuinely strong answer.

**Memoization is not free.** Every `useMemo` / `useCallback` / `React.memo` costs:

* memory to hold the cached value **and** the dependency array
* a comparison on **every** render, forever
* a dependency array that has to stay correct — a missing dep is a stale-value bug, not a slow render
* extra indirection for every future reader of the code

So this is a net loss:

```jsx
const fullName = useMemo(
    () => `${firstName} ${lastName}`,
    [firstName, lastName]
);
```

String concatenation is measured in nanoseconds. The memo machinery around it costs more than the work it skips, and you've made the component harder to read to achieve it.

Same for:

```jsx
const doubled = useMemo(() => count * 2, [count]);      // ❌
const isEmpty = useMemo(() => items.length === 0, [items]);  // ❌
```

## The honest position to state in an interview

> "Memoization has a real cost — memory, a comparison on every render, and a dependency array that can go stale. I memoize when profiling shows an actual problem, or when a reference has to stay stable for a `React.memo` child or a Hook dependency. Otherwise the cheapest render is the one where React just calls the function."

## Important Interview Point — the React Compiler

React 19's **React Compiler** inserts this memoization automatically at build time, which removes most of the need to write `useMemo` and `useCallback` by hand. Mentioning it shows you follow the ecosystem — but keep the fundamentals sharp, because it is opt-in, most codebases haven't adopted it, and you still have to explain what it is doing on your behalf.

---

# What Is Actually Worth Optimizing

Good candidates for memoization and related work:

### Genuinely expensive computation

```javascript
const sorted = useMemo(
    () => rows.slice().sort(comparator),
    [rows, comparator]
);
```

Thousands of rows, a heavy transform, a parse over a large payload.

### Large or complex subtrees under frequently-updating state

```text
typing in a search box
        ↓
state update on every keystroke
        ↓
re-render
        ↓
a 400-node dashboard renders 12 times a second
```

### Components rendered many times over

A memoized row component in a 1,000-row table earns its comparison a thousand times per render pass. A memoized page header does not.

### Props that are objects, arrays, or functions

Where referential stability is the *only* thing standing between you and a full subtree re-render.

### The inverse

Do not memoize: leaf components with primitive props, components that render once, cheap derived values, or anything you haven't measured. And note that if a component's props change on every render anyway, `React.memo` is pure overhead — you've added a comparison that always fails.

---

# State Colocation

The optimization people forget, and usually the best one available.

```jsx
function App() {
    const [search, setSearch] = useState("");

    return (
        <>
            <SearchBox value={search} onChange={setSearch} />
            <HugeDashboard />
        </>
    );
}
```

Every keystroke updates `App`'s state → `App` re-renders → `HugeDashboard` re-renders. The dashboard doesn't know the search box exists.

Move the state to the component that actually needs it:

```jsx
function SearchSection() {
    const [search, setSearch] = useState("");

    return <SearchBox value={search} onChange={setSearch} />;
}

function App() {
    return (
        <>
            <SearchSection />
            <HugeDashboard />
        </>
    );
}
```

Now a keystroke re-renders `SearchSection` and nothing else. `HugeDashboard` is no longer in the update path at all.

> **State colocation:** keep state as close as possible to the components that read it. Lift it only as high as it actually needs to go.

Why this beats memoization:

* No comparison cost at runtime — the render simply never happens
* No dependency arrays to keep correct
* The code gets *simpler*, not more complex
* It cannot go stale

## Important Interview Point

The instinct many developers learned early is "lift state up." That's correct advice for *sharing*, but it gets over-applied until every piece of state lives in a top-level `App`. State that lives higher than it needs to is one of the most common structural performance problems in real React codebases — and the fix reads as an architecture answer rather than an API answer, which is exactly the register you want in a senior interview.

---

# Passing Children as Props

The second architectural trick, and the one that impresses. Sometimes you *can't* move the state down, because the state lives in a component that must wrap the expensive tree.

```jsx
function App() {
    const [count, setCount] = useState(0);

    return (
        <div onClick={() => setCount(count + 1)}>
            <p>{count}</p>
            <HugeDashboard />     {/* re-renders on every click */}
        </div>
    );
}
```

Restructure so the stateful component receives the expensive tree as `children`:

```jsx
function Counter({ children }) {
    const [count, setCount] = useState(0);

    return (
        <div onClick={() => setCount(count + 1)}>
            <p>{count}</p>
            {children}
        </div>
    );
}

function App() {
    return (
        <Counter>
            <HugeDashboard />
        </Counter>
    );
}
```

Now clicking re-renders `Counter` only. `HugeDashboard` does not re-render, and there is no `React.memo` anywhere.

**Why it works:** the `<HugeDashboard />` element object is created in `App`. `App` isn't re-rendering — only `Counter` is. So `Counter` receives the *identical* `children` element reference it had last time, and React bails out on that subtree.

This pattern goes by "lifting content up" or "components as props". It is worth having ready, because when an interviewer asks "how would you prevent this re-render **without** memoization?", this is the answer they're hoping for.

---

# Context Performance

Context is a genuine performance footgun, and this is a favourite senior question.

The rule: **every component consuming a context re-renders when that context's `value` changes**, compared with `Object.is`. There is no partial subscription — reading one field of a context object subscribes you to the whole object.

## Problem 1 — a new value object every render

```jsx
<AppContext.Provider value={{ user, theme, setTheme }}>
    {children}
</AppContext.Provider>
```

That object literal is recreated on every provider render, so every consumer re-renders every time — even if `user` and `theme` are untouched.

Fix:

```jsx
const value = useMemo(
    () => ({ user, theme, setTheme }),
    [user, theme, setTheme]
);

return (
    <AppContext.Provider value={value}>
        {children}
    </AppContext.Provider>
);
```

## Problem 2 — one context holding everything

Even with the memo, a component that only reads `theme` still re-renders when `user` changes, because the value object's reference changed.

Split by **update frequency**:

```text
❌ AppContext { user, theme, settings, notifications, cart }

✅ UserContext          — changes on login/logout
✅ ThemeContext         — changes rarely
✅ NotificationContext  — changes constantly
```

## Problem 3 — state and dispatch in one value

A very common refinement. The state changes constantly; the dispatch function never does.

```jsx
const StateContext = createContext(null);
const DispatchContext = createContext(null);

function Provider({ children }) {
    const [state, dispatch] = useReducer(reducer, initialState);

    return (
        <DispatchContext.Provider value={dispatch}>
            <StateContext.Provider value={state}>
                {children}
            </StateContext.Provider>
        </DispatchContext.Provider>
    );
}
```

Components that only *dispatch* — buttons, forms — now never re-render when the state changes. `dispatch` from `useReducer` is guaranteed stable, so no memo is needed on it.

## Important Interview Point

`React.memo` on a consumer **does not** stop a context-driven re-render. The subscription is inside the component. If asked "how do you stop a context consumer from re-rendering?", the real answers are: split the context, memoize the value, move the consumer down so less re-renders with it, or use an external store (Zustand, Redux with selectors) where subscribers can select a slice.

React has **no built-in context selector**. That's a genuine limitation worth naming — libraries like `use-context-selector` exist precisely because of it, and it's a large part of why teams reach for Zustand or Redux Toolkit for high-frequency global state.

The wider Context-vs-Redux discussion is in [Module 9](react-hooks.md).

---

# Debouncing in React

**Debounce: wait until the activity stops, then run once.**

Without it, a search input fires a request per keystroke:

```text
r  ra  rak  rake  rakes  rakesh
↓   ↓    ↓    ↓     ↓      ↓
6 keystrokes → 6 requests
```

With a 400 ms debounce:

```text
r  ra  rak  rake  rakes  rakesh
                            ↓
                     (400ms of silence)
                            ↓
                       1 request
```

The plain-JavaScript implementation ([Module 6](../javascript/async-event-loop.md)):

```javascript
function debounce(fn, delay) {
    let timeoutId;

    return function (...args) {
        clearTimeout(timeoutId);
        timeoutId = setTimeout(() => fn.apply(this, args), delay);
    };
}
```

Typical delays: **300–500 ms** for search, **~1 s** for autosave. Under ~200 ms users out-type the debounce and you barely save any requests.

## The React trap

This is broken:

```jsx
function Search() {
    const [query, setQuery] = useState("");

    // ❌ new debounced function on EVERY render
    const handleChange = debounce((value) => {
        fetchResults(value);
    }, 400);

    return <input onChange={e => handleChange(e.target.value)} />;
}
```

Every render creates a **new** debounced function with its **own** `timeoutId` closure. Nothing ever gets cancelled, so every keystroke fires 400 ms later — you have added latency and kept all the requests. This is one of the most common real bugs in React codebases, and it's a great thing to be able to spot on sight.

## The idiomatic fix — debounce the *value*, not the callback

```jsx
function useDebounced(value, delay = 400) {
    const [debounced, setDebounced] = useState(value);

    useEffect(() => {
        const id = setTimeout(() => setDebounced(value), delay);
        return () => clearTimeout(id);
    }, [value, delay]);

    return debounced;
}
```

```jsx
function Search() {
    const [query, setQuery] = useState("");
    const debouncedQuery = useDebounced(query, 400);

    useEffect(() => {
        if (!debouncedQuery) return;

        const controller = new AbortController();

        fetch(`/api/search?q=${debouncedQuery}`, { signal: controller.signal })
            .then(res => res.json())
            .then(setResults)
            .catch(err => {
                if (err.name !== "AbortError") setError(err);
            });

        return () => controller.abort();
    }, [debouncedQuery]);

    return <input value={query} onChange={e => setQuery(e.target.value)} />;
}
```

Three things this gets right, and all three are worth saying out loud:

1. **The input stays fully controlled** — `query` updates on every keystroke, so typing never feels laggy. Only the *request* is debounced.
2. **The effect's cleanup cancels the pending timer**, so the debounce actually debounces.
3. **`AbortController` cancels the in-flight request**, which kills the race condition where an older response lands after a newer one.

Debouncing without cancellation is a half-answer. Say both.

If you keep the callback-debounce shape instead, the function reference must be stable — `useMemo(() => debounce(fn, 400), [])` plus a cleanup that cancels on unmount.

---

# Throttling

**Throttle: run at most once per interval, while the activity continues.**

```javascript
function throttle(fn, limit) {
    let waiting = false;

    return function (...args) {
        if (waiting) return;

        fn.apply(this, args);
        waiting = true;

        setTimeout(() => { waiting = false; }, limit);
    };
}
```

Visually:

```text
Scroll events:  ████████████████████████████████

Throttled:      █      █      █      █      █
```

Use it for continuous streams where you want *regular* updates, not just a final one:

* scroll position / scroll-spy
* mouse move, drag
* window resize
* progress reporting
* rate-limited APIs

## The React-specific note

For scroll and resize in particular, prefer the platform when you can. `IntersectionObserver` and `ResizeObserver` do the work off the main thread's hot path and are strictly better than a throttled scroll listener for "is this element visible" or "how big is this box". Naming those two APIs is a nice signal that your performance knowledge isn't only React-shaped.

Also: mark genuinely passive listeners as passive so scrolling isn't blocked:

```javascript
window.addEventListener("scroll", onScroll, { passive: true });
```

---

# Debounce vs Throttle

| | Debounce | Throttle |
| - | -------- | -------- |
| **Rule** | Wait until it stops | At most once per interval |
| **Runs** | Once, at the end | Repeatedly, at a fixed rate |
| **You want** | The **final** value | A **steady stream** of values |
| **Classic use** | Search, autosave, validation | Scroll, drag, resize, mousemove |
| **If the user never stops** | Never fires | Keeps firing on schedule |

That last row is the cleanest way to explain the difference, and it's the version to give in an interview:

> "Debounce waits for silence — if the user never pauses, it never fires. Throttle fires on a schedule regardless. So I ask whether I need the *final* value or a *steady stream*. Search needs the final query; a scroll indicator needs the stream."

---

# `useDeferredValue` and `useTransition`

React 18's concurrent features are the *React-native* answer to a class of problem people reach for debouncing to solve — and mentioning them is a strong differentiator, because most candidates stop at debounce.

The scenario: an input filtering a huge list. The input must feel instant; the list can lag.

```jsx
function Search({ items }) {
    const [query, setQuery] = useState("");
    const deferredQuery = useDeferredValue(query);

    const results = useMemo(
        () => items.filter(i => i.name.includes(deferredQuery)),
        [items, deferredQuery]
    );

    return (
        <>
            <input value={query} onChange={e => setQuery(e.target.value)} />
            <ResultList results={results} />
        </>
    );
}
```

`query` updates immediately so typing is never blocked. `deferredQuery` lags behind, and React renders the expensive list at lower priority — interrupting and restarting that work if the user types again.

`useTransition` is the same idea when you own the update:

```jsx
const [isPending, startTransition] = useTransition();

function handleChange(e) {
    setQuery(e.target.value);              // urgent — the input

    startTransition(() => {
        setFilter(e.target.value);         // non-urgent — the results
    });
}
```

`isPending` lets you show a subtle loading state while the deferred work happens.

## The distinction that matters

```text
Debounce           → do LESS work        (skip renders/requests entirely)
useDeferredValue   → do the work LATER   (at lower priority, interruptible)
```

They solve different problems and compose fine:

* Reducing **network requests**? Debounce. Concurrent features don't help — the request still fires.
* Keeping the **UI responsive** during expensive rendering? `useDeferredValue` / `useTransition`.

Saying that distinction cleanly is a senior answer. The React 18 rendering model behind it is in [Module 9](react-hooks.md).

---

# Code Splitting

Your app has Home, Dashboard, Analytics, Admin, Reports, Settings. A user who lands on Home downloads, parses, and compiles the JavaScript for all six.

```text
Before:                    After:
app.js  ──  2.1 MB         main.js       180 KB   ← loaded now
                           dashboard.js  240 KB   ← on /dashboard
                           analytics.js  610 KB   ← on /analytics
                           admin.js      320 KB   ← almost never
```

**Code splitting** breaks the bundle into chunks the bundler loads on demand. The mechanism is the dynamic `import()` ([Module 7](../javascript/es6-modern-js.md)):

```javascript
const module = await import("./heavyThing.js");
```

A dynamic `import()` returns a promise and tells the bundler "make this a separate chunk."

## Where to split, in priority order

1. **By route** — the highest-value split, and the easiest. Nobody needs the admin panel's code on the login screen.
2. **By heavy dependency** — charting libraries, rich-text editors, PDF viewers, map SDKs, date/locale bundles. One of these is often bigger than your entire application.
3. **By interaction** — modals, drawers, wizards, anything behind a click.

## The trade-off to mention

Splitting too finely creates request waterfalls and loading flashes on every navigation. The counter-measure is **prefetching**: start fetching the chunk on hover or on idle, before the user commits.

```jsx
<Link
    to="/analytics"
    onMouseEnter={() => import("./pages/Analytics")}
>
    Analytics
</Link>
```

Naming that trade-off is the difference between "I know `React.lazy`" and "I've shipped code splitting."

---

# `React.lazy`

`React.lazy` is React's binding between a dynamic `import()` and the component tree.

```jsx
const AdminDashboard = React.lazy(() => import("./AdminDashboard"));
```

```jsx
<Suspense fallback={<Loading />}>
    <AdminDashboard />
</Suspense>
```

What actually happens:

```text
Bundler sees import()      →  emits AdminDashboard as its own chunk,
                              excluded from the main bundle

<AdminDashboard /> renders →  React calls the loader, which requests
   for the first time         the chunk over the network

While the promise pends   →  the nearest <Suspense> shows its fallback

Promise resolves          →  React renders the real component
                              (the module stays cached — no refetch)
```

The precise claim: the chunk is **not downloaded, parsed, or executed until the component first renders.** Loose phrasings like "it isn't in the initial execution path" are the kind of thing an interviewer will probe.

Rules:

* The lazy module must have a **default export** that is a component.
* The `React.lazy(...)` call goes at **module scope**, not inside a component — declaring it during render creates a new lazy component every render, which remounts the subtree and refetches.
* There must be a `<Suspense>` boundary **somewhere above** it, or React throws.
* Handle failures. A chunk request can fail on a flaky network or after a deploy invalidates the old chunk — wrap lazy routes in an **error boundary** with a retry. This is a real production concern and very few candidates raise it.

---

# `Suspense`

`Suspense` lets a component tell React "I'm not ready yet," and React shows the nearest boundary's fallback instead of the partially-ready tree.

```jsx
<Suspense fallback={<Spinner />}>
    <Profile />
    <Timeline />
</Suspense>
```

Boundary placement is a design decision, not a formality:

```jsx
{/* One spinner replaces the entire page */}
<Suspense fallback={<PageSkeleton />}>
    <Sidebar />
    <Feed />
</Suspense>

{/* Each region loads independently — usually better UX */}
<Suspense fallback={<SidebarSkeleton />}>
    <Sidebar />
</Suspense>
<Suspense fallback={<FeedSkeleton />}>
    <Feed />
</Suspense>
```

Prefer **skeletons matching the final layout** over a centred spinner — they reduce perceived load time and avoid the layout shift that hurts your CLS score.

## What Suspense does and does not do

**Works today, everywhere:** `React.lazy` code splitting.

**Works with integration:** data fetching. Suspense does not know how to fetch anything. It reacts to a component suspending, and that requires a data layer built for it — a framework (Next.js App Router, Remix), a query library (`useSuspenseQuery` in TanStack Query), or React 19's `use()` Hook. Writing `fetch` inside `useEffect` will **never** trigger Suspense.

That distinction is a good interview answer, because "Suspense handles async data" is the confident-sounding wrong answer.

---

# List Virtualization

Rendering 100,000 rows means 100,000 components rendered, ~100,000+ DOM nodes created, style and layout computed for all of them, and memory held for all of them. The browser will struggle regardless of how good your React code is.

**Virtualization (windowing)** renders only what's on screen, plus a small overscan buffer:

```text
100,000 items in data
          ↓
   ┌─────────────┐
   │  viewport   │  ← ~20 rendered DOM nodes
   └─────────────┘
          ↓
scroll → recycle the window, swap the rows
```

The container is given the full scroll height (`itemCount × itemHeight`) so the scrollbar behaves correctly, and the visible rows are absolutely positioned at the right offset. Scrolling swaps which items are rendered.

Libraries: **`react-window`**, **TanStack Virtual**, **`react-virtuoso`**. Don't hand-roll it — variable heights, sticky headers, keyboard navigation, and scroll restoration are where it gets hard.

Use it for: data tables, chat histories, infinite feeds, file explorers, long dropdowns and autocompletes, log viewers.

## The trade-offs to name

* **Ctrl+F breaks** — off-screen content isn't in the DOM.
* **Accessibility needs care** — a virtualized list needs correct ARIA roles and `aria-setsize` / `aria-posinset` so screen readers know the real size.
* **Variable-height rows** need measurement and are noticeably harder than fixed-height rows.
* **Print and SEO** see only the rendered window.

And the question to ask before reaching for it: **does the user actually need 100,000 rows?** Pagination, search, or filtering is often the better product answer. "I'd first check whether the requirement is real" is a good thing to say out loud.

---

# Expensive Calculations

When the profiler shows time spent *inside* a component rather than in the number of renders, memoization is only one option — and often not the best.

**1. Improve the algorithm.** An O(n²) lookup inside a `.map` beats every memo you could add:

```jsx
// ❌ O(n × m) — a scan per row
rows.map(row => users.find(u => u.id === row.userId));

// ✅ O(n + m) — one index, then lookups
const usersById = useMemo(
    () => new Map(users.map(u => [u.id, u])),
    [users]
);

rows.map(row => usersById.get(row.userId));
```

`Map`, `Set`, and complexity are in [Module 4](../javascript/objects-arrays.md) and [Module 7](../javascript/es6-modern-js.md).

**2. Move it out of render.** Sorting and filtering on the **server** beats doing it in the browser for large datasets, every time.

**3. Compute it once, not per item.** Hoist invariant work out of the loop — `new Intl.NumberFormat(...)` created inside a `.map` over 5,000 rows is a real and common cost.

**4. Memoize it** — `useMemo`, once you've measured it.

**5. Get it off the main thread.** For genuinely heavy work (parsing a large file, image processing, crypto, big data transforms), a **Web Worker** keeps the UI responsive in a way no React API can. Mentioning Web Workers signals that you know React's rendering budget shares one thread with everything else.

**6. Defer it** — `useDeferredValue`, so the expensive render doesn't block input.

---

# Images and Assets

Frontend performance isn't only React. On a typical page, images are the largest thing you ship, and the LCP element is usually an image.

```html
<img
    src="product-800.webp"
    srcset="product-400.webp 400w,
            product-800.webp 800w,
            product-1600.webp 1600w"
    sizes="(max-width: 768px) 100vw, 50vw"
    alt="Product"
    width="800"
    height="600"
    loading="lazy"
    decoding="async"
/>
```

Each attribute earns its place:

* **`srcset` + `sizes`** — the browser picks a resolution appropriate to the viewport and DPR instead of downloading a 1600px image for a 400px slot.
* **`width` + `height`** — reserves the space before the image loads. This is the single biggest fix for **CLS**.
* **`loading="lazy"`** — defers off-screen images. **Never** put it on your LCP image; that delays the metric you're being measured on.
* **Modern formats** — WebP and AVIF are typically 30–50% smaller than JPEG at equivalent quality.

For the hero image, do the opposite of lazy loading — prioritize it:

```html
<link rel="preload" as="image" href="hero.webp" />
```

```html
<img src="hero.webp" fetchpriority="high" alt="" />
```

Also worth naming: serve through a **CDN**, compress properly, and use `font-display: swap` plus preloaded self-hosted fonts so text isn't invisible while a webfont loads.

---

# Network Performance

A perfectly optimized render tree doesn't help if the user waits four seconds for JSON. In real applications the network is more often the bottleneck than rendering is.

### Cache, and stop refetching

A query library (TanStack Query, SWR, RTK Query) gives you caching, deduplication, background refetching, and stale-while-revalidate for free. Most "global state" in a React app is **server state**, and hand-rolling it in `useEffect` is how you end up with duplicate requests and race conditions.

### Kill request waterfalls

```text
❌ Waterfall — 900ms
   fetch user (300ms) → fetch orders (300ms) → fetch items (300ms)

✅ Parallel — 300ms
   Promise.all([fetchUser(), fetchOrders(), fetchItems()])
```

Sequential `await`s that don't depend on each other are one of the most common and most expensive mistakes in React data fetching ([Module 6](../javascript/async-event-loop.md)).

### Cancel what you no longer need

`AbortController` on unmount and on dependency change — it prevents both wasted bandwidth and the race where a stale response overwrites a fresh one.

### Send less

Pagination or infinite scroll instead of "return everything." Ask the backend for the fields you use. Compression (gzip/Brotli) on all text responses. HTTP caching headers so repeat visits hit the cache rather than the origin.

### Start earlier

`preconnect` to your API origin, `dns-prefetch` for third parties, and prefetch the data for the route the user is probably about to open.

### Optimistic updates

For mutations, update the UI immediately and reconcile with the server response. Perceived performance is performance — a "instant" toggle that quietly reverts on failure feels dramatically faster than a spinner.

---

# Bundle Optimization

### Tree shaking

Bundlers drop exports that are never imported. It only works when:

* the module is **ESM** (`import`/`export`, not CommonJS `require`) — static analysis needs statically known imports
* the code is free of side effects, or the package declares `"sideEffects": false` in its `package.json`

That precision matters: "tree shaking removes unused code" is only true under those conditions, and plenty of libraries ship in a form that defeats it.

### Import cost awareness

```javascript
import _ from "lodash";                    // ❌ pulls the whole library
import debounce from "lodash/debounce";    // ✅ one module
import { debounce } from "lodash-es";      // ✅ ESM, tree-shakeable
```

And before either: does an 8-line `debounce` need a dependency at all? For a single small utility, often not. That judgement — dependency weight vs. maintenance cost — is a senior conversation.

Classic offenders worth knowing: `moment` (large, no tree shaking → `date-fns` or `Temporal`/`Intl`), full-icon-set imports, and locale/timezone data bundles.

### Analyze before you guess

* `rollup-plugin-visualizer` (Vite)
* `webpack-bundle-analyzer` (webpack)
* `source-map-explorer` (any build with source maps)
* Chrome DevTools **Coverage** tab — shows how much of your shipped JS/CSS actually executed on load

The Coverage tab is the most under-used tool on that list, and "I'd check Coverage to see how much of the bundle is unused on first paint" is a great line in an interview.

### Also

Modern build targets (don't ship ES5 polyfills to browsers that don't need them), Brotli compression, long-term caching with hashed filenames, and a vendor/app chunk split so app deploys don't invalidate the vendor cache.

---

# Measuring Performance

Never say:

> "I added `useMemo` and it felt faster."

## React DevTools Profiler

Record an interaction, then read:

* **Which components rendered** — often surprising
* **How long each render took** and the total commit duration
* **Why each one rendered** — props changed / state changed / parent rendered / context changed. Enable "Record why each component rendered" in the Profiler settings; it turns guesswork into a direct answer.
* **The flamegraph** — width is time, so wide bars are your targets
* **The ranked chart** — components sorted by render cost

Also in the Components tab settings: **"Highlight updates when components render."** Turn it on, use the app, and watch for regions flashing that have no business flashing. It finds unnecessary renders in seconds.

One practical detail worth knowing: profiling needs either a **development build** or a **production build with profiling enabled** (`react-dom/profiling`). A plain production build strips the instrumentation. Development builds are also meaningfully slower than production, so use them to find *relative* hot spots, not to quote absolute timings.

## The `<Profiler>` API

For programmatic measurement, including in production:

```jsx
<Profiler id="Sidebar" onRender={(id, phase, actualDuration) => {
    log({ id, phase, actualDuration });
}}>
    <Sidebar />
</Profiler>
```

## Chrome DevTools

* **Performance** — the real flamechart: scripting, layout, paint, long tasks. The only place you can see that your 120 ms "React problem" is actually a 90 ms forced reflow.
* **Network** — waterfalls, payload sizes, cache hits. Throttle to Fast 3G and see what your users see.
* **Coverage** — unused JavaScript and CSS.
* **Memory** — heap snapshots for leak hunting (detached DOM nodes, listeners never removed, timers never cleared).
* **Lighthouse** — a lab audit with concrete suggestions.

## Field data

Lab tools measure your machine. **Real User Monitoring** measures your users — the `web-vitals` library, or whatever your platform provides. Saying "lab data finds problems, field data tells you whether they matter" is a strong, senior-sounding distinction.

---

# Core Web Vitals

Google's three user-centric metrics. Know what each one measures, its threshold, and what breaks it.

| Metric | Measures | Good | Usually caused by |
| ------ | -------- | ---- | ----------------- |
| **LCP** — Largest Contentful Paint | When the main content appears | ≤ 2.5 s | Slow server, render-blocking resources, huge unoptimized hero image, client-side-only rendering |
| **INP** — Interaction to Next Paint | Responsiveness across *all* interactions | ≤ 200 ms | Long tasks blocking the main thread, expensive renders on input, heavy event handlers |
| **CLS** — Cumulative Layout Shift | Unexpected visual movement | ≤ 0.1 | Images without dimensions, injected banners/ads, late-loading fonts, content appearing above existing content |

Two details that show you're current:

* **INP replaced FID** as a Core Web Vital in March 2024. FID only measured the delay before the *first* interaction was processed; INP measures the full interaction-to-paint latency across the whole visit. It is a much harder metric, and it is the one React apps most often fail — because heavy renders block the main thread.
* Thresholds are assessed at the **75th percentile** of real users, not the average.

The React connection to state explicitly: **INP is the React metric.** Unnecessary re-renders, expensive computation during render, and giant DOM commits are exactly what makes interactions slow to paint. Which is where memoization, colocation, virtualization, and `useTransition` genuinely earn their place.

---

# The Senior-Level Optimization Strategy

The question:

> **"How would you optimize a slow React application?"**

The junior answer starts with `useMemo`. Do this instead.

### Step 1 — Reproduce and measure

Find the specific slow interaction. Profile it with React DevTools and the Chrome Performance panel. Check the Network tab. Run Lighthouse. Throttle CPU and network to something like a real device.

### Step 2 — Classify the bottleneck

```text
Slow initial load?     → bundle size, network, server response, images
Slow interactions?     → rendering, expensive computation, long tasks
Slow after a while?    → memory leak, unbounded list, uncleaned subscriptions
Slow data?             → API latency, waterfalls, over-fetching, no caching
Janky scrolling?       → DOM size, layout thrash, unthrottled handlers
```

### Step 3 — Fix the matching layer

```text
Large bundle           →  code splitting, lazy loading, tree shaking, lighter deps
Too many renders       →  state colocation, children-as-props, React.memo
Expensive computation  →  better algorithm, server-side work, useMemo, Web Worker
Huge list              →  virtualization (or pagination — question the requirement)
Too many requests      →  debounce, caching, deduplication, pagination
Slow requests          →  parallelize, cancel, compress, cache, preconnect
Blocked interactions   →  useTransition / useDeferredValue, break up long tasks
Large images           →  responsive images, modern formats, lazy loading, CDN
Layout shift           →  explicit dimensions, skeletons, font-display
```

### Step 4 — Measure again

Confirm the fix moved the number. Then check you didn't regress something else — a memo that added memory, a split that added a waterfall.

### Step 5 — Guard it

Bundle-size budgets in CI, Lighthouse CI, RUM in production. An optimization without a regression guard has a half-life of about two sprints.

## The answer to memorize

> "First I measure rather than blindly adding memoization — React DevTools Profiler and Chrome Performance to see whether the bottleneck is unnecessary rendering, expensive computation, network, bundle size, or DOM size. Then I fix the matching layer: state colocation or restructuring props before `React.memo`; a better algorithm or a Web Worker before `useMemo`; virtualization for large lists; debouncing and caching for requests; code splitting and lazy loading for bundle size. Then I measure again to confirm it actually helped, and add a budget or RUM so it doesn't regress. Most of the time the biggest win isn't a memo — it's moving state down, splitting a route, or fixing a request waterfall."

That last sentence is the one that sounds like someone who has actually done it.

---

# Common Performance Mistakes

### ❌ Memoizing everything on principle

Cost with no measured benefit, plus dependency arrays that will eventually go stale.

### ❌ `React.memo` on a component with inline object/array/function props

The comparison runs every render and always fails. Strictly worse than no memo.

### ❌ Defining a component inside another component

```jsx
function Parent() {
    function Child() { ... }        // ❌ new component type every render

    return <Child />;
}
```

React sees a *different type* each render, so it unmounts and remounts the subtree — losing all its state, refs, and DOM. This is one of the worst and most common performance-and-correctness bugs in React, and it doesn't look like a bug.

### ❌ Index as `key` in a reorderable or filterable list

State attaches to the wrong row, inputs show the wrong values, animations break ([Module 8](react-fundamentals.md)).

### ❌ Keeping state too high

One keystroke re-rendering an entire dashboard. Colocate.

### ❌ One giant Context

Every consumer re-renders on every change to anything.

### ❌ Rendering thousands of DOM nodes

Virtualize, paginate, or question the requirement.

### ❌ Fetching on every keystroke

Debounce plus cancel.

### ❌ Sequential awaits that could be parallel

Three 300 ms requests taking 900 ms.

### ❌ Effects with missing or wrong dependencies

Either a stale value (a correctness bug) or an effect running every render (a performance bug).

### ❌ Never cleaning up

Intervals, listeners, observers, and subscriptions that outlive the component. This is a memory leak *and* a slow degradation — the app is fine for a minute and unusable after ten.

### ❌ Optimizing in development and never checking production

Dev builds are slower, Strict Mode double-renders, and `console.log`s in a render path cost real time.

---

# The Performance Mental Model

```text
                        React Performance
                                │
        ┌───────────────┬───────┴───────┬───────────────┐
        │               │               │               │
    Rendering        Network         Bundle          Assets
        │               │               │               │
  ┌─────┴─────┐    ┌────┴────┐     ┌────┴────┐     ┌────┴────┐
  │           │    │         │     │         │     │         │
Fewer      Cheaper  Fewer   Faster  Smaller  Later  Smaller  Later
renders    renders  requests reqs   bundle   chunks  images  loading
  │           │       │        │       │        │       │       │
State      useMemo  Debounce Parallel Tree-  Code   srcset  loading
coloca-    Virtua-  Caching  Abort    shaking split  WebP/   ="lazy"
tion       lization Dedupe   Compress Lighter Lazy   AVIF    preload
children-  Web      Pagina-  Preconn- deps    load   width/  hero
as-props   Workers  tion     ect             prefetch height
React.memo useDefer
useCallback redValue

                                │
                                ▼
                     ALWAYS: measure → fix → measure
```

Compressed to one line:

**Render less often → render less work → ship less code → fetch less data → and prove it with a profiler.**

---

# Common Interview Questions

Aim to answer each out loud in **1–3 minutes**.

### Rendering

1. What causes a React component to re-render?
2. Does a re-render always mean a DOM update?
3. What is reconciliation, and why is it O(n)?
4. Is the Virtual DOM faster than direct DOM manipulation?
5. If a parent re-renders, do all children re-render? Why?
6. How would you find unnecessary renders in an app you've never seen?

### Memoization

7. What does `React.memo` do, and what does it compare?
8. What is `useMemo` for? Name two distinct reasons to use it.
9. What is `useCallback` for? Does it make the function faster?
10. What's the difference between the three?
11. What is referential equality, and why does it matter in React?
12. Why does a memoized child still re-render when it receives `style={{ color: "red" }}`?
13. When should you *not* use `useMemo` or `useCallback`?
14. Does `React.memo` prevent a re-render caused by context?
15. What does `React.memo`'s custom comparator return to skip a render?

### Architecture

16. What is state colocation, and why is it often better than memoization?
17. How can you stop a child re-rendering *without* `React.memo`?
18. How can Context hurt performance, and what are three ways to fix it?
19. Why would you split one context into several?

### Events and concurrency

20. What is debouncing? What is throttling? When would you use each?
21. How do you implement a debounced search in React correctly?
22. What breaks if you create the debounced function inside the component body?
23. What is `useDeferredValue`, and how is it different from debouncing?
24. When would you use `useTransition`?

### Loading

25. What is code splitting, and where would you split first?
26. How does `React.lazy` work?
27. What is `Suspense`, and does it handle data fetching?
28. What happens if a lazy chunk fails to load?

### Large data

29. How would you render 100,000 rows?
30. How does virtualization work under the hood?
31. What are the downsides of virtualization?

### Production

32. How would you investigate a slow React application?
33. What tools would you use, and what does each one tell you?
34. What are LCP, INP, and CLS, and what are their thresholds?
35. Which Core Web Vital do React apps most often fail, and why?
36. How would you reduce a 2 MB JavaScript bundle?
37. How do you prevent performance regressions after you've fixed them?

---

# Senior-Level Summary

By the end of this module, you should be able to explain:

* Performance is **six layers** — download, parse/execute, render frequency, render cost, DOM size, network — and memoization only touches one of them.
* Rendering is **render → reconciliation → commit**. A **render** is React calling your function; it often produces **no DOM change at all**.
* The Virtual DOM doesn't make React fast; it makes declarative UI possible with predictable, batched DOM writes.
* Four things cause a re-render: **own state, parent re-rendered, context changed, external store changed**. Props changing is not a cause — it's a consequence.
* By default a re-render cascades through the **entire subtree**, regardless of props.
* Everything about memoization reduces to **referential equality**: objects, arrays, and functions are new on every render, and React compares props with `Object.is`.
* `React.memo` skips a **parent-driven** re-render on shallow prop equality; it cannot stop one caused by the component's own state, context, or an external store. Its custom comparator returns `true` to **skip**.
* `useMemo` memoizes a **value**, `useCallback` a **function** — and `useCallback(fn, deps)` is exactly `useMemo(() => fn, deps)`. Both are **hints, not guarantees**.
* `useCallback` alone does nothing: it needs `React.memo` on the other end, or a dependency array consuming it.
* Memoization costs memory, a comparison every render, and a dependency array that can go stale. **Measure first.**
* **State colocation** and **passing children as props** eliminate renders entirely, at zero runtime cost — reach for them before `React.memo`.
* Context re-renders **every consumer** on an `Object.is` change. Memoize the value, split by update frequency, split state from dispatch — and know React has **no built-in selector**.
* **Debounce** waits for silence and fires once; **throttle** fires on a schedule. Debounce needs a **stable reference** and pairs with `AbortController`.
* **`useDeferredValue` / `useTransition`** do the work *later* at lower priority; **debounce** does *less* work. Different problems.
* **Code splitting** by route first, then heavy dependencies, then interactions — and prefetch to avoid trading bundle size for waterfalls.
* `React.lazy` chunks aren't downloaded, parsed, or executed until first render; wrap them in `Suspense` **and** an error boundary.
* **Suspense doesn't fetch data.** It needs a framework, a Suspense-aware query library, or `use()`.
* **Virtualization** renders only the visible window — and costs you Ctrl+F, easy accessibility, and simple variable heights.
* For expensive work: better algorithm > server-side > memoize > Web Worker > defer.
* **INP is the React metric.** LCP ≤ 2.5 s, INP ≤ 200 ms, CLS ≤ 0.1, at the 75th percentile; INP replaced FID in March 2024.
* The strategy, always: **measure → classify → fix the right layer → measure again → guard against regression.**

---

# Practice Questions

Answer these without running them.

### 1. How many times does `Child` render across three clicks, and why?

```jsx
const Child = React.memo(function Child({ config }) {
    console.log("Child render");
    return <div>{config.label}</div>;
});

function Parent() {
    const [count, setCount] = useState(0);

    return (
        <>
            <button onClick={() => setCount(count + 1)}>{count}</button>
            <Child config={{ label: "hello" }} />
        </>
    );
}
```

Fix it two different ways — one with a Hook, one without.

---

### 2. Why does this fire a request for every keystroke, 400 ms late?

```jsx
function Search() {
    const [query, setQuery] = useState("");

    const search = debounce(value => fetchResults(value), 400);

    return (
        <input
            value={query}
            onChange={e => {
                setQuery(e.target.value);
                search(e.target.value);
            }}
        />
    );
}
```

---

### 3. `Row` is memoized and `rows` never changes. Does `Row` re-render when `selectedId` changes? What about when `count` changes?

```jsx
function Table({ rows }) {
    const [selectedId, setSelectedId] = useState(null);
    const [count, setCount] = useState(0);

    const handleSelect = useCallback(id => setSelectedId(id), []);

    return (
        <>
            <button onClick={() => setCount(count + 1)}>{count}</button>
            {rows.map(row => (
                <Row key={row.id} row={row} onSelect={handleSelect} />
            ))}
        </>
    );
}
```

---

### 4. Every consumer re-renders whenever `user` changes, including ones that only read `theme`. Give two different fixes.

```jsx
function AppProvider({ children }) {
    const [user, setUser] = useState(null);
    const [theme, setTheme] = useState("light");

    const value = useMemo(
        () => ({ user, setUser, theme, setTheme }),
        [user, theme]
    );

    return (
        <AppContext.Provider value={value}>
            {children}
        </AppContext.Provider>
    );
}
```

---

### 5. What is wrong here, and what does the user observe?

```jsx
function Dashboard({ userId }) {
    const Chart = React.lazy(() => import("./Chart"));

    return (
        <Suspense fallback={<Spinner />}>
            <Chart userId={userId} />
        </Suspense>
    );
}
```

---

### 6. The input feels laggy while typing. `items` has 50,000 entries. Give two fixes and say what each one does differently.

```jsx
function Filter({ items }) {
    const [query, setQuery] = useState("");

    const visible = items.filter(i => i.name.includes(query));

    return (
        <>
            <input value={query} onChange={e => setQuery(e.target.value)} />
            <List items={visible} />
        </>
    );
}
```

---

### 7. Why does every row lose its input state when a row is deleted?

```jsx
{rows.map((row, index) => (
    <EditableRow key={index} row={row} />
))}
```

---

### 8. Rewrite so `ExpensiveTree` does not re-render when the modal opens or closes — without using `React.memo`.

```jsx
function Page() {
    const [isOpen, setIsOpen] = useState(false);

    return (
        <div>
            <button onClick={() => setIsOpen(true)}>Open</button>
            {isOpen && <Modal onClose={() => setIsOpen(false)} />}
            <ExpensiveTree />
        </div>
    );
}
```

---

# Coding Practice

Build these small, and profile each one before and after.

1. **Render counter** — a `useRenderCount()` Hook that logs how many times a component has rendered. Drop it into a small app and find a render you didn't expect.
2. **Debounced search** — controlled input, 400 ms debounce, `AbortController` cancellation, loading and error states, and no request when the query is empty.
3. **Throttled scroll indicator** — a top progress bar showing scroll position. Build it with `throttle`, then rebuild it with `IntersectionObserver` where possible and compare.
4. **Virtualized list** — render 50,000 rows with fixed heights. Hand-roll the windowing first (compute the visible range from `scrollTop`), then swap in `react-window` and compare what you had to get right.
5. **Route-based code splitting** — three routes with `React.lazy`, a shared `Suspense` skeleton, an error boundary with retry, and prefetch on link hover. Check the Network tab to confirm the chunks load when you expect.
6. **Context split** — build a provider holding `user`, `theme`, and a fast-changing `notifications` list in one context. Add render logs, watch everything re-render, then split it into three and watch the logs go quiet.
7. **The colocation refactor** — an app with a search input and an artificially expensive dashboard, with state at the top. Fix it three ways: `React.memo`, colocation, and children-as-props. Profile all three and decide which you'd ship.

---

| ← Previous | Index | Next → |
| ---------- | ----- | ------ |
| [Module 9 — React Hooks](react-hooks.md) | [Study Plan](../README.md) | Module 11 — State Management *(not written yet)* |
