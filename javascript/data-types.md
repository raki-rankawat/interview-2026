[← Study Plan](../README.md)

# Module 2 — Data Types & Memory in JavaScript

This topic is extremely important because it connects to many interview areas:

* Why objects behave differently from primitives.
* Why React state updates sometimes fail.
* Why immutability matters.
* Why `===` comparisons behave differently.
* How JavaScript manages memory.

---

## What is a Data Type?

A **data type** defines the kind of value a variable can store and determines what operations can be performed on that value.

```javascript
let age = 25;
let name = "Rocky";
let isDeveloper = true;
```

Here:

* `25` is a Number
* `"Rocky"` is a String
* `true` is a Boolean

JavaScript is a **dynamically typed language**, meaning the type of a value is determined at runtime, not when the code is written.

```javascript
let value = 10;

value = "hello";

value = true;
```

This is valid JavaScript.

The variable itself does not have a fixed type; the value does.

---

# Primitive Data Types

JavaScript has **7 primitive types**.

---

## 1. String

A string represents a sequence of characters.

```javascript
let name = "Rakesh";
let message = 'Hello';
let template = `Welcome`;
```

Strings are **immutable** — you cannot change an existing string value.

```javascript
let name = "John";

name[0] = "M";

console.log(name);
```

Output:

```
John
```

Why?

Because the original string was not modified. A new string must be created instead:

```javascript
name = "Mohn";
```

Now the variable points to a new string.

---

## 2. Number

JavaScript has only one numeric type: `Number`.

It represents:

* Integers
* Floating point numbers
* `Infinity`
* `NaN`

```javascript
let age = 25;

let price = 19.99;

let infinity = Infinity;

let result = NaN;
```

---

### Important Interview Point

Unlike Java or C++, JavaScript does not have:

```
int
float
double
long
```

Everything is:

```javascript
Number
```

(The one exception is `BigInt`, covered below.)

---

### Special Number Values

**NaN** means *Not a Number*.

```javascript
console.log("hello" / 2);
```

Output:

```
NaN
```

But interestingly:

```javascript
typeof NaN
```

returns:

```
"number"
```

This one is **not** a bug. `NaN` is defined by the IEEE 754 floating-point standard as a numeric value that represents an invalid result, so it genuinely belongs to the number type.

Also worth knowing:

```javascript
NaN === NaN
```

Output:

```
false
```

`NaN` is the only value in JavaScript not equal to itself. Use `Number.isNaN(value)` to test for it.

---

## 3. Boolean

Boolean represents logical values. Only two possible values:

```javascript
true
false
```

```javascript
let isLoggedIn = true;

if (isLoggedIn) {
    console.log("Dashboard");
}
```

Used heavily in:

* conditions
* authentication
* UI rendering

React example:

```jsx
{isLoading && <Spinner />}
```

---

## 4. Undefined

`undefined` means a variable has been declared but has not received a value. JavaScript assigns it automatically.

```javascript
let username;

console.log(username);
```

Output:

```
undefined
```

Common cases:

**Declared but not assigned**

```javascript
let user;
```

**Function returns nothing**

```javascript
function test() {

}

console.log(test());
```

Output:

```
undefined
```

---

## 5. Null

`null` represents an **intentional** absence of value.

```javascript
let selectedUser = null;
```

Meaning: "There is currently no user selected."

Compare:

```javascript
let a;

console.log(a);   // undefined  → the value is missing accidentally
```

```javascript
let b = null;

console.log(b);   // null → the absence was intentional
```

---

### Interview Question

Why does `typeof null` return `"object"`?

**Answer:** This *is* a historical bug from the original JavaScript implementation, where values were tagged by their low bits and the null pointer (`0x00`) shared the object tag. It was never fixed because changing it would break existing applications.

---

## 6. Symbol

Symbol creates a unique identifier.

```javascript
const id = Symbol("id");
```

Every symbol is unique, even with the same description:

```javascript
const a = Symbol("user");

const b = Symbol("user");

console.log(a === b);
```

Output:

```
false
```

Used for:

* unique object properties
* avoiding naming conflicts
* library development

---

## 7. BigInt

BigInt allows JavaScript to safely represent integers larger than the Number limit.

```javascript
const bigNumber = 123456789012345678901234567890n;
```

The `n` suffix indicates BigInt.

Normal Number limit:

```javascript
Number.MAX_SAFE_INTEGER
```

