[← Study Plan](../README.md)

# Module 4 — Objects & Arrays (Complete Guide)

---

# Table of Contents

This module covers:

```
1. Objects
2. Creating Objects
3. Properties & Methods
4. Property Access
5. Object Destructuring
6. Object Spread & Rest
7. Object Methods
8. Shallow Copy vs Deep Copy
9. Object.freeze(), seal(), preventExtensions()
10. Optional Chaining & Nullish Coalescing
11. Arrays
12. Array Methods
13. Higher-Order Array Methods
14. Array Mutation vs Non-Mutation
15. Sorting
16. Searching
17. Flattening Arrays
18. Common Interview Questions
19. React Applications
20. Performance Considerations
```

---

# Part 1 — Objects

## What is an Object?

### Definition

An **object** is a collection of related data stored as **key-value pairs**.

Unlike primitive values, objects can hold multiple values of different types and represent real-world entities.

Example:

```javascript
const user = {
    name: "Rakesh",
    age: 30,
    isDeveloper: true
};
```

Each entry consists of:

```
Key      Value

name  -> "Rakesh"

age   -> 30

isDeveloper -> true
```

---

## Why do Objects Exist?

Imagine storing user information without objects.

```javascript
const firstName = "Rakesh";
const lastName = "Rankawat";
const age = 30;
const city = "Milan";
```

This quickly becomes difficult to manage.

Using an object:

```javascript
const user = {
    firstName: "Rakesh",
    lastName: "Rankawat",
    age: 30,
    city: "Milan"
};
```

Everything about the user is grouped together.

Objects improve:

* Organization
* Readability
* Maintainability
* Reusability

---

# Creating Objects

## Object Literal (Most Common)

```javascript
const car = {
    brand: "BMW",
    year: 2025
};
```

---

## Object Constructor

```javascript
const user = new Object();

user.name = "Rocky";
```

Rarely used today.

---

## Object.create()

```javascript
const animal = {
    eats: true
};

const dog = Object.create(animal);

dog.name = "Buddy";
```

Useful when working with prototypes.

`dog.eats` is `true` even though `dog` has no `eats` property of its own — it is found on the prototype. Module 5 covers this properly.

`Object.create(null)` creates an object with **no** prototype at all — no `toString`, no `hasOwnProperty`. That's the safe choice for a pure dictionary where user-supplied keys could collide with inherited names.

---

## Constructor Function

```javascript
function User(name, age) {
    this.name = name;
    this.age = age;
}

const user = new User("Rocky", 30);
```

Mostly replaced by ES6 classes.

---

## ES6 Class

```javascript
class User {

    constructor(name) {
        this.name = name;
    }

}

const user = new User("Rocky");
```

Classes are syntactic sugar over JavaScript's prototype system — with a few real differences: class bodies always run in strict mode, classes are not hoisted the way function declarations are, and calling one without `new` throws a `TypeError`.

---

# Properties

Properties store data.

```javascript
const user = {

    name: "Rocky",

    age: 30

};
```

```
name
age
```

are properties.

---

# Methods

Methods are simply functions stored as object properties.

```javascript
const user = {

    name: "Rocky",

    greet() {

        console.log("Hello");

    }

};
```

---

# Property Access

## Dot Notation

```javascript
user.name
```

Most common.

---

## Bracket Notation

```javascript
user["name"]
```

Useful when the property name is dynamic.

Example:

```javascript
const key = "age";

console.log(user[key]);
```

Bracket notation is also the only option when the key isn't a valid identifier:

```javascript
const config = {
    "api-url": "https://example.com"
};

config["api-url"];    // works
config.api-url;       // parsed as `config.api - url` → ReferenceError
```

---

## Computed Property Names

The same brackets work when *creating* an object:

```javascript
const field = "email";

const form = {
    [field]: "rocky@example.com"
};
```

Output:

```javascript
{ email: "rocky@example.com" }
```

This is how a single React change handler updates the right field:

```javascript
setForm(prev => ({
    ...prev,
    [event.target.name]: event.target.value
}));
```

---

# Adding Properties

```javascript
user.country = "Italy";
```

---

# Updating Properties

```javascript
user.age = 31;
```

---

# Removing Properties

```javascript
delete user.age;
```

---

# Checking Property Existence

```javascript
"name" in user
```

returns

```
true
```

