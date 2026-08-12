[← Study Plan](../README.md)

# Module 7 — ES6+ & Modern JavaScript (Complete Guide)

> **Difficulty:** ⭐⭐⭐⭐☆
> **Interview Frequency:** ⭐⭐⭐⭐⭐
> **Importance for React Developers:** ⭐⭐⭐⭐⭐

---

# Table of Contents

```text
1. Why ES6 Was Introduced
2. let vs const vs var (Review)
3. Template Literals
4. Destructuring
5. Spread Operator
6. Rest Operator
7. Default Parameters
8. Enhanced Object Literals
9. Computed Property Names
10. Optional Chaining
11. Nullish Coalescing
12. Logical Assignment Operators
13. Modules (import/export)
14. Dynamic Imports
15. Map
16. Set
17. WeakMap & WeakSet
18. Iterators
19. Generators
20. Symbol
21. BigInt
22. Modern Array Methods
23. Common Interview Questions
24. React Applications
```

---

# 1. Why Was ES6 Introduced?

## Definition

ES6 (ECMAScript 2015) was the biggest update in JavaScript's history.

Before ES6, JavaScript lacked many features found in modern languages.

Problems included:

* `var` causing scope issues
* No modules
* No classes
* Verbose string concatenation
* Difficult object copying
* Callback-heavy code
* No destructuring

ES6 introduced cleaner syntax and powerful new language features.

Since 2015 the language ships **yearly**, which is why you'll hear ES2020, ES2021, ES2023 rather than "ES7, ES8". Knowing roughly which era a feature comes from is useful in interviews — it tells you whether you can use it without a transpiler.

---

# 2. let vs const vs var (Review)

The short version — [Module 1](variables.md) covers this in full:

| | `var` | `let` | `const` |
| --- | --- | --- | --- |
| Scope | Function | Block | Block |
| Initialized on hoisting | Yes (`undefined`) | No (TDZ) | No (TDZ) |
| Redeclaration | Yes | No | No |
| Reassignment | Yes | Yes | No |

Default to `const`. Reach for `let` only when you actually reassign. `var` belongs in legacy code.

Remember that `const` prevents **reassignment**, not mutation — `const user = {}` still allows `user.name = "Rocky"`.

---

# 3. Template Literals

## Definition

Template literals allow embedded expressions inside strings.

Instead of:

```javascript
const name = "Rocky";

console.log("Hello " + name);
```

Use:

```javascript
const name = "Rocky";

console.log(`Hello ${name}`);
```

---

## Multi-line Strings

Without template literals:

```javascript
const text =
"Hello\n" +
"World";
```

With template literals:

```javascript
const text = `
Hello
World
`;
```

---

## Expression Interpolation

```javascript
const a = 5;
const b = 10;

console.log(`${a} + ${b} = ${a + b}`);
```

Output:

```
5 + 10 = 15
```

Any expression works inside `${}` — ternaries, function calls, template literals nested in template literals. Statements do not; `${if (x) ...}` is a syntax error.

---

## Tagged Templates

A function placed directly before a template literal receives the pieces instead of the finished string:

```javascript
function highlight(strings, ...values) {
    return strings.reduce(
        (out, str, i) => out + str + (values[i] ? `<b>${values[i]}</b>` : ""),
        ""
    );
}

const name = "Rocky";

highlight`Hello ${name}!`;
```

Output:

```
Hello <b>Rocky</b>!
```

Worth recognising because it powers tools you'll meet on the job — `styled-components`, `graphql-tag`, and the `css` helper in Emotion are all tagged templates. Interviewers rarely ask you to write one, but "what is this backtick syntax doing" comes up in code reading.

---

## React Example

```jsx
<h1>{`Welcome ${user.name}`}</h1>
```

In JSX you'd usually write `<h1>Welcome {user.name}</h1>` — the template literal only earns its place when you're building a single string, such as a `className`.

---

# 4. Destructuring

We've already seen the basics.

Let's go deeper.

---

## Array Destructuring

```javascript
const numbers = [10,20,30];

const [first, second] = numbers;
```

Result:

```
first = 10

second = 20
```

---

## Skipping Values

```javascript
const [,,third] = numbers;
```

Result:

```
third = 30
```

---

## Swapping Variables

Without destructuring:

```javascript
let a = 10;
let b = 20;

let temp = a;
a = b;
b = temp;
```

Modern JavaScript:

