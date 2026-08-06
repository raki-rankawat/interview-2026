[← Study Plan](../../README.md)

# Module 3 — Functions, Execution Context & Closures

---

# 1. Functions in JavaScript

## Definition

A **function** is a reusable block of code designed to perform a specific task.

A function can:

* Receive input (**parameters**)
* Execute logic
* Return output (**return value**)

Example:

```javascript
function add(a, b) {
    return a + b;
}

const result = add(5, 3);

console.log(result);
```

Output:

```
8
```

Here:

* `a` and `b` are parameters.
* `5` and `3` are arguments.
* The function returns a value.

---

# Why Do Functions Exist?

Without functions:

```javascript
console.log("Hello Rocky");
console.log("Hello Alex");
console.log("Hello John");
```

You repeat code.

With functions:

```javascript
function greet(name) {
    console.log(`Hello ${name}`);
}

greet("Rocky");
greet("Alex");
greet("John");
```

Benefits:

* Reusability
* Maintainability
* Testing
* Separation of responsibilities

---

# 2. Function Declaration

## Definition

A function declaration creates a named function using the `function` keyword.

Example:

```javascript
function calculatePrice(price, tax) {
    return price + tax;
}
```

---

## Hoisting Behavior

Function declarations are fully hoisted.

This works:

```javascript
sayHello();

function sayHello() {
    console.log("Hello");
}
```

Output:

```
Hello
```

Why?

During the creation phase, JavaScript stores the entire function in memory — not just the name, but the whole function body.

---

## Important Interview Point

"Fully hoisted" means *within its own scope*. A function declaration inside a block does not escape that block in strict mode (which includes all ES modules and all `class` bodies):

```javascript
"use strict";

if (true) {
    function blockFn() {
        return "hi";
    }
}

blockFn();
```

Output:

```
ReferenceError: blockFn is not defined
```

The safe habit: declare functions at the top level of a scope, not inside `if` blocks.

---

# 3. Function Expression

## Definition

A function expression stores a function inside a variable.

Example:

```javascript
const greet = function () {
    console.log("Hello");
};
```

The function itself is anonymous.

---

## Hoisting Difference

This fails:

```javascript
sayHello();

const sayHello = function () {
    console.log("Hello");
};
```

Error:

```
ReferenceError: Cannot access 'sayHello' before initialization
```

Why?

Because only the variable declaration is hoisted, not the function assignment. `sayHello` is in its Temporal Dead Zone (see [Module 1 — Variables](variables.md)).

---

## Important Interview Point

The keyword changes the error you get, and interviewers use this to check whether you actually understand hoisting or just memorised a rule:

```javascript
sayHello();          // TypeError: sayHello is not a function

var sayHello = function () {
    console.log("Hello");
};
```

* `var` → hoisted **and** initialized to `undefined`, so the name exists. You then try to call `undefined`, which is a **`TypeError`**.
* `let` / `const` → hoisted but **uninitialized**, so touching the name at all is a **`ReferenceError`**.
* Function declaration → the whole function is in memory. It just works.

Same root cause, three different outcomes.

---

# 4. Arrow Functions

Introduced in ES6.

Example:

```javascript
const add = (a, b) => {
    return a + b;
};
```

Short version (implicit return):

```javascript
const add = (a, b) => a + b;
```

To implicitly return an object literal, wrap it in parentheses — otherwise the braces are read as a function body:

```javascript
const makeUser = (name) => ({ name });
```

---

# Arrow Function Differences

Arrow functions are not just shorter syntax.

They behave differently.

---

## 1. No own `this`

Regular function:

```javascript
const user = {

    name: "Rocky",

    sayName: function () {
        console.log(this.name);
    }

};

user.sayName();
```

Output:

```
Rocky
```

Because `this` is determined by **how the function is called** — here it's called as a method of `user`, so `this` is `user`.

---

Arrow function:

```javascript
const user = {

    name: "Rocky",

    sayName: () => {
        console.log(this.name);
    }

};

user.sayName();
```

An arrow function has no `this` of its own. It inherits `this` from the scope where it was **defined** — and an object literal is not a scope, so it reaches past `user` to whatever `this` is at the top level of the file.

---

## Important Interview Point

What this actually prints depends on where the file runs:

