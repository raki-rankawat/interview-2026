[← Study Plan](../README.md)

# Module 8 — React Fundamentals: Components, JSX, Props & State

> **Difficulty:** ⭐⭐☆☆☆
> **Interview Frequency:** ⭐⭐⭐⭐⭐
> **Importance for React Developers:** ⭐⭐⭐⭐⭐

---

# Phase 3 Roadmap

Phase 3 covers the foundations you must be able to explain confidently in an interview. It is split across notes rather than dumped into one lesson.

```text
Covered in THIS note
 1. What is React?
 2. Imperative vs Declarative
 3. Component-based architecture
 4. Function components
 5. JSX
 6. JSX rules
 7. Props
 8. Props are read-only
 9. State
10. Props vs State
11. Component composition
12. Conditional rendering
13. Rendering lists and keys

Coming in the next Phase 3 notes
14. Controlled vs uncontrolled components
15. React rendering basics
16. Virtual DOM
17. Reconciliation
18. Common React mistakes
```

---

# Table of Contents

```text
1.  What is React?
2.  Imperative vs Declarative
3.  Component-Based Architecture
4.  Function Components
5.  JSX
6.  JSX Rules
7.  Props
8.  Props Are Read-Only
9.  State
10. Props vs State
11. Component Composition
12. Conditional Rendering
13. Rendering Lists
14. Why React Needs `key`
15. The Big Picture
16. Common Interview Questions
17. Senior-Level Summary
18. Practice Questions
```

---

# What is React?

**React is a JavaScript library for building user interfaces out of reusable, composable components.**

It was originally developed at Facebook (now Meta).

The important words in that definition:

> **JavaScript + UI + Components + Declarative**

## Why React?

Before libraries like React, you manipulated the DOM directly:

```javascript
const button = document.querySelector("#counter");

button.addEventListener("click", () => {
    button.textContent = "Clicked";
});
```

You explicitly tell the browser:

> Find this element → change its content.

React takes a different approach. You describe **what the UI should look like for the current state**:

```jsx
function Counter() {
    const [count, setCount] = useState(0);

    return (
        <button onClick={() => setCount(count + 1)}>
            Count: {count}
        </button>
    );
}
```

You are not saying "find the button and modify its text." You are saying:

> Given `count`, the UI looks like this.

React figures out the DOM operations needed to make that true.

## Important Interview Point

React is a **library**, not a framework. It solves the view layer. Routing, data fetching, forms, and build tooling come from separate packages you choose (React Router, TanStack Query, React Hook Form, Vite). Next.js is the framework built *on top of* React that bundles those decisions for you.

That distinction is a common follow-up question, and "library, because it only owns rendering — everything else is a choice you make" is the answer interviewers are listening for.

---

# Imperative vs Declarative

## Imperative

You tell the program **how** to do something — every step.

```javascript
const element = document.querySelector("#message");

element.textContent = "Hello";
element.style.color = "red";
```

You control the sequence of mutations.

## Declarative

You describe **what** you want for a given input.

```jsx
function Message({ isError }) {
    return (
        <p className={isError ? "error" : "success"}>
            Hello
        </p>
    );
}
```

React determines the necessary DOM changes.

## Interview Question — *Why is React called declarative?*

A strong answer:

> "React is declarative because instead of manually manipulating the DOM and describing every step required to update the UI, we describe what the UI should look like for a given state. When that state changes, React works out the minimal set of DOM updates needed."

That is much better than:

> "React automatically updates the DOM."

The second answer describes an effect. The first describes the model — and the model is what the interviewer is testing.

The mental shorthand worth remembering:

```text
UI = f(state)
```

---

# Component-Based Architecture

A component is an independent, reusable piece of UI.

Imagine an e-commerce application:

```text
App
│
├── Header
│   ├── Logo
│   └── Navigation
│
├── ProductList
│   ├── ProductCard
│   ├── ProductCard
│   └── ProductCard
│
└── Footer
```