```javascript
[a,b] = [b,a];
```

---

## Nested Destructuring

```javascript
const user = {

    name:"Rocky",

    address:{
        city:"Milan"
    }

};

const {

address:{city}

}=user;
```

Guard it — if `address` is missing this throws, because you can't read a property off `undefined`:

```javascript
const { address: { city } = {} } = user;
```

---

## Function Parameter Destructuring

Instead of

```javascript
function createUser(user){

console.log(user.name);

}
```

Use

```javascript
function createUser({name}){

console.log(name);

}
```

Very common in React.

Give the whole parameter a default so calling it with no arguments doesn't throw:

```javascript
function createUser({ name = "Guest" } = {}) { }
```

---

# 5. Spread Operator (...)

## Definition

Spread expands iterable values.

---

## Arrays

```javascript
const a=[1,2];

const b=[...a,3,4];
```

Output

```
[1,2,3,4]
```

---

## Objects

```javascript
const user={

name:"Rocky"

};

const updated={

...user,

age:30

};
```

---

## Copy Arrays

```javascript
const copy=[...numbers];
```

---

## Copy Objects

```javascript
const copy={...user};
```

Remember:

Spread creates a **shallow copy**.

---

## Two different features with the same syntax

* **Array spread** (ES6) requires an **iterable**. `[...null]` throws `TypeError: null is not iterable`. Strings, `Set`, `Map`, and `NodeList` are all iterable, which is why `[...new Set(arr)]` and `[...document.querySelectorAll("li")]` work.
* **Object spread** (ES2018) copies **own enumerable properties**. It accepts anything — `{...null}` quietly gives `{}`.

Object spread also *invokes* getters rather than copying them, and does not carry the prototype across. A spread copy of a class instance is a plain object with the same data and none of the methods.

---

# 6. Rest Operator (...)

Looks identical to spread.

But does the opposite.

Spread

```
Expand
```

Rest

```
Collect
```

Example

```javascript
const [first,...rest]=[1,2,3,4];
```

Output

```
first=1

rest=[2,3,4]
```

Objects

```javascript
const {

name,

...others

}=user;
```

The way to tell them apart at a glance: **rest is on the left of `=` (or in a parameter list), spread is on the right.** Rest must also be last — `function f(...args, last)` is a syntax error.

---

# 7. Default Parameters

Old way

```javascript
function greet(name){

name=name||"Guest";

}
```

Problem

```
0

false

""
```

become

```
Guest
```

Modern

```javascript
function greet(name="Guest"){

console.log(name);

}
```

---

## Important Interview Point

Defaults fire **only for `undefined`** — not for `null`:

```javascript
greet(undefined);   // "Guest"
greet(null);        // null
greet(0);           // 0
```

Same rule as `??` and as destructuring defaults. If `null` should also fall back, you need `name ?? "Guest"` inside the body.

Two more properties worth knowing: defaults are evaluated **at call time** (so `function log(time = Date.now())` gives a fresh timestamp per call, unlike Python), and they're evaluated **left to right**, so a later parameter can use an earlier one:

```javascript
function range(start, end = start + 10) { }
```

---

# 8. Enhanced Object Literals

Before

```javascript
const name="Rocky";

const user={

name:name

};
```

Now

```javascript
const name="Rocky";

const user={

name

};
```

---

## Method Shorthand

Before

```javascript
const user={

greet:function(){

}

};
```

Now

```javascript
const user={

greet(){

}

};
```

Not purely cosmetic: shorthand methods have **no `prototype` property** and cannot be called with `new`. They can also use `super`, which a `function` expression in the same slot cannot. See [Module 5](this-prototypes.md).

---

# 9. Computed Property Names

```javascript
const key="city";

const user={

[key]:"Milan"

};
```

Produces

```javascript
{

city:"Milan"

}
```

The React pattern this enables — one handler for every field in a form:

```javascript
setForm(prev => ({
    ...prev,
    [e.target.name]: e.target.value
}));
```

---

# 10. Optional Chaining

Instead of

```javascript
user.address.city
```

which throws if `address` is missing:

```
TypeError: Cannot read properties of undefined (reading 'city')
```

Use

```javascript
user.address?.city
```

Returns

```
undefined
```

---

## Function Calls

```javascript
user.sayHello?.();
```

Calls only if the function exists.

---

## It short-circuits the whole chain