| Environment | Top-level `this` | Result |
| ----------- | ---------------- | ------ |
| Browser, classic `<script>` | `window` | Logs `""` — an empty string |
| ES module (`<script type="module">`, `.mjs`) | `undefined` | `TypeError: Cannot read properties of undefined (reading 'name')` |
| Node.js CommonJS file | `module.exports` (`{}`) | Logs `undefined` |

The common shorthand answer is "it logs `undefined`," and that is true in Node. But in a browser script it logs an **empty string**, because `window.name` is a real built-in property — the name of the browsing context — and it defaults to `""`. It is not undefined; it is a genuine string that happens to be empty.

The point that matters in all three cases is the same: **the arrow never binds `this` to `user`.** Knowing *why* the browser prints something different is what separates a memorised answer from an understood one.

---

## When lexical `this` is exactly what you want

Arrows are not a defect to work around. Before ES6, this was the classic bug:

```javascript
const timer = {

    seconds: 0,

    start: function () {
        setInterval(function () {
            this.seconds++;      // `this` is NOT `timer`
        }, 1000);
    }

};
```

The callback is invoked by the timer, not as a method, so `this` is the global object (or `undefined` in strict mode). People used to fix it with `const self = this;` or `.bind(this)`.

With an arrow, the callback keeps the surrounding `this`:

```javascript
const timer = {

    seconds: 0,

    start: function () {
        setInterval(() => {
            this.seconds++;      // `this` IS `timer`
        }, 1000);
    }

};
```

Rule of thumb: **method → regular function. Callback inside a method → arrow.**

---

## 2. Cannot be used as constructors

Regular function:

```javascript
function Person(name) {
    this.name = name;
}

const user = new Person("Rocky");
```

Works.

Arrow:

```javascript
const Person = (name) => {
    this.name = name;
};

new Person("Rocky");
```

Error:

```
TypeError: Person is not a constructor
```

Arrow functions have no `[[Construct]]` internal method and no `prototype` property, so `new` has nothing to work with. They also have no `super` and no `new.target`.

---

## 3. No `arguments` object

Regular:

```javascript
function test() {
    console.log(arguments);
}

test(1, 2, 3);
```

Output:

```
[Arguments] { '0': 1, '1': 2, '2': 3 }
```

Arrow:

```javascript
const test = () => {
    console.log(arguments);
};

test(1, 2, 3);
```

Output:

```
ReferenceError: arguments is not defined
```

---

## Important Interview Point

An arrow doesn't *block* `arguments` — it simply has none of its own, so the name resolves through the scope chain exactly like any other variable. Nested inside a regular function, an arrow sees that **outer function's** `arguments`:

```javascript
function outer() {
    const inner = () => console.log(arguments);
    inner("ignored");
}

outer(1, 2, 3);
```

Output:

```
[Arguments] { '0': 1, '1': 2, '2': 3 }
```

It logged `outer`'s arguments, not `inner`'s. This surprises people.

In modern code, use rest parameters instead — they work in both function types and give you a real array:

```javascript
const test = (...args) => {
    console.log(args);       // [1, 2, 3] — a real array
};
```

---

# 5. First-Class Functions

## Definition

JavaScript treats functions as **first-class citizens**.

Meaning:

Functions can be:

1. Stored in variables.
2. Passed as arguments.
3. Returned from other functions.

A function in JavaScript is an object — it can hold properties, and it is passed around by reference like any other object.

---

## Stored in Variables

```javascript
const greet = function () {
    console.log("Hi");
};
```

---

## Passed as Arguments

Example:

```javascript
function execute(callback) {

    callback();

}

execute(function () {
    console.log("Running");
});
```

Output:

```
Running
```

This concept powers:

* Array methods
* Event handlers
* React callbacks

Example:

```javascript
button.addEventListener(
    "click",
    handleClick
);
```

`handleClick` is passed as a value.

Note the missing parentheses. `handleClick` passes the function; `handleClick()` would **call** it immediately and pass its return value instead — a very common bug, and the same mistake as `onClick={handleClick()}` in React.

---

## Returned from Functions

Example:

```javascript
function createMultiplier(x) {

    return function (y) {

        return x * y;

    };

}

const double = createMultiplier(2);

double(5);
```

Output:

```
10
```

This introduces our next topic.

---

# 6. Execution Context

## Definition

An **execution context** is the environment where JavaScript code is evaluated and executed.

Every time JavaScript runs code, it creates an execution context.

There are three main types:

1. Global Execution Context
2. Function Execution Context
3. Eval Execution Context (rare — `eval` is discouraged and effectively never used in production)

---

# Global Execution Context

Created when JavaScript starts running.

It contains:

* Global variables
* Global functions
* Global object

Example:

```javascript
var name = "Rocky";
```

This exists in the global context.

---

# Function Execution Context

Created whenever a function runs.

Example:

```javascript
function greet() {

    let message = "Hello";

}

greet();
```

When `greet()` executes:

A new execution context is created.

Called it three times? Three separate execution contexts, each with its own `message`. This independence is what makes closures work later on.

---

# Execution Context Phases

Every execution context has two phases.

---

# Phase 1: Creation Phase

JavaScript prepares memory.

It creates:

* Variables
* Functions
* Scope chain
* `this`

Example:

```javascript
console.log(age);

var age = 25;
```

During creation:

```
age = undefined
```

---

# Phase 2: Execution Phase

JavaScript runs code line by line.

Example:

```javascript
age = 25;
```

Now:

```
age = 25
```

---

# 7. Call Stack

## Definition

The call stack is a data structure that tracks function execution order.

JavaScript executes one thing at a time — it is **single-threaded**.

Example:

```javascript
function first() {

    second();

}

function second() {

    console.log("Hello");

}

first();
```

Call stack:

```
Global
 |
first()
 |
second()
```

After `second()` finishes:

```
Global
 |
first()
```

After `first()` finishes:

```
Global
```

Push on call, pop on return. Last in, first out.

---

## Important Interview Point

The stack is finite. Recursion with no base case fills it:

```javascript
function boom() {
    return boom();
}

boom();
```

Output:

```
RangeError: Maximum call stack size exceeded
```

This is also why one long synchronous task freezes the page: nothing else can run until the stack is empty. Async work (timers, fetch, promises) is what keeps the stack clear — that's the Event Loop, coming in a later module.

---

# 8. Scope Chain

## Definition

The scope chain determines where JavaScript looks for variables.

Example:

```javascript
const name = "Rocky";

function outer() {

    const age = 25;

    function inner() {

        console.log(name);
        console.log(age);

    }

    inner();

}

outer();
```

`inner()` searches:

1. Its own scope
2. Outer function scope
3. Global scope

If it isn't found anywhere, you get a `ReferenceError`.

---

## Lexical Scope

The chain is decided by **where the function is written in the source code**, not by where it is called from. That is what "lexical" means.

```javascript
const value = "global";

function printValue() {
    console.log(value);
}

function run() {
    const value = "local";
    printValue();
}

run();
```

Output:

```
global
```

`printValue` was *defined* next to the global `value`, so that is the one it sees — even though it was *called* from inside `run`. The scope chain is fixed when the function is created, not when it is invoked.

Contrast this with `this`, which **is** decided at call time. Scope is lexical; `this` is dynamic. Confusing the two is the root of most `this` bugs.

---

# 9. Closures

This is one of the most important JavaScript concepts.

---

# Definition

A closure is created when a function remembers variables from its **outer lexical scope**, even after the outer function has finished executing.

Simple explanation:

> A closure allows a function to "remember" the environment where it was created.

---

Example:

```javascript
function createCounter() {

    let count = 0;

    return function () {

        count++;

        console.log(count);

    };

}

const counter = createCounter();

counter();
counter();
counter();
```

Output:

```
1
2
3
```

---

## What happened?

When:

```javascript
createCounter()
```

runs:

Memory:

```
count = 0
```

It returns the inner function.

Normally, the outer function's execution context is popped off the call stack and its variables are discarded.

But because the inner function still references `count`, JavaScript keeps that variable alive.

This is closure.

---

## Each call creates a fresh closure

```javascript
const counterA = createCounter();
const counterB = createCounter();

counterA();   // 1
counterA();   // 2
counterB();   // 1
```

`counterB` starts at 1, not 3. Every call to `createCounter()` creates a new execution context with its own `count`. Closures capture a **scope**, not a snapshot value — and not a shared global.

---

## Closures give you private state

`count` cannot be reached from outside:

```javascript
const counter = createCounter();

counter.count;    // undefined
count;            // ReferenceError: count is not defined
```

The only way to change it is through the returned function. This is the module pattern, and it's how libraries created private variables long before `#private` class fields existed.

---

## Do closures leak memory?