---

## Important Interview Point

`in` searches the **whole prototype chain**, not just the object itself:

```javascript
const user = { name: "Rocky" };

console.log("name" in user);        // true
console.log("toString" in user);    // true  ← inherited from Object.prototype
```

For "does this object *itself* have the key", use `Object.hasOwn()` (ES2022):

```javascript
Object.hasOwn(user, "toString");    // false
Object.hasOwn(user, "name");        // true
```

Also worth knowing: checking `user.age !== undefined` is not the same test. A property that exists and is explicitly set to `undefined` passes `in` but fails that check.

---

# Object Destructuring

Instead of

```javascript
const name = user.name;

const age = user.age;
```

Use:

```javascript
const { name, age } = user;
```

---

## Rename During Destructuring

```javascript
const {

    name: fullName

} = user;
```

Now

```javascript
console.log(fullName);
```

---

## Default Values

```javascript
const {

country = "India"

} = user;
```

The default applies only when the value is `undefined` — not when it is `null`, `0`, or `""`. Same rule as `??`.

---

## Destructuring in Function Parameters

This is the form you'll write most often in React:

```javascript
function UserCard({ name, age = 0, onSelect }) {
    // ...
}
```

Props are destructured straight out of the props object, with a default for `age`.

---

# Nested Destructuring

```javascript
const user = {

address:{

city:"Milan"

}

};

const {

address:{

city

}

}=user;
```

Careful: this throws if `address` is missing, because you can't destructure a property off `undefined`. Guard it with a default:

```javascript
const { address: { city } = {} } = user;
```

---

# Array Destructuring

Position matters instead of name:

```javascript
const [first, second] = [10, 20, 30];
```

Skip with commas, collect the rest:

```javascript
const [, , third] = [10, 20, 30];      // 30

const [head, ...tail] = [1, 2, 3, 4];  // head = 1, tail = [2, 3, 4]
```

Swapping without a temp variable:

```javascript
let a = 1;
let b = 2;

[a, b] = [b, a];
```

This is exactly what `useState` returns — an array you destructure by position, which is why you can name the pair whatever you like:

```javascript
const [count, setCount] = useState(0);
```

---

# Spread Operator

Copies properties.

```javascript
const updatedUser = {

...user,

age:31

};
```

React uses this everywhere.

Order matters — later keys overwrite earlier ones:

```javascript
{ ...user, age: 31 }   // age is 31
{ age: 31, ...user }   // user.age wins, the 31 is discarded
```

---

# Rest Operator

Collects remaining properties.

```javascript
const {

name,

...rest

}=user;
```

A common real use is stripping a field you don't want to pass on:

```javascript
const { password, ...safeUser } = user;
```

---

# Object.keys()

Returns property names.

```javascript
Object.keys(user);
```

Output

```javascript
["name","age"]
```

---

# Object.values()

```javascript
Object.values(user);
```

Output

```javascript
["Rocky",30]
```

---

# Object.entries()

```javascript
Object.entries(user);
```

Output

```javascript
[
["name","Rocky"],
["age",30]
]
```

Great for loops.

```javascript
for (const [key, value] of Object.entries(user)) {
    console.log(key, value);
}
```

---

## Important Interview Point — key order

Object keys are **not** purely insertion-ordered. Integer-like keys come first in ascending numeric order, then string keys in insertion order:

```javascript
const scores = { "2": "b", "1": "a", name: "Rocky" };

Object.keys(scores);
```

Output:

```javascript
["1", "2", "name"]
```

If order matters, use a `Map` — it preserves true insertion order and allows non-string keys.

---

# Object.assign()

Creates shallow copies.

```javascript
const copy = Object.assign({},user);
```

Equivalent to:

```javascript
const copy = {

...user

};
```

With an empty `{}` target these behave the same. With a non-empty target, `Object.assign` **mutates** that target and returns it, while spread always builds a new object — that's the difference to state in an interview.

---

# Shallow Copy vs Deep Copy

Both `Object.assign({}, user)` and `{ ...user }` are **shallow**. Top-level values are copied; nested objects are still shared by reference.

```javascript
const user = {
    name: "Rocky",
    address: { city: "Milan" }
};

const copy = { ...user };

copy.address.city = "Rome";

console.log(user.address.city);
```

Output:

```
Rome
```