Once `?.` sees `null` or `undefined`, **the rest of the expression is never evaluated**:

```javascript
user?.address.city.length
```

If `user` is `undefined`, the result is `undefined` — `.address`, `.city`, and `.length` are all skipped, no error. That's why one `?.` at the genuinely-optional link is usually enough; you don't need it on every dot.

Use it where a value is *legitimately* optional. Scattering `?.` over a path that should never be null converts a loud crash into a silent `undefined` that surfaces three components later.

---

# 11. Nullish Coalescing

```javascript
const age=user.age??18;
```

Uses default only for

* null
* undefined

Not

```
0

false

""
```

---

Difference

```javascript
0||100
```

returns

```
100
```

while

```javascript
0??100
```

returns

```
0
```

Interview favorite.

The realistic version: `const pageSize = settings.pageSize || 20` silently rewrites a deliberate `0` to `20`, and `const label = props.label || "Untitled"` replaces an intentional empty string. Use `??` whenever `0`, `""`, or `false` are legal values.

One syntax rule: you cannot mix `??` with `||` or `&&` without parentheses — `a ?? b || c` is a `SyntaxError`, deliberately, because the precedence would be ambiguous.

---

# 12. Logical Assignment Operators

Introduced in ES2021.

---

## ||=

```javascript
user.name ||= "Guest";
```

Equivalent to

```javascript
if(!user.name){

user.name="Guest";

}
```

---

## &&=

Assigns only when the current value is **truthy** — useful for "transform it if it's there":

```javascript
user.name &&= user.name.trim();
```

If `user.name` is `undefined`, nothing happens and no error is thrown. If it's a string, it gets trimmed.

(`loggedIn &&= true` is a common example in tutorials, but it does nothing meaningful — it only ever rewrites a truthy value to `true`.)

---

## ??=

```javascript
user.age??=18;
```

Assigns only if

```
null

undefined
```

A neat use is initialising a nested accumulator in one line:

```javascript
grouped[key] ??= [];
grouped[key].push(item);
```

---

## Important Interview Point — they short-circuit the *assignment*

`x ||= y` is **not** the same as `x = x || y`. When the condition fails, no assignment happens at all — the property is never written:

```javascript
const config = Object.freeze({ retries: 3 });

config.retries ||= 5;                    // fine — no write attempted
config.retries = config.retries || 5;    // TypeError in strict mode
```

The second form always writes, even when the value is unchanged. This matters for frozen objects, for setters with side effects, and for reactive systems (Vue, MobX, Proxy-based stores) that trigger on every write.

---

# 13. Modules

Before ES6

Everything lived globally.

Now

```javascript
export function add(){}
```

Import

```javascript
import {add} from "./math";
```

---

## Default Export

```javascript
export default App;
```

Import

```javascript
import App from "./App";
```

---

## Named Export

```javascript
export const PI=3.14;

export function add(){}
```

Import

```javascript
import {PI,add} from "./math";
```

---

## Mixed

```javascript
export default User;

export const age=30;
```

Import

```javascript
import User,{age} from "./user";
```

---

## Four properties of ES modules worth stating

**1. Imports are static and hoisted.** The whole import graph is resolved before any code runs, which is what makes tree-shaking possible. You cannot write `if (x) import "./a"` — that's what dynamic `import()` is for.

**2. Modules are always strict mode**, with no way to opt out. This is why the `this` traps from [Module 5](this-prototypes.md) throw in React code instead of logging `undefined`.

**3. A module is a singleton.** It's evaluated once on first import and cached, no matter how many files import it. That's how a module-level variable becomes shared state across your app — sometimes intentionally, sometimes as a surprising bug.

**4. Imports are live bindings, not copies:**

```javascript
// counter.js
export let count = 0;
export function increment() { count++; }

// main.js
import { count, increment } from "./counter.js";

console.log(count);   // 0
increment();
console.log(count);   // 1  ← the import updated
```

CommonJS `require` copies the value, so the equivalent code there would still print `0`. You also can't assign to an import — `count = 5` throws, imports are read-only from the consumer's side.

---

## Named vs default — which to prefer

Named exports are generally the better default in a React codebase: they're refactor-safe (renaming the export updates every import), they enable editor auto-import, and they can't be silently renamed at the import site the way `import Whatever from "./Button"` can. Default exports remain the convention for React components in many teams, and `React.lazy` requires one — so know both and follow the codebase you're in.

---