Instead of one enormous component, the application is divided into small ones.

```jsx
function ProductCard() {
    return (
        <article>
            <h2>MacBook Pro</h2>
            <p>€2000</p>
            <button>Add to Cart</button>
        </article>
    );
}
```

Then reused:

```jsx
function ProductList() {
    return (
        <>
            <ProductCard />
            <ProductCard />
            <ProductCard />
        </>
    );
}
```

## Benefits

* Reusability
* Maintainability
* Separation of concerns
* Easier testing — a component is a function you can render in isolation
* Easier debugging — a bug lives inside one boundary
* Better collaboration — two people can work on sibling components without conflicts
* Smaller units of logic

## Important Interview Point

The benefit interviewers care about most is **isolation of change**. If a bug appears in the price display, you know it lives in `ProductCard` — not somewhere in a 2000-line file that touches the whole page. Reusability is the answer everyone gives; isolation is the answer that sounds like experience.

---

# Function Components

Modern React uses **function components**.

```jsx
function Welcome() {
    return <h1>Hello Rakesh</h1>;
}
```

Or:

```jsx
const Welcome = () => {
    return <h1>Hello Rakesh</h1>;
};
```

A component:

1. Is a function.
2. Starts with an uppercase letter.
3. Returns JSX (or `null`, a string, a number, or an array).
4. Can receive props.
5. Can hold state and use hooks.

Class components still exist and still work, but new code is written with functions — hooks only work inside function components.

## Why uppercase?

This is not a style convention. It changes what the JSX compiles to.

```jsx
<welcome />   // compiles to  jsx("welcome", {})   → a DOM tag named "welcome"
<Welcome />   // compiles to  jsx(Welcome, {})     → a reference to your function
```

A lowercase tag becomes a **string**, and React treats strings as host (DOM) elements. A capitalized tag becomes a **variable reference**, so React calls your function.

Get this wrong and React renders an unknown `<welcome>` element into the DOM with no error — one of the quieter beginner bugs.

Dot notation is the exception: `<Layout.Header />` is always treated as a component, regardless of case.

---

# JSX

JSX stands for **JavaScript XML**. It lets you write HTML-like syntax inside JavaScript.

```jsx
const element = <h1>Hello World</h1>;
```

**JSX is not HTML, and it is not part of JavaScript.** It is syntax that a compiler (Babel, SWC, esbuild, TypeScript) transforms into ordinary function calls before the browser ever sees it.

## How JSX compiles

There are two transforms, and knowing both is a good interview signal.

**Classic transform** (older setups, `React` must be in scope):

```jsx
<h1 className="title">Hello World</h1>
```

becomes:

```javascript
React.createElement("h1", { className: "title" }, "Hello World");
```

**Automatic transform** (the default since React 17 — the compiler injects the import itself):

```javascript
import { jsx as _jsx } from "react/jsx-runtime";

_jsx("h1", { className: "title", children: "Hello World" });
```

This is why modern React code does not need `import React from "react"` just to write JSX.

## What that call returns

Not a DOM node — a plain JavaScript object describing what should be rendered:

```jsx
console.log(<h1 className="title">Hello World</h1>);
```

Output:

```text
{
    type: "h1",
    key: null,
    props: { className: "title", children: "Hello World" },
    ...internal fields
}
```

That object is a **React element**: a lightweight description of UI. React builds a tree of these on every render and compares it against the previous tree to decide what to touch in the real DOM. (That comparison is reconciliation — covered in a later Phase 3 note.)

## Interview Question — *Is JSX HTML?*

> "No. JSX is syntactic sugar that compiles to function calls — `React.createElement`, or `jsx()` from the automatic runtime. Those calls return plain objects describing the UI, and React uses those objects to decide what to render. You can write React without JSX; it's just far less readable."

---

# JSX Rules

## Rule 1 — A JSX expression needs a single root