Not by themselves. A closed-over variable stays in memory exactly as long as the closure that references it stays reachable. That's correct behaviour, not a leak.

It becomes a leak when you hold the closure longer than you meant to:

* An event listener that is never removed, capturing a large object.
* A `setInterval` that is never cleared.
* A subscription in a React `useEffect` with no cleanup function.

The fix is always the same: remove the listener, clear the interval, return a cleanup function. The closure and its captured scope are then collected normally.

---

# Real React Example

Consider:

```javascript
function Component() {

    const user = "Rocky";

    function handleClick() {

        console.log(user);

    }

    return <button onClick={handleClick}>Click</button>;

}
```

`handleClick` remembers `user`.

That is closure.

This is not an edge case — it is how every React component works. Each render creates new local variables and new function definitions that close over them. Props, state, and handlers are all captured by the closures defined in that render.

---

# Common React Problem: Stale Closure

Example:

```javascript
function Counter() {

    const [count, setCount] = useState(0);

    function handleClick() {

        setCount(count + 1);

        setTimeout(() => {
            console.log(count);
        }, 3000);

    }

    return <button onClick={handleClick}>{count}</button>;

}
```

Click once, then wait three seconds. The log shows `0`, not `1`.

Why?

That `handleClick` belongs to the render where `count` was `0`. Its `count` is a `const` captured by the closure for that render — it can never change. React re-renders and creates a *new* `handleClick` with `count = 1`, but the already-scheduled timeout is still holding the old one.

This is a **stale closure**: the function is looking at a value that was correct when it was created and is now out of date.

The functional update form avoids the same trap when setting state, because React hands you the current value instead of you reading a captured one:

```javascript
setCount(prev => prev + 1);
```

For reading a fresh value inside an async callback, a `useRef` holds a mutable box that is shared across renders rather than captured per render.

Missing dependencies in a `useEffect` array cause the identical bug: the effect keeps running the closure from the render where it was created.

---

# Interview Questions

## Beginner

1. What is a function?
2. Difference between function declaration and expression?
3. What are arrow functions?
4. Difference between arrow and normal functions?

---

## Intermediate

5. What is execution context?
6. What happens when a function runs?
7. Explain call stack.
8. What is lexical scope?

---

## Senior

9. Explain closures.
10. Where are closures used in React?
11. Explain stale closures.
12. How does JavaScript manage memory for closures?
13. Are closures bad for performance?

---

# Senior-Level Summary

You should be able to explain:

* Functions are first-class objects in JavaScript.
* Function declarations are hoisted differently from expressions — and the error you get (`TypeError` vs `ReferenceError`) tells you which one you're dealing with.
* Arrow functions have lexical `this`, no `arguments`, and cannot be constructed.
* Scope is lexical and fixed at definition; `this` is dynamic and decided at call time.
* JavaScript executes code using execution contexts.
* Each function call creates a new execution context.
* The call stack manages execution order, and it is single-threaded.
* Scope determines variable accessibility.
* Closures allow functions to preserve access to outer variables, giving you private state.
* Closures are not a memory leak on their own — uncleaned listeners and timers are.
* React hooks rely heavily on closures, which is exactly why stale closures happen.

---

# Practice Questions

Answer these:

### 1. What is the output?

```javascript
function test() {

    let x = 10;

    return function () {

        console.log(x);

    };

}

const fn = test();

fn();
```

---

### 2. What happens?

```javascript
const obj = {

    name: "Rocky",

    getName: () => {

        console.log(this.name);

    }

};

obj.getName();
```

Explain why — and explain why the answer is different in a browser `<script>`, an ES module, and a Node CommonJS file.

---

### 3. Predict output:

```javascript
function outer() {

    let count = 0;

    return function () {

        count++;

        console.log(count);

    };

}

const a = outer();

const b = outer();

a();
a();

b();
```

---

### 4. Explain:

Why does React's `useState` sometimes show an old value inside `setTimeout`?

---

### 5. Predict output:

```javascript
const value = "outer";

function show() {
    console.log(value);
}

function wrapper() {
    const value = "inner";
    show();
}

wrapper();
```

Which one wins, and what principle decides it?

---

| ← Previous | Index | Next → |
| ---------- | ----- | ------ |
| [Module 2 — Data Types & Memory](data-types.md) | [Study Plan](../../README.md) | Module 4 — Objects & Arrays *(not written yet)* |