# 14. Dynamic Import

```javascript
const module=await import("./math");
```

Useful for

* Lazy loading
* Code splitting
* Performance optimization

React uses this behind `React.lazy()`.

---

## What it actually returns

`import()` returns a promise for the **module namespace object**, not the module's default export. So you have to reach into it:

```javascript
const module = await import("./math");
module.add(2, 3);                        // named export

const { default: App } = await import("./App");   // default export
```

This is exactly why `React.lazy` requires a module with a **default export**:

```javascript
const Dashboard = React.lazy(() => import("./Dashboard"));
```

If `Dashboard.js` only has named exports, that line fails at runtime. The workaround is to map it yourself:

```javascript
const Dashboard = React.lazy(() =>
    import("./Dashboard").then(m => ({ default: m.Dashboard }))
);
```

Unlike static imports, `import()` can take a variable path and can sit inside an `if` — but bundlers need enough of a literal prefix to know what to split, so a fully dynamic `import(userInput)` usually won't work in a bundled app.

---

# 15. Map

## Definition

Map stores key-value pairs.

Unlike objects:

Keys can be any type.

Example

```javascript
const map=new Map();

map.set("name","Rocky");

map.set(1,"Number");

map.set({}, "Object");
```

---

## The API

```javascript
map.get("name");        // "Rocky"
map.has("name");        // true
map.delete("name");     // true
map.size;               // number of entries — O(1)

for (const [key, value] of map) { }
[...map.keys()];
```

---

## Important Interview Point — object keys are compared by reference

```javascript
const map = new Map();

map.set({}, "Object");

map.get({});      // undefined — a different {}
```

You need to hold on to the same reference:

```javascript
const key = {};

map.set(key, "Object");
map.get(key);     // "Object"
```

Key equality is **SameValueZero**, so `NaN` works as a key (unlike `indexOf`), and `0` and `-0` are treated as the same key.

---

## Why Not Objects?

Objects convert keys to strings.

Maps preserve key types.

Also provide:

* predictable iteration order
* convenient methods
* better performance for frequent insertions/deletions in some scenarios

Concretely: `obj[1]` and `obj["1"]` are the same property, but `map.get(1)` and `map.get("1")` are not. And an object inherits keys from `Object.prototype`, so `"toString" in obj` is `true` — a `Map` has no such collisions, which makes it the safer choice for user-supplied keys. See the comparison table in [Module 4](objects-arrays.md).

---

# 16. Set

Stores unique values.

```javascript
const ids=new Set([1,2,2,3]);
```

Result

```
1

2

3
```

Great for removing duplicates.

```javascript
const unique=[...new Set(array)];
```

---

## The performance reason to use one

`Set.has()` is O(1); `Array.includes()` is O(n). Inside a loop that difference is O(n) versus O(n×m):

```javascript
// ❌ scans selectedIds for every item
items.filter(item => selectedIds.includes(item.id));

// ✅ one pass to build, O(1) per lookup
const selected = new Set(selectedIds);
items.filter(item => selected.has(item.id));
```

This is the same lookup-table move as the `Map` example in [Module 4](objects-arrays.md), and it's the optimisation most worth having ready in an interview.

Note that uniqueness is by **SameValueZero** too, so `new Set([NaN, NaN]).size` is `1`, but `new Set([{}, {}]).size` is `2` — two different object references.

---

# 17. WeakMap

Similar to Map.

Difference:

Keys must be objects.

WeakMaps do not prevent garbage collection of their keys.

Common use:

Private metadata associated with objects.

The consequence of that weakness: a `WeakMap` is **not iterable** and has **no `size`**. You can't enumerate it, because entries may vanish at any moment when the garbage collector runs. If you need to list what's in it, you needed a `Map`.

The practical use is caching data *about* an object without keeping that object alive — attach metadata to a DOM node, and when the node is removed, both it and the entry are collected. With a plain `Map`, the entry would pin the node in memory forever. That's a genuine memory leak, and it's the reason this feature exists.

---

# 18. WeakSet

Like Set

But stores only objects.

Also weakly referenced.

Rare in interviews but worth recognizing.

---

# 19. Iterators

An iterator produces values one at a time.

Arrays already implement the iterator protocol.

```javascript
const arr=[1,2,3];

const iterator=arr[Symbol.iterator]();

console.log(iterator.next());
```

Result:

```javascript
{ value: 1, done: false }
```

