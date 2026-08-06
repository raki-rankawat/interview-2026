[← Study Plan](../README.md)

# Module 1 — Variables in JavaScript

## What is a Variable?

A variable is a **named storage location in memory** used to store data that can be referenced and manipulated by your program.

Think of it like a labeled box:

```
Age  ───► 25

Name ───► "Rocky"

LoggedIn ───► true
```

Instead of remembering where the value is stored in memory, JavaScript lets you access it using a variable name.

Example:

```javascript
let age = 25;
let name = "Rocky";
const isStudent = true;
```

---

## Why Do Variables Exist?

Imagine writing:

```javascript
console.log(25);
console.log(25 + 5);
console.log(25 * 2);
```

If the value changes, you'd have to update it everywhere.

Instead:

```javascript
let age = 25;

console.log(age);
console.log(age + 5);
console.log(age * 2);
```

Now changing the value only requires changing one line.

---

# Variable Declarations

JavaScript provides three keywords:

* `var`
* `let`
* `const`

Although all declare variables, they behave differently in terms of scope, hoisting, and reassignment.

---

# `var`

## Definition

`var` is the original way of declaring variables in JavaScript (before ES6, released in 2015).

Example:

```javascript
var name = "John";
```

---

## Characteristics

* Function-scoped
* Hoisted and initialized with `undefined`
* Can be reassigned
* Can be redeclared in the same scope
* In browsers, global `var` declarations become properties of the `window` object

---

## Scope

Unlike `let` and `const`, `var` ignores block boundaries.

```javascript
if (true) {
    var message = "Hello";
}

console.log(message);
```

Output:

```
Hello
```

The variable is still accessible because `var` only creates function scope.

---

## Why is this a problem?

Consider:

```javascript
for (var i = 0; i < 3; i++) {
}

console.log(i);
```

Output:

```
3
```

The loop variable leaked outside the loop.

With `let`:

```javascript
for (let i = 0; i < 3; i++) {
}

console.log(i);
```

Output:

```
ReferenceError
```

This is safer because the loop variable only exists inside the loop.

---

## Hoisting

When JavaScript executes code, it first creates an execution context.

During this creation phase:

* Function declarations are fully hoisted.
* `var` declarations are hoisted and initialized to `undefined`.
* `let` and `const` are hoisted but remain uninitialized until execution reaches their declaration (the Temporal Dead Zone).

Example:

```javascript
console.log(a);

var a = 10;
```

Internally, JavaScript treats it approximately like this:

```javascript
var a;

console.log(a);

a = 10;
```

Output:

```
undefined
```

---

## Common Mistakes

```javascript
var score = 10;
var score = 20;
```

This is allowed.

It can accidentally overwrite variables and make debugging difficult.

---

# `let`

## Definition

`let` was introduced in ES6 (2015) to solve many of the issues with `var`.

---

## Characteristics

* Block-scoped
* Cannot be redeclared in the same scope
* Can be reassigned
* Subject to the Temporal Dead Zone

Example:

```javascript
let age = 25;

age = 26;
```

This is valid because `let` allows reassignment.

---

## Block Scope

```javascript
{
    let city = "Milan";
}

console.log(city);
```

Output:

```
ReferenceError
```

The variable only exists within the block.

---

# Temporal Dead Zone (TDZ)

## Definition

A temporal dead zone (TDZ) is the area of a block where a variable is inaccessible until the moment the computer completely initializes it with a value.

---

## Explanation

`let` and `const` declarations **are** hoisted to the top of their block — but unlike `var`, they are not initialized with `undefined`. The binding exists, yet the engine refuses to let you touch it until execution reaches the declaration line.

That window — from the start of the block to the line of declaration — is the TDZ.

```javascript
{
    // TDZ for `city` starts here
    console.log(city);   // ReferenceError: Cannot access 'city' before initialization

    let city = "Milan";  // TDZ for `city` ends here

    console.log(city);   // "Milan"
}
```

---

## TDZ vs `var`

```javascript
console.log(a); // undefined  → var is hoisted AND initialized
var a = 10;

console.log(b); // ReferenceError → let is hoisted but NOT initialized
let b = 10;
```

Note the difference in errors:

* An undeclared variable gives `ReferenceError: b is not defined`.
* A TDZ variable gives `ReferenceError: Cannot access 'b' before initialization`.

The second message proves the binding exists — it's just unusable.

---

## Why does the TDZ exist?

* It turns silent bugs into loud errors. With `var`, using a variable too early gives you `undefined` and the bug surfaces much later.
* It makes `const` meaningful. A `const` cannot be assigned twice, so it can't be pre-initialized to `undefined` and then set — it must stay uninitialized until its one assignment.

---

# `const`

## Definition

`const` creates a block-scoped variable that **cannot be reassigned** after initialization.

---

## Characteristics

* Block-scoped
* Must be initialized immediately
* Cannot be reassigned
* Objects and arrays declared with `const` can still have their contents modified

Example:

```javascript
const PI = 3.14159;
```

Attempting to reassign it:

```javascript
PI = 4;
```

Produces:

```
TypeError
```

---

## Objects with `const`

Many developers initially think `const` makes objects immutable.

It does not.

```javascript
const user = {
    name: "Rocky"
};

user.name = "Alex";
```

This works because you're modifying the object's contents, not changing the variable to reference a different object.

This is **not** allowed:

```javascript
user = {};
```

because it changes the reference itself.

---

# `var` vs `let` vs `const`

| Feature                     | var               | let   | const |
| --------------------------- | ----------------- | ----- | ----- |
| Scope                       | Function          | Block | Block |
| Hoisted                     | Yes               | Yes   | Yes   |
| Initialized during hoisting | Yes (`undefined`) | No    | No    |
| Redeclaration               | Yes               | No    | No    |
| Reassignment                | Yes               | Yes   | No    |
| Temporal Dead Zone          | No                | Yes   | Yes   |

---

# Best Practices

* Use `const` by default.
* Use `let` only when reassignment is required.
* Avoid `var` in modern codebases unless maintaining legacy code.

---

# Interview Questions

* Explain the difference between `var`, `let`, and `const`.
* What is hoisting?
* What is the Temporal Dead Zone?
* Why was `let` introduced?
* Why can a `const` object still be modified?
* Why is `var` generally avoided today?

---

# Real-World Scenario

Imagine a React component where a loop creates click handlers.

Using `var`:

```javascript
for (var i = 0; i < 3; i++) {
    button.onclick = () => console.log(i);
}
```

Every click logs:

```
3
```

because there's one shared `i`.

Using `let`:

```javascript
for (let i = 0; i < 3; i++) {
    button.onclick = () => console.log(i);
}
```

Each iteration gets its own binding, so the handlers log `0`, `1`, and `2` as expected.

---

| ← Previous | Index | Next → |
| ---------- | ----- | ------ |
| — (first module) | [Study Plan](../README.md) | [Module 2 — Data Types & Memory](data-types.md) |