This will not compile:

```jsx
return (
    <h1>Hello</h1>
    <p>World</p>
);
```

Two adjacent elements are two expressions, and a `return` takes one value.

Wrap them:

```jsx
return (
    <div>
        <h1>Hello</h1>
        <p>World</p>
    </div>
);
```

Or use a **Fragment**, which groups children without adding a DOM node:

```jsx
return (
    <>
        <h1>Hello</h1>
        <p>World</p>
    </>
);
```

Output — the `<div>` version:

```html
<div>
    <h1>Hello</h1>
    <p>World</p>
</div>
```

Output — the Fragment version:

```html
<h1>Hello</h1>
<p>World</p>
```

That difference matters in flex and grid layouts, and inside `<table>` and `<ul>`, where a stray wrapper `<div>` breaks the layout or produces invalid HTML.

If you need a `key` on a fragment (rendering a list of pairs, for example), the shorthand cannot take one — use the long form:

```jsx
{rows.map(row => (
    <React.Fragment key={row.id}>
        <dt>{row.term}</dt>
        <dd>{row.definition}</dd>
    </React.Fragment>
))}
```

The single-root rule applies to a JSX *expression*. A component itself may also return `null`, a string, a number, or an array of elements.

---

## Rule 2 — JavaScript goes inside `{}`

```jsx
const name = "Rakesh";

return <h1>Hello {name}</h1>;
```

Anything that **evaluates to a value** works:

```jsx
<h1>{2 + 2}</h1>
<p>{user.name}</p>
<p>{isLoggedIn ? "Logout" : "Login"}</p>
<p>{items.length > 0 && "You have items"}</p>
```

**Statements do not.** `if`, `for`, and `switch` produce no value, so this is a syntax error:

```jsx
return (
    <div>
        {if (isLoggedIn) {
            "Welcome"
        }}
    </div>
);
```

Use an expression instead:

```jsx
return (
    <div>
        {isLoggedIn && <p>Welcome</p>}
    </div>
);
```

Or move the `if` above the `return`, where statements are legal:

```jsx
function Greeting({ isLoggedIn }) {
    let message = null;

    if (isLoggedIn) {
        message = <p>Welcome</p>;
    }

    return <div>{message}</div>;
}
```

## Important Interview Point — what React renders and what it skips

Inside `{}`:

| Value | Rendered output |
| ----- | --------------- |
| `"text"` | the text |
| `42` | `42` |
| `0` | **`0`** — a visible zero |
| `true` / `false` | nothing |
| `null` / `undefined` | nothing |
| `NaN` | `NaN` |
| array | each item rendered in order |
| object | **throws** — "Objects are not valid as a React child" |