The nested object was never copied — both objects point at the same `address`.

For a true deep copy in modern environments:

```javascript
const deep = structuredClone(user);
```

`structuredClone` handles nested objects, `Map`, `Set`, `Date`, and cyclic references. It cannot clone functions, DOM nodes, or class prototypes.

The old `JSON.parse(JSON.stringify(obj))` trick still works but silently destroys `undefined`, functions, `Date` (becomes a string), `Map`, `Set`, and `NaN`/`Infinity` (become `null`), and throws on cycles.

This is covered in depth in [Module 2 — Data Types & Memory](data-types.md).

---

# Object.freeze()

Makes an object immutable.

```javascript
const user = {

name:"Rocky"

};

Object.freeze(user);

user.name="Alex";
```

---

## Important Interview Point

What happens next depends on strict mode:

* **Sloppy mode** (a classic `<script>`, a plain `.js` file with no `"use strict"`): the assignment fails **silently**. `user.name` is still `"Rocky"`.
* **Strict mode** (every ES module, every `class` body, anything with `"use strict"`): it throws `TypeError: Cannot assign to read only property 'name' of object`.

Since all modern React code is ES modules, you get the error, not the silent no-op. Saying "nothing happens" is only half right.

`Object.freeze` is also **shallow**:

```javascript
const config = Object.freeze({
    api: { url: "/v1" }
});

config.api.url = "/v2";    // works — the nested object is not frozen
```

Freezing deeply means recursing over the object yourself.

---

# Object.seal()

Allows:

✅ Updating properties

❌ Adding new ones

❌ Removing existing ones

---

# preventExtensions()

Allows

✅ Updating existing properties

❌ Adding new properties

---

## The three compared

The draft above misses one row — `preventExtensions` still lets you **delete**:

| | Add | Update | Delete | Reconfigure |
| ---------------------------- | :-: | :----: | :----: | :---------: |
| `Object.preventExtensions()` | ❌ | ✅ | ✅ | ✅ |
| `Object.seal()`              | ❌ | ✅ | ❌ | ❌ |
| `Object.freeze()`            | ❌ | ❌ | ❌ | ❌ |

Each has a matching check: `Object.isExtensible()`, `Object.isSealed()`, `Object.isFrozen()`.

---

# Optional Chaining

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

```javascript
undefined
```

instead of throwing an error.

It also works for calls and for indexes:

```javascript
user.getName?.();      // undefined if getName doesn't exist
users?.[0]?.name;
```

Use it where a value is **legitimately** optional. Sprinkling `?.` over a path that should never be null just hides bugs — the crash was telling you something real.

---

# Nullish Coalescing

```javascript
const city = user.city ?? "Unknown";
```

Uses the default only when the value is `null` or `undefined`.

Unlike `||`, it does **not** replace valid falsy values like `0`, `false`, or `""`.

Example:

```javascript
console.log(0 || 100);   // 100
console.log(0 ?? 100);   // 0
```

This distinction often comes up in interviews.

The real-world version: `const pageSize = input || 10` silently rewrites a deliberate `0` to `10`. With `??`, the `0` survives.

---

# Part 2 — Arrays

## Definition

An array is an ordered collection of values.

```javascript
const numbers = [10,20,30];
```

Arrays can store any type.

```javascript
const values = [

10,

"Hello",

true,

{},

[]

];
```

---

## Important Interview Point — arrays are objects

```javascript
typeof [];           // "object"
typeof {};           // "object"
```

`typeof` cannot tell them apart. Use:

```javascript
Array.isArray([]);   // true
Array.isArray({});   // false
```

Under the hood an array is an object with integer-like keys and a `length` property that updates itself. That's why `arr.foo = "bar"` is legal — and why you should never do it.

---

# Accessing Elements

```javascript
numbers[0]
```

For the last element, `at()` (ES2022) beats the arithmetic:

```javascript
numbers[numbers.length - 1];   // old way
numbers.at(-1);                // same thing
```

---

# Length

```javascript
numbers.length
```

`length` is writable, which surprises people:

```javascript
const arr = [1, 2, 3];
arr.length = 1;

console.log(arr);   // [1]
```

---

# push()

Adds to the end.

```javascript
numbers.push(40);
```

Returns the new length.

---

# pop()

Removes the last element.

Returns the removed element.

---

# shift()

Removes the first element.