Result:

```
9007199254740991
```

Beyond this, precision can be lost.

---

# Primitive vs Reference Types

This is one of the most important JavaScript concepts.

**Primitive Types** — stored directly:

```
String
Number
Boolean
Null
Undefined
Symbol
BigInt
```

**Reference Types** — stored as references:

```
Object
Array
Function
Date
Map
Set
```

---

## Memory Behavior

### Primitive Example

```javascript
let a = 10;

let b = a;

b = 20;

console.log(a);
```

Output:

```
10
```

Why? Because the value was copied.

```
Before:          After:

a → 10           a → 10
b → 10           b → 20
```

They are independent.

---

### Reference Example

```javascript
let user1 = {
    name: "John"
};

let user2 = user1;

user2.name = "Alex";

console.log(user1.name);
```

Output:

```
Alex
```

Why? Because both variables point to the same object.

```
user1 ----\
           Object { name: "John" }
user2 ----/
```

Changing through one reference affects the other.

---

## Equality Comparison

### Primitive Comparison

```javascript
let a = 10;

let b = 10;

console.log(a === b);   // true
```

Values are compared.

---

### Object Comparison

```javascript
const a = {};

const b = {};

console.log(a === b);   // false
```

Why? Because the references are different.

```
a → Object 1

b → Object 2
```

But:

```javascript
const a = {};

const b = a;

console.log(a === b);   // true
```

Because they point to the same object.

---

# Shallow Copy vs Deep Copy

Very important for React.

## Shallow Copy

Copies only the first level.

```javascript
const user = {
    name: "John",
    address: {
        city: "Milan"
    }
};

const copy = {
    ...user
};
```

Now:

```
user !== copy                     // new outer object
user.address === copy.address     // nested object is still shared
```

---

## Deep Copy

Creates a completely independent copy.

```javascript
const copy = structuredClone(user);
```

Now:

```
user.address !== copy.address
```

Note: `structuredClone` cannot clone functions, DOM nodes, or class prototypes — it throws on functions and returns plain objects for class instances.

---

# Why This Matters in React

React relies heavily on reference comparison.

Wrong:

```javascript
state.user.name = "Alex";

setState(state);
```

React may not detect the change because the reference is unchanged.

Correct:

```javascript
setState({
    ...state,
    user: {
        ...state.user,
        name: "Alex"
    }
});
```

New references are created at every level that changed.

---

# Common Interview Questions

### 1. Is JavaScript pass-by-value or pass-by-reference?

JavaScript is **always pass-by-value**. However, for objects, the value being copied is the reference.

```javascript
function change(user) {
    user.name = "Alex";
}

const person = {
    name: "John"
};

change(person);

console.log(person.name);   // "Alex"
```

Because the copied value points to the same object.

But reassigning the parameter does **not** affect the caller:

```javascript
function replace(user) {
    user = { name: "Alex" };
}

replace(person);

console.log(person.name);   // still "John"
```

This is the proof that it is pass-by-value.

---

### 2. Difference between `null` and `undefined`?

* `undefined` means no value assigned.
* `null` means intentional empty value.

---

### 3. Why does React require immutable updates?

Because React compares references to determine whether something changed.

---

# Senior-Level Summary

A strong frontend engineer should be able to explain:

* JavaScript is dynamically typed.
* Primitive values are immutable and copied by value.
* Objects are mutable and accessed through references.
* Equality checks compare values for primitives and references for objects.
* React relies on immutable state updates because reference changes help React detect updates efficiently.
* Shallow copies are not enough for nested state.
* Deep cloning should be used carefully because it can be expensive.

---

# Practice Questions

### Question 1

What will this output?

```javascript
let a = 5;

let b = a;

b = 10;

console.log(a);
```

### Question 2

What will this output?

```javascript
const user1 = {
    name: "John"
};

const user2 = user1;

user2.name = "Alex";

console.log(user1.name);
```

### Question 3

Why does this return `false`?

```javascript
console.log({} === {});
```

### Question 4

Explain the difference:

```javascript
const a = null;

let b;
```

### Question 5

Why is this problematic in React?

```javascript
state.items.push(newItem);

setState(state);
```

---

| ← Previous | Index | Next → |
| ---------- | ----- | ------ |
| [Module 1 — Variables](variables.md) | [Study Plan](../README.md) | [Module 3 — Functions & Closures](functions-closures.md) |