The `0` row is the one that bites people. See the `&&` pitfall in [Conditional Rendering](#conditional-rendering) below.

---

## Rule 3 — Use `className`, not `class`

HTML:

```html
<div class="container">
```

JSX:

```jsx
<div className="container">
```

Why: JSX compiles to JavaScript, and `class` is a reserved word. Same reason `for` on a `<label>` becomes `htmlFor`.

---

## Rule 4 — Attributes are camelCase

HTML:

```html
<button onclick="" tabindex="">
```

JSX:

```jsx
<button onClick={handleClick} tabIndex={0}>
```

Exceptions worth knowing: `data-*` and `aria-*` attributes keep their dashes.

```jsx
<div data-testid="cart" aria-label="Shopping cart" />
```

## Rule 5 — Every tag must close

```jsx
<img src={url} alt="" />
<br />
<input type="text" />
```

Self-closing tags are optional in HTML. In JSX they are mandatory — it is XML syntax.

---

# Props

**Props are data passed from a parent component to a child component.** Think of them as a component's inputs — the arguments to the function.

```jsx
function User({ name }) {
    return <h2>Hello {name}</h2>;
}
```

Parent:

```jsx
function App() {
    return <User name="Rakesh" />;
}
```

Data flows one way:

```text
App
 │
 │ name="Rakesh"
 ▼
User
```

Output:

```html
<h2>Hello Rakesh</h2>
```

## Multiple props

```jsx
function User({ name, age, role }) {
    return (
        <div>
            <h2>{name}</h2>
            <p>{age}</p>
            <p>{role}</p>
        </div>
    );
}
```

Usage:

```jsx
<User
    name="Rakesh"
    age={32}
    role="Frontend Developer"
/>
```

Note the difference:

```jsx
name="Rakesh"   // a string literal
age={32}        // a JS expression — here, the number 32
age="32"        // the STRING "32" — a real source of bugs
```

## Props arrive as one object

`props` is a single object; the destructuring in the parameter list is just [ES6 destructuring](../javascript/es6-modern-js.md) applied to it.

```jsx
function User(props) {
    return <h2>{props.name}</h2>;
}

// identical:
function User({ name }) {
    return <h2>{name}</h2>;
}
```

## Default values

Use a destructuring default:

```jsx
function Button({ variant = "primary", disabled = false, children }) {
    return (
        <button className={variant} disabled={disabled}>
            {children}
        </button>
    );
}
```

The legacy `Button.defaultProps = { ... }` API is removed for function components in React 19 — destructuring defaults are the current way.

## Spreading props

```jsx
const config = { name: "Rakesh", age: 32 };

<User {...config} />
```

This is the [spread operator](../javascript/es6-modern-js.md) copying each key onto the element's props. Convenient for passthrough wrappers:

```jsx
function Input({ label, ...rest }) {
    return (
        <label>
            {label}
            <input {...rest} />
        </label>
    );
}
```

Use it deliberately. Blindly spreading unknown props makes it hard to see what a component actually accepts, and it can leak invalid attributes into the DOM.

---

# Props Are Read-Only

This is extremely important.

A child must **never modify its props**.

```jsx
function User(props) {
    props.name = "John";   // ❌ never do this
}
```

Two reasons:

1. **It does nothing useful.** The value lives in the parent. Mutating the local copy does not tell the parent anything and does not schedule a re-render — on the next render, the parent passes the original value again and your change vanishes.
2. **React actively prevents it in development.** React freezes the props object in dev builds, so the assignment throws a `TypeError` (module and JSX code always runs in strict mode).

React's data flow is **one-way**: data goes down, events go up. Props being read-only is what makes that flow predictable — you can always find where a value came from by walking up the tree.

## How a child asks for a change

The parent passes a **callback**.

Parent:

```jsx
function Parent() {
    const handleDelete = () => {
        console.log("Delete user");
    };

    return <User onDelete={handleDelete} />;
}
```

Child:

```jsx
function User({ onDelete }) {
    return (
        <button onClick={onDelete}>
            Delete
        </button>
    );
}
```

The child does not touch the parent's data. It reports an event, and the parent decides what to do. This pattern is called **lifting state up**, and it is the answer to "how does a child communicate with a parent?"

---

# State

**State is data owned by a component that can change over time, and whose change triggers a re-render.**

```jsx
import { useState } from "react";

function Counter() {
    const [count, setCount] = useState(0);

    return (
        <button onClick={() => setCount(count + 1)}>
            {count}
        </button>
    );
}
```

* `count` — the current value for this render.
* `setCount` — the updater. Calling it tells React "this value changed, schedule a re-render."
* `0` — the initial value, used only on the first render.

## Why not just a normal variable?

```jsx
function BrokenCounter() {
    let count = 0;   // ❌

    return (
        <button onClick={() => { count++; }}>
            {count}
        </button>
    );
}
```

This is broken twice over:

1. React does not know anything changed, so nothing re-renders.
2. Even if something else caused a re-render, the function runs again from the top and `count` resets to `0`.

State exists to solve exactly those two problems: React **remembers** it between renders, and **watching** it is what schedules the re-render.

## Interview Question — *When should something be state?*

Something should be state only if all three hold:

* it changes over time, **and**
* the change should be visible in the UI, **and**
* it cannot be computed from existing props or state.

If it can be derived, derive it during render instead of storing it:

```jsx
// ❌ redundant state that can drift out of sync
const [items, setItems] = useState([]);
const [total, setTotal] = useState(0);

// ✅ derived on every render — always correct
const [items, setItems] = useState([]);
const total = items.reduce((sum, item) => sum + item.price, 0);
```

Values that change but are *not* rendered — a timer id, a DOM node — belong in a ref, not state. That is Phase 4.

## Why you must never mutate state directly

```jsx
const [user, setUser] = useState({ name: "Rakesh", age: 32 });

// ❌
user.age = 33;
setUser(user);

// ✅
setUser({ ...user, age: 33 });
```

React decides whether to re-render by comparing the **reference** (`Object.is`), not by inspecting the contents. Mutating keeps the same reference, so `setUser(user)` looks like "nothing changed" and React can bail out of the re-render entirely.

This is the direct payoff of [Module 2 — Data Types & Memory](../javascript/data-types.md) and [Module 4 — Objects & Arrays](../javascript/objects-arrays.md): objects and arrays are held by reference, so a new state value means a **new object**.

The same rule for arrays — use the non-mutating methods:

```jsx
setItems([...items, newItem]);                          // add
setItems(items.filter(item => item.id !== id));         // remove
setItems(items.map(item =>                              // update one
    item.id === id ? { ...item, done: true } : item
));
```

`push`, `splice`, `sort`, and `reverse` mutate — they will not reliably trigger a render.

## State updates are asynchronous and batched

```jsx
function handleClick() {
    setCount(count + 1);
    setCount(count + 1);
    setCount(count + 1);
    console.log(count);   // still the OLD value
}
```

Output, starting from `count === 0`:

```text
0
```

and `count` ends up as **1**, not 3.

`count` is a constant captured by this render — calling the setter does not reassign it. All three calls compute `0 + 1`. React batches the updates and re-renders once.

When the next value depends on the previous one, use the **updater function**:

```jsx
setCount(prev => prev + 1);
setCount(prev => prev + 1);
setCount(prev => prev + 1);
```

Now `count` ends up as **3** — React applies the functions in order against the latest value.

This is a closure over a render's variables, and it is the same mechanism as the stale closure problem in [Module 3 — Functions & Closures](../javascript/functions-closures.md). Interviewers ask this constantly.

---

# Props vs State

An interview-critical comparison.

| Props | State |
| ----- | ----- |
| Passed in from the parent | Declared and managed inside the component |
| Read-only for the receiving component | Updated via its setter |
| Configure a component | Represent changing data |
| External input | Internal data |
| The parent owns the value | The component owns the value |
| A new value from the parent re-renders the child | An update re-renders the component and its subtree |

Example:

```jsx
function User({ name }) {
    const [isOnline, setIsOnline] = useState(false);

    return (
        <div>
            <h2>{name}</h2>
            <p>{isOnline ? "Online" : "Offline"}</p>
            <button onClick={() => setIsOnline(!isOnline)}>
                Toggle
            </button>
        </div>
    );
}
```

```text
name       → prop   (owned by the parent)
isOnline   → state  (owned by User)
```

## Important Interview Point

One component's state is very often another component's props. When the parent's state changes, it re-renders and passes the new value down:

```jsx
function App() {
    const [theme, setTheme] = useState("dark");

    return <Header theme={theme} />;   // App's state, Header's prop
}
```

So "props don't change" is imprecise, and saying it will get you corrected. A component cannot change *its own* props — but the parent can hand it a different value on the next render.

## Strong interview answer

> "Props are read-only inputs passed from a parent to a child; state is data a component owns and can change over time. A component can't modify its own props, but it re-renders when the parent passes new ones. State is for data the component itself needs to track and react to. And one component's state is frequently the prop it passes to its children."

---

# Component Composition

Rather than building components with more and more configuration flags, React encourages **composition** — passing UI in as content.

```jsx
function Card({ children }) {
    return (
        <div className="card">
            {children}
        </div>
    );
}
```

Usage:

```jsx
<Card>
    <h2>Product</h2>
    <p>€100</p>
    <button>Buy</button>
</Card>
```

Output:

```html
<div class="card">
    <h2>Product</h2>
    <p>€100</p>
    <button>Buy</button>
</div>
```

`children` is a **special prop** holding whatever sits between the opening and closing tags. It is not magic — this is equivalent:

```jsx
<Card children={<h2>Product</h2>} />
```

## Why composition beats configuration

Watch what happens without it:

```jsx
// ❌ every new requirement adds another flag
<Card
    title="Product"
    showPrice={true}
    price={100}
    showBuyButton={true}
    buttonLabel="Buy"
    showBadge={false}
/>
```

Every variation the design team invents means another prop and another branch inside `Card`. With composition, `Card` only owns the frame — the caller supplies the contents, and `Card` never changes again.

## Multiple slots

`children` is one slot. For several, pass elements as named props:

```jsx
function Layout({ sidebar, content }) {
    return (
        <div className="layout">
            <aside>{sidebar}</aside>
            <main>{content}</main>
        </div>
    );
}

<Layout
    sidebar={<Navigation />}
    content={<ProductList />}
/>
```

## Important Interview Point

Composition is also the cheapest fix for **prop drilling** — threading a prop through layers of components that don't use it. Instead of passing `user` down four levels, render the element that needs `user` at the top and pass it as `children`. Reach for Context only when composition genuinely cannot reach the consumer (Phase 6).

---

# Conditional Rendering

## Ternary — when there are two branches

```jsx
{isLoggedIn ? <Dashboard /> : <Login />}
```

## `&&` — when there is one branch

```jsx
{isAdmin && <AdminPanel />}
```

## Early return — usually the cleanest

```jsx
function Dashboard({ user }) {
    if (!user) {
        return <Login />;
    }

    return <DashboardContent user={user} />;
}
```

Early returns keep the happy path unindented, and they let you handle loading and error states without nesting three ternaries. Returning `null` renders nothing:

```jsx
function Banner({ message }) {
    if (!message) return null;

    return <div className="banner">{message}</div>;
}
```

## Important Interview Point — the `&&` zero trap

This is the single most common React rendering bug, and a favourite interview question.

```jsx
{cart.length && <Cart items={cart} />}
```

With an empty cart, `cart.length` is `0`. `&&` returns the falsy left operand — `0` — and React **renders `0`** as text. A stray zero appears on the page.

Fixes:

```jsx
{cart.length > 0 && <Cart items={cart} />}     // coerce to a boolean
{cart.length ? <Cart items={cart} /> : null}   // ternary
{Boolean(cart.length) && <Cart items={cart} />}
```

The rule: **only ever put a real boolean on the left of `&&` in JSX.** `false`, `null`, and `undefined` render nothing; `0` and `NaN` render themselves.

---

# Rendering Lists

Given:

```javascript
const users = [
    { id: 1, name: "Rakesh" },
    { id: 2, name: "John" },
    { id: 3, name: "Sarah" }
];
```

Render with `map`:

```jsx
function UserList() {
    return (
        <ul>
            {users.map(user => (
                <li key={user.id}>
                    {user.name}
                </li>
            ))}
        </ul>
    );
}
```

Output:

```html
<ul>
    <li>Rakesh</li>
    <li>John</li>
    <li>Sarah</li>
</ul>
```

`map` is used rather than `forEach` because JSX needs a **returned array** of elements — `forEach` returns `undefined`. This is [Module 4](../javascript/objects-arrays.md) put to work.

Note that `key` does not appear in the output. It is consumed by React, never rendered, and never passed to the child component:

```jsx
function Row({ key, name }) {   // ❌ key is always undefined here
    return <li>{name}</li>;
}
```

If the child needs the value, pass it twice: `<Row key={user.id} id={user.id} />`.

---

# Why React Needs `key`

A very common interview question.

The `key` tells React **which item is which** between two renders, so it can tell "changed" apart from "moved," "added," and "removed."

```text
Before:        After:

A              A
B              C
C              D
```

With keys, React sees:

```text
A → same, keep the DOM node
B → removed, unmount it
C → same, keep the DOM node (moved)
D → added, mount it
```

Without stable keys, React falls back to comparing by position and would instead see "the second item's text changed from B to C, the third from C to D" — more DOM work, and any state inside those items ends up attached to the wrong row.

## Good

```jsx
<li key={user.id}>
```

A stable id that belongs to the **data**, not the render.

## Usually bad

```jsx
<li key={index}>
```

The index describes a position, not an item. Insert at the top of the list and every index shifts — React now believes every item changed.

## What actually breaks

Index keys are only visibly harmful when list items hold state or DOM state. Consider a list of inputs:

```jsx
{items.map((item, index) => (
    <input key={index} defaultValue={item.name} />
))}
```

Type into the first input, then prepend a new item. React matches the new item to `key=0` — the input you typed into — so your text stays on the wrong row, and focus jumps. With `key={item.id}`, each input follows its item.

## When index keys are acceptable

All three must hold:

* the list is never reordered, filtered, or inserted into (append-only or fully static), **and**
* the items have no state, and no uncontrolled DOM state such as inputs, **and**
* the items have no genuinely unique id available.

## Two more rules

* Keys must be unique **among siblings**, not globally. Two different lists can both use `key={1}`.
* Never use `Math.random()` or `Date.now()` as a key. A fresh key on every render means React unmounts and remounts every item, every time — the worst possible outcome, and it destroys state and focus.

## Strong interview answer

> "Keys give React a stable identity for each item in a list so it can match elements between renders instead of comparing by position. With good keys it can move, keep, or remove DOM nodes correctly. Index keys tie identity to position, so any reorder or insert makes React think the content changed — which causes extra DOM work and, more visibly, leaves component state and input focus attached to the wrong item."

---

# The Big Picture

The mental model to walk into the interview with:

```text
                 React Application
                        │
                        ▼
                   Components
                        │
             ┌──────────┴──────────┐
             ▼                     ▼
           Props                  State
             │                     │
      Parent → Child          Component manages
             │                     │
             └──────────┬──────────┘
                        ▼
                      JSX
                        │
                        ▼
                      React
                        │
                        ▼
                       UI
```

Compressed to one line:

**React → Components → JSX → Props → State → Rendering**

And to one equation:

```text
UI = f(props, state)
```

---

# Common Interview Questions

Don't memorize answers. Aim to explain each one out loud in **1–3 minutes**.

### Fundamentals

* What is React?
* Is React a library or a framework?
* Why is React called declarative?
* What are the advantages of component-based architecture?
* What is JSX?
* Is JSX HTML?
* How does JSX get converted to JavaScript?
* What does `React.createElement` return?
* What is a function component?
* Why must component names start with an uppercase letter?
* What is a Fragment, and when do you need one?

### Props

* What are props?
* Are props mutable?
* Can a child modify its parent's props?
* How does a child communicate with a parent?
* What is the `children` prop?
* How do you give a prop a default value?

### State

* What is state?
* Why can't you use a plain variable instead of state?
* Why shouldn't you modify state directly?
* What happens when state changes?
* Why does calling `setCount(count + 1)` three times only increment by one?
* What is the difference between props and state?
* When should something be state versus a derived value?

### Rendering

* What is conditional rendering?
* How do you render a list in React?
* Why does React need keys?
* Why is using the array index as a key sometimes problematic?
* When is an index key actually fine?
* What is component composition, and how does it help with prop drilling?

---

# Senior-Level Summary

By the end of this module, you should be able to explain:

* React is a **library** for the view layer; you describe the UI for a given state and React works out the DOM updates. `UI = f(state)`.
* Declarative means describing the target UI, not the steps to mutate the DOM.
* Components are functions that take props and return React elements; composing them isolates change.
* JSX is not HTML — it compiles to `React.createElement` (classic) or `jsx()` from `react/jsx-runtime` (automatic), and those calls return **plain objects**, not DOM nodes.
* A JSX expression needs one root; `{}` accepts expressions, not statements; `class` is `className` because JSX is JavaScript.
* Props are read-only inputs flowing parent → child; events flow child → parent through callbacks. React freezes props in development.
* State is data the component owns; React remembers it across renders, and updating it schedules a re-render.
* State must be replaced, not mutated — React compares by reference, so mutation can silently skip the re-render.
* State updates are batched and asynchronous within an event handler; use the updater form when the next value depends on the previous one.
* A component cannot change its own props, but a parent re-render can hand it new ones — one component's state is another's props.
* Composition (`children`, element props) scales better than adding configuration flags, and it solves most prop drilling without Context.
* `&&` in JSX renders `0` — put a real boolean on the left.
* Keys give list items a stable identity; index keys tie identity to position and misplace state, focus, and DOM nodes on reorder.

---

# Practice Questions

Answer these without running them.

### 1. What renders, and why?

```jsx
function Cart({ items }) {
    return (
        <div>
            {items.length && <p>{items.length} items</p>}
        </div>
    );
}

<Cart items={[]} />
```

---

### 2. What is the final value of `count` after one click, and why?

```jsx
function Counter() {
    const [count, setCount] = useState(0);

    function handleClick() {
        setCount(count + 1);
        setCount(count + 2);
    }

    return <button onClick={handleClick}>{count}</button>;
}
```

How would you rewrite `handleClick` so it adds 3?

---

### 3. Why does the name never update on screen?

```jsx
function Profile() {
    const [user, setUser] = useState({ name: "Rakesh" });

    function rename() {
        user.name = "John";
        setUser(user);
    }

    return (
        <div>
            <h2>{user.name}</h2>
            <button onClick={rename}>Rename</button>
        </div>
    );
}
```

---

### 4. What does this render into the DOM, and what is wrong with it?

```jsx
function App() {
    return <welcome name="Rakesh" />;
}
```

---

### 5. Predict the bug

A list of todos, each with an editable input, keyed by index. The user types into the third input, then deletes the first todo. What does the user see, and why?

---

### 6. Refactor for composition

```jsx
<Modal
    title="Delete user"
    body="This cannot be undone."
    showCancel={true}
    cancelLabel="Cancel"
    showConfirm={true}
    confirmLabel="Delete"
    confirmVariant="danger"
/>
```

Rewrite `Modal` so that adding a new footer button never requires changing `Modal` itself.

---

# Coding Practice

Build these small. No libraries, no styling — the point is the data flow.

1. **Counter** — increment, decrement, reset. Then make a "+3" button that works correctly with a single click.
2. **User card list** — an array of users rendered with `map` and proper keys, each card built with props only.
3. **Toggle list** — a list where each row has its own "online/offline" state. Decide deliberately whether that state lives in the row or the parent, and be able to justify it.
4. **Reusable `<Card>`** — frame only, contents passed as `children`. Use it for three visually different cards without adding a single prop to `Card`.
5. **Filtered list** — one search input in the parent, the filtered results passed down as props. Nothing derived should be stored in state.

---

| ← Previous | Index | Next → |
| ---------- | ----- | ------ |
| [Module 7 — ES6+ & Modern JavaScript](../javascript/es6-modern-js.md) | [Study Plan](../README.md) | [Module 9 — React Hooks](react-hooks.md) |