Returns the removed element. Every remaining element must be re-indexed, which is why it's O(n).

---

# unshift()

Adds to the beginning.

---

# splice()

Mutates the array.

```javascript
numbers.splice(1,2);
```

Removes 2 elements starting at index 1, and returns the removed ones. It can insert too:

```javascript
numbers.splice(1, 0, 99);   // insert 99 at index 1, remove nothing
```

---

# slice()

Does **not** mutate.

```javascript
numbers.slice(1,3);
```

Returns a new array with elements from index 1 up to *but not including* 3.

**Remember the pair:** `splice` mutates and returns what it removed; `slice` copies and returns what it selected. One letter apart, opposite behaviour — a very common interview question.

---

# map()

Returns a new array after transforming each element.

```javascript
const prices = [10,20,30];

const doubled = prices.map(price => price * 2);

console.log(doubled);
```

Output:

```javascript
[20, 40, 60]
```

**React example:**

```jsx
users.map(user => (
  <UserCard key={user.id} user={user} />
));
```

---

## map() vs forEach()

```javascript
const result = [1, 2, 3].forEach(n => n * 2);

console.log(result);
```

Output:

```
undefined
```

`forEach` always returns `undefined` — you use it for side effects. `map` returns the new array, so it's the one you can chain and the one JSX can render. Using `forEach` where you meant `map` produces a blank UI with no error.

---

# filter()

Returns only elements that satisfy a condition.

```javascript
const even = [1,2,3,4].filter(n => n % 2 === 0);
```

Output:

```javascript
[2, 4]
```

The callback's return value is coerced to a boolean, so `filter(Boolean)` is a neat way to drop all falsy entries.

---

# reduce()

One of the most powerful methods.

```javascript
const total = [10,20,30].reduce(

(sum,n)=>sum+n,

0

);
```

Output:

```
60
```

Common uses:

* Sum values
* Group data
* Build lookup objects
* Flatten arrays
* Count occurrences

Grouping, written out — this is a standard interview task:

```javascript
const people = [
    { name: "Rocky", city: "Milan" },
    { name: "Alex", city: "Rome" },
    { name: "John", city: "Milan" }
];

const byCity = people.reduce((acc, person) => {
    (acc[person.city] ||= []).push(person);
    return acc;
}, {});
```

Output:

```javascript
{
  Milan: [{ name: "Rocky", ... }, { name: "John", ... }],
  Rome:  [{ name: "Alex", ... }]
}
```

`Object.groupBy(people, p => p.city)` (ES2024) does the same thing in one line, but check your browser and Node targets before relying on it.

---

## Important Interview Point

Always pass the initial value. Without it, `reduce` uses the first element as the seed — and on an empty array it throws:

```javascript
[].reduce((a, b) => a + b);
```

Output:

```
TypeError: Reduce of empty array with no initial value
```

```javascript
[].reduce((a, b) => a + b, 0);   // 0 — safe
```

Filtered lists are empty all the time, so this bug reaches production easily.

---

# find()

Returns the first matching element.

```javascript
users.find(user => user.id === 10);
```

Returns `undefined` if nothing matches — not `null`, and not `-1`.

---

# findIndex(), indexOf(), findLast()

```javascript
users.findIndex(u => u.id === 10);   // index, or -1 if not found
[10, 20].indexOf(20);                // 1  — compares by value with ===
users.findLast(u => u.active);       // ES2023, searches from the end
```

`indexOf` uses strict equality, so it can't find objects by shape — `[{id: 1}].indexOf({id: 1})` is `-1`, because those are two different references. Use `findIndex` for objects.

---

# some()

Returns `true` if at least one element matches.

---

# every()

Returns `true` only if every element matches.

Note the edge case interviewers like: `[].every(...)` is `true` and `[].some(...)` is `false`, for any callback.

---

# includes()

```javascript
["React","Vue"].includes("React");
```

Unlike `indexOf`, it finds `NaN`:

```javascript
[NaN].indexOf(NaN);      // -1
[NaN].includes(NaN);     // true
```

---

# sort()

**Important Interview Point**

By default, `sort` converts every element to a string and compares UTF-16 code units. The classic demonstration:

```javascript
console.log([10, 9, 1].sort());
```

Output:

```javascript
[1, 10, 9]
```

`"10"` sorts before `"9"` because the character `"1"` comes before `"9"`.