Understanding iterators helps explain `for...of` and generators.

---

## The protocol, and what `for...of` really does

An object is **iterable** if it has a `[Symbol.iterator]()` method returning an object with a `next()` that yields `{ value, done }`. That's the whole contract. `for...of`, spread, `Array.from`, and destructuring all just call it.

Which means you can make your own type iterable:

```javascript
const range = {
    from: 1,
    to: 3,
    [Symbol.iterator]() {
        let current = this.from;
        const last = this.to;
        return {
            next: () => current <= last
                ? { value: current++, done: false }
                : { value: undefined, done: true }
        };
    }
};

[...range];   // [1, 2, 3]
```

---

## `for...of` vs `for...in`

A perennial interview question:

```javascript
const arr = ["a", "b"];
arr.extra = "x";

for (const i in arr) console.log(i);   // "0", "1", "extra"
for (const v of arr) console.log(v);   // "a", "b"
```

* **`for...in`** iterates **enumerable string keys**, including inherited ones. It's for objects — and even then `Object.keys()` is usually clearer.
* **`for...of`** iterates **values** via the iterator protocol. It's for arrays, strings, `Map`, `Set`, and anything else iterable. Plain objects are *not* iterable, so `for...of` over one throws.

Never use `for...in` on an array.

---

# 20. Generators

Generators pause and resume execution.

```javascript
function* numbers(){

yield 1;

yield 2;

yield 3;

}
```

Use

```javascript
const gen=numbers();

gen.next();
```

Output

```javascript
{

value:1,

done:false

}
```

Common use cases:

* Lazy evaluation
* Infinite sequences
* Custom iteration

A generator is the easy way to satisfy the iterator protocol — the `range` object above collapses to four lines:

```javascript
function* range(from, to) {
    for (let i = from; i <= to; i++) yield i;
}

[...range(1, 3)];   // [1, 2, 3]
```

Because execution is lazy, an infinite sequence is safe as long as you stop consuming it:

```javascript
function* naturals() {
    let n = 1;
    while (true) yield n++;
}
```

Values are computed only when requested. In frontend work you'll meet generators mainly through Redux-Saga; day to day they're rarer than the interview frequency suggests.

---

# 21. Symbol

Review:

Symbols create unique identifiers.

```javascript
const id=Symbol("id");
```

Useful for preventing property name collisions.

Every symbol is unique — `Symbol("id") !== Symbol("id")`. The string is only a debug label. Symbol-keyed properties are skipped by `Object.keys`, `for...in`, and `JSON.stringify`, which is what makes them safe for metadata on objects you don't own.

The **well-known symbols** are where you meet them in practice: `Symbol.iterator` is the one from the section above, and it's the usual interview answer for "where have you actually used a symbol".

---

# 22. BigInt

Review:

Allows safe integers beyond `Number.MAX_SAFE_INTEGER`.

```javascript
const huge=9007199254740993n;
```

Cannot mix `BigInt` and `Number` directly without explicit conversion.

```javascript
1n + 1;      // TypeError: Cannot mix BigInt and other types
1n + BigInt(1);   // 2n
```

`typeof huge` is `"bigint"`. `JSON.stringify` throws on one, which is the practical reason large IDs from an API usually arrive as strings rather than numbers.

---

# 23. Modern Array Methods

Recent JavaScript versions introduced non-mutating alternatives.

## toSorted()

```javascript
const sorted = numbers.toSorted((a,b)=>a-b);
```

Unlike `sort()`, the original array is unchanged.

---

## toReversed()

```javascript
const reversed = numbers.toReversed();
```

---

## toSpliced()

Immutable version of `splice()`.

---

## with()

Replace an element immutably.

```javascript
const updated = numbers.with(1, 100);
```

Original array remains unchanged.

---

## findLast()

Returns the last matching element.

```javascript
const lastEven = numbers.findLast(n => n % 2 === 0);
```

---

These are all ES2023. They've been available across current browsers and Node 20+ since 2023, but check your build target before using them in production — a project supporting older Safari still needs `[...arr].sort()`. They matter for React precisely because they return a **new reference**, which is what triggers a re-render ([Module 4](objects-arrays.md)).

---

# React Applications

Modern React relies on ES6+ everywhere:

* Destructuring props:

```jsx
function UserCard({ user }) {
    return <h2>{user.name}</h2>;
}
```

* Spread for immutable updates:

```javascript
setUser({
    ...user,
    age: 31
});
```

* Dynamic imports:

```javascript
const Dashboard = React.lazy(() => import("./Dashboard"));
```

* Optional chaining:

```javascript
user.profile?.avatar
```

* Nullish coalescing:

```javascript
const pageSize = settings.pageSize ?? 20;
```

---

# Common Interview Questions

### Beginner

* Difference between `let`, `const`, and `var`?
* What is destructuring?
* What is the spread operator?
* What is the rest operator?

### Intermediate

* Difference between spread and rest?
* Difference between `??` and `||`?
* Difference between default and named exports?
* What is a `Map` and when would you use it?
* Difference between `for...in` and `for...of`?

### Senior

* Why is the spread operator only a shallow copy?
* When should you use `Map` instead of a plain object?
* How does dynamic import improve performance?
* Why are immutable array methods (`toSorted`, `toSpliced`) useful in React?
* Explain the iterator protocol and how `for...of` works under the hood.
* Why does a `WeakMap` have no `size` property?
* Why are ES module imports live bindings, and when does that matter?

---

# Senior-Level Summary

By the end of this module, you should confidently explain:

* ES6 modernized JavaScript with cleaner syntax and modularity, and the language has shipped yearly since.
* Destructuring, spread, and rest improve readability and support immutable updates.
* `??` solves problems that `||` introduces with valid falsy values — and defaults, destructuring defaults, and `??` all trigger on `undefined` only, never `null`.
* Logical assignment operators short-circuit the **assignment**, not just the value.
* ES modules are static, strict, cached singletons with live bindings.
* Dynamic imports return a module namespace object and enable lazy loading and code splitting.
* `Map` and `Set` solve problems that objects and arrays don't handle elegantly — including O(1) lookups and non-string keys.
* `WeakMap`/`WeakSet` hold keys weakly, which is why they can't be iterated and why they prevent a specific class of memory leak.
* The iterator protocol is what `for...of`, spread, and destructuring are built on, and generators are the shorthand for implementing it.
* New immutable array methods align well with React's rendering model.
* Modern JavaScript features aren't just syntactic sugar—they often improve maintainability, performance, and correctness.

---

# Practice Questions

Answer these without running them:

### 1. What does each line log?

```javascript
function greet(name = "Guest") { console.log(name); }

greet();
greet(undefined);
greet(null);
greet("");
```

---

### 2. Why does the second lookup fail?

```javascript
const cache = new Map();

cache.set({ id: 1 }, "first");

console.log(cache.get({ id: 1 }));
```

Fix it in two different ways.

---

### 3. One of these throws. Which, and why?

```javascript
const config = Object.freeze({ retries: 3 });

config.retries ||= 5;
config.retries = config.retries || 5;
```

---

### 4. What does this log, and what would CommonJS `require` have logged instead?

```javascript
// counter.js
export let count = 0;
export function increment() { count++; }

// main.js
import { count, increment } from "./counter.js";
increment();
console.log(count);
```

---

### 5. Explain:

`React.lazy(() => import("./Dashboard"))` throws at runtime for a module that exports `Dashboard` as a named export. Explain what `import()` actually resolves to, and write the one-line fix.

---

# We've Completed the JavaScript Section 🎉

At this point, you've covered JavaScript from fundamentals to advanced topics:

* ✅ Variables & Scope
* ✅ Data Types & Memory
* ✅ Functions & Closures
* ✅ Objects & Arrays
* ✅ `this` & Prototypes
* ✅ Asynchronous JavaScript & Event Loop
* ✅ ES6+ & Modern JavaScript

This is a strong foundation for frontend interviews.

## Next Phase — **TypeScript**

We'll cover it with the same depth, including:

* Type system fundamentals
* Interfaces vs Types
* Generics
* Utility types (`Partial`, `Pick`, `Omit`, `Record`, etc.)
* Conditional and mapped types
* Type guards
* Discriminated unions
* Declaration merging
* Module augmentation
* Advanced interview questions
* React + TypeScript best practices

This is the level of TypeScript knowledge typically expected for mid-level and senior React roles.

---

| ← Previous | Index | Next → |
| ---------- | ----- | ------ |
| [Module 6 — Async JavaScript & the Event Loop](async-event-loop.md) | [Study Plan](../README.md) | [Module 8 — React Fundamentals](../react/react-fundamentals.md) |