Correct:

```javascript
numbers.sort((a, b) => a - b);   // ascending
numbers.sort((a, b) => b - a);   // descending
```

Two more things worth saying:

* `sort` **mutates** the original array *and* returns it. `const sorted = arr.sort()` gives you two names for one array.
* Since ES2019 it is guaranteed **stable** — equal elements keep their relative order, which is what makes multi-key sorting work.

For strings with accents or non-English characters, `a.localeCompare(b)` beats `<`:

```javascript
["ö", "a", "z"].sort((a, b) => a.localeCompare(b));
```

---

# reverse()

Mutates the array.

---

# flat()

```javascript
[[1],[2],[3]].flat();
```

Output:

```javascript
[1,2,3]
```

`flat()` defaults to one level deep. Pass a depth, or `Infinity` for fully nested data:

```javascript
[1, [2, [3, [4]]]].flat();           // [1, 2, [3, [4]]]
[1, [2, [3, [4]]]].flat(Infinity);   // [1, 2, 3, 4]
```

---

# flatMap()

Maps and flattens in one step (one level only).

```javascript
const orders = [
    { items: ["pen", "book"] },
    { items: ["laptop"] }
];

orders.flatMap(order => order.items);
```

Output:

```javascript
["pen", "book", "laptop"]
```

Returning `[]` from the callback drops an element, which makes `flatMap` a map-and-filter in a single pass.

---

# Mutation vs Non-Mutation

Interviewers love this.

### Mutating methods

* push
* pop
* shift
* unshift
* splice
* sort
* reverse
* fill
* copyWithin

### Non-mutating methods

* map
* filter
* reduce
* find
* some
* every
* slice
* concat
* flat
* flatMap
* toSorted (ES2023)
* toReversed (ES2023)
* toSpliced (ES2023)
* with (ES2023)

React strongly prefers non-mutating updates.

The ES2023 additions exist precisely for this problem — `arr.toSorted()` gives you the sorted copy that `arr.sort()` never did, and `arr.with(2, newValue)` replaces one index without touching the original:

```javascript
const updated = todos.with(2, { ...todos[2], done: true });
```

The pre-ES2023 equivalent is `[...todos].sort()` — copy first, then mutate the copy.

---

# Common React Mistakes

❌ Wrong

```javascript
todos.push(newTodo);

setTodos(todos);
```

The array reference doesn't change.

---

✅ Correct

```javascript
setTodos([...todos, newTodo]);
```

A new array reference is created.

---

## Why the reference matters

React's state updates use `Object.is` to compare old and new state. `todos === todos` is `true` after a `push`, so React concludes nothing changed and skips the re-render. Your data changed; your UI didn't. Nothing errors — that's what makes it hard to spot.

The same rule drives `React.memo`, `useMemo`, and `useEffect` dependency arrays. All of them compare references, not contents.

---

## The nested update trap

Spread is shallow, so one level of spread is not enough for nested state:

```javascript
// ❌ mutates the nested object that state still points at
setUser(prev => {
    prev.address.city = "Rome";
    return { ...prev };
});

// ✅ new object at every level you touch
setUser(prev => ({
    ...prev,
    address: { ...prev.address, city: "Rome" }
}));
```

---

## Keys

```jsx
{todos.map((todo, index) => (
    <TodoItem key={index} todo={todo} />
))}
```

Using the array index as a `key` breaks as soon as the list can be reordered, filtered, or have items inserted anywhere but the end — React matches elements by key, so the wrong component keeps the wrong state. Use a stable id from the data:

```jsx
<TodoItem key={todo.id} todo={todo} />
```

Index keys are acceptable only for a static list that never changes order.

---

# Performance Considerations

Understanding complexity helps you discuss trade-offs.

| Operation                 | Average Complexity |
| ------------------------- | -----------------: |
| Array access by index     |               O(1) |
| push()                    |     O(1) amortized |
| pop()                     |               O(1) |
| shift()                   |               O(n) |
| unshift()                 |               O(n) |
| map()                     |               O(n) |
| filter()                  |               O(n) |
| reduce()                  |               O(n) |
| find()                    |               O(n) |
| includes()                |               O(n) |
| sort()                    |         O(n log n) |
| Object property access    |               O(1) |
| Object property insertion |               O(1) |
| Object property deletion  |               O(1) |

Two footnotes worth knowing:

* `push` is **amortized** O(1) — the backing store occasionally has to grow and copy, which is O(n) for that one call.
* `delete obj.key` is O(1) on paper, but in V8 it can push the object out of its optimised "hidden class" shape into dictionary mode, making every later property access on it slower. Setting the value to `undefined`, or building a new object without the key, is usually the better move in a hot path.

The pattern that actually bites in interviews is the accidental O(n²):

```javascript
// O(n × m) — a scan of `orders` for every user
users.map(user => orders.find(o => o.userId === user.id));
```

Build a lookup first, and it's O(n + m):

```javascript
const ordersByUser = new Map(orders.map(o => [o.userId, o]));

users.map(user => ordersByUser.get(user.id));
```

That is the single most useful optimisation to be able to reach for out loud.

---

# Objects vs Map

| | Object | Map |
| --- | --- | --- |
| Key types | strings and symbols only | any value, including objects |
| Order | integer keys first, then insertion order | true insertion order |
| Size | `Object.keys(o).length` — O(n) | `map.size` — O(1) |
| Iteration | needs `Object.keys/entries` | directly iterable |
| Prototype keys | inherits `toString` etc. | none |

Use an object for records with known, fixed fields. Use a `Map` for a real dictionary — many keys, frequent adds and deletes, non-string keys, or when order matters.

---

# Common Interview Questions

### Beginner

* What is an object?
* Difference between an object and an array?
* Dot notation vs bracket notation?
* What is destructuring?
* What is the spread operator?

### Intermediate

* Difference between `map()` and `forEach()`?
* Difference between `slice()` and `splice()`?
* Explain shallow copy vs deep copy.
* Explain optional chaining.
* Difference between `??` and `||`.

### Senior

* Why are immutable updates important in React?
* When would you use a `Map` instead of an object?
* How would you efficiently group a list of objects?
* Why is `sort()` considered mutating, and how can ES2023's `toSorted()` help?
* How would you optimize rendering a list with thousands of items?

---

# Senior-Level Summary

By the end of this module, you should confidently explain:

* How objects and arrays are represented and manipulated.
* The difference between mutation and immutability.
* Why React depends on reference changes for state updates — and that the comparison is `Object.is`, not a deep equality check.
* When to use each array method and its time complexity.
* Modern JavaScript features like optional chaining, nullish coalescing, destructuring, and the spread/rest operators.
* That spread, `Object.assign`, and `Object.freeze` are all **shallow**, and what to reach for when you need depth.
* The trade-offs between different collection types and update patterns.

---

# Practice Questions

Answer these without running them:

### 1. What is the output?

```javascript
const a = { x: 1, nested: { y: 2 } };
const b = { ...a };

b.x = 99;
b.nested.y = 99;

console.log(a.x, a.nested.y);
```

---

### 2. What is the output, and why?

```javascript
console.log([1, 5, 10, 40].sort());
```

---

### 3. Predict both lines:

```javascript
const settings = { retries: 0, label: "" };

console.log(settings.retries || 3);
console.log(settings.retries ?? 3);
```

---

### 4. What does this log, and what did the author probably intend?

```javascript
const names = ["a", "b"].forEach(n => n.toUpperCase());

console.log(names);
```

---

### 5. Explain:

A component calls `setItems(items)` after `items.sort((a, b) => a.price - b.price)`. The list does not re-render. Give both reasons this fails, and fix it in one line.

---

## Coming Up Next — Module 5: `this`, Prototypes & the Prototype Chain

This is where many experienced developers still struggle. We'll cover:

* What `this` really is (and why it's determined by the call site, not where a function is written).
* Implicit, explicit, default, and constructor binding.
* `call()`, `apply()`, and `bind()`.
* Prototypes and the prototype chain.
* How inheritance works under the hood.
* How ES6 classes are built on prototypes.
* How JavaScript resolves property and method lookups.
* Prototype-related interview questions and common pitfalls.

Understanding this module will also make topics like classes, inheritance, and even parts of React and TypeScript much easier to reason about.

---

| ← Previous | Index | Next → |
| ---------- | ----- | ------ |
| [Module 3 — Functions & Closures](functions-closures.md) | [Study Plan](../README.md) | [Module 5 — `this` & Prototypes](this-prototypes.md) |
