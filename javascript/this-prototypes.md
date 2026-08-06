[← Study Plan](../README.md)

# Module 5 — `this`, Prototypes & Prototype Chain (Complete Guide)

> **Difficulty:** ⭐⭐⭐⭐⭐
> **Interview Frequency:** ⭐⭐⭐⭐⭐
> **Importance for React Developers:** ⭐⭐⭐⭐☆

---

# Table of Contents

```text
1. What is `this`?
2. How JavaScript Determines `this`
3. Default Binding
4. Implicit Binding
5. Explicit Binding
6. Constructor Binding
7. Arrow Functions & Lexical `this`
8. call(), apply(), bind()
9. Losing `this`
10. Prototype
11. Prototype Chain
12. Constructor Functions
13. Object.create()
14. ES6 Classes
15. Inheritance
16. instanceof
17. Object.getPrototypeOf()
18. Interview Questions
19. React Applications
20. Best Practices
```

---

# Part 1 — Understanding `this`

## Definition

`this` is a special keyword that refers to **the object that is executing the current function**.

The biggest misconception:

> **`this` does NOT refer to the object where the function was defined.**

It refers to **how the function is called**.

This is one of the most important JavaScript rules.

Compare it with scope, from [Module 3](functions-closures.md): scope is **lexical**, fixed when you write the code. `this` is **dynamic**, decided at the call site. Two systems that look similar and behave oppositely — mixing them up is the root of nearly every `this` bug.

---

# Why Does `this` Exist?

Imagine creating multiple users.

Without `this`:

```javascript
const user1 = {
    name: "Rocky"
};

const user2 = {
    name: "Alex"
};

function greet() {
    console.log(user1.name);
}
```

Now the function only works for one object.

With `this`:

```javascript
function greet() {
    console.log(this.name);
}
```

Now the same function can work for any object.

---

# Rule 1 — Default Binding

When a normal function is called by itself:

```javascript
function greet() {
    console.log(this);
}

greet();
```

In browsers:

* Non-strict mode → `window`
* Strict mode → `undefined`

Interviewers love asking this.

The reason is worth one sentence: in sloppy mode a missing `this` gets **substituted** with the global object. Strict mode removed that substitution, so you get `undefined` and the bug announces itself. Every ES module and every `class` body is strict, so `undefined` is the answer that matters for modern code.

---

# Rule 2 — Implicit Binding

If a function is called as an object method:

```javascript
const user = {

    name: "Rocky",

    greet() {
        console.log(this.name);
    }

};

user.greet();
```

Output

```text
Rocky
```

Why?

Because the function was called using:

```javascript
user.greet()
```

The object before the dot becomes `this`.

Only the **last** dot counts:

```javascript
const app = {
    user: {
        name: "Rocky",
        greet() { console.log(this.name); }
    }
};

app.user.greet();   // "Rocky" — `this` is `user`, not `app`
```

---

# Important Interview Trap

```javascript
const user = {

    name: "Rocky",

    greet() {
        console.log(this.name);
    }

};

const fn = user.greet;

fn();
```

Why?

Because

```javascript
fn();
```

is now a standalone function.

The object connection has been lost. Assignment copies the function, not the dot that came before it.

---

## Important Interview Point — what this actually prints

"Output: `undefined`" is the usual answer, and in modern code it is wrong:

| Where the code runs | `this` | Result |
| ------------------- | ------ | ------ |
| ES module / `"use strict"` | `undefined` | **`TypeError: Cannot read properties of undefined (reading 'name')`** — it throws |
| Browser, classic `<script>` | `window` | Logs `""` — `window.name` is a real built-in string, empty by default |
| Node.js CommonJS, sloppy mode | `globalThis` | Logs `undefined` |

So in a React codebase — all ES modules, all strict — this snippet **throws**, it does not quietly log `undefined`. Only the sloppy-mode Node case gives you the textbook answer.

Same trap as the arrow-function `this` case in [Module 3](functions-closures.md). If an interviewer asks "what does it print", the strong answer is "it depends on strict mode — and here's why," not a single word.

---

# Rule 3 — Explicit Binding

Sometimes we choose what `this` should be.

JavaScript provides:

* `call()`
* `apply()`
* `bind()`

---

# call()

## Definition

Invokes a function immediately while specifying `this`.

Example

```javascript
function greet() {
    console.log(this.name);
}

const user = {
    name: "Rocky"
};

greet.call(user);
```

Output

```text
Rocky
```

---

Passing parameters

```javascript
function greet(city) {
    console.log(this.name, city);
}

greet.call(user, "Milan");
```

Output

```text
Rocky Milan
```

---

# apply()

Exactly like `call()`.

Difference:

Arguments are passed as an array.

```javascript
greet.apply(user, ["Milan"]);
```

Mnemonic: **a**pply takes an **a**rray, **c**all takes a **c**omma-separated list.

Spread has made `apply` largely redundant — `greet.call(user, ...args)` does the same job — but it still turns up in older code and in interview questions.

---

# bind()

Unlike `call()`:

It **does not execute immediately**.

It returns a new function.

```javascript
const greetRocky = greet.bind(user);

greetRocky();
```

Output

```text
Rocky
```

---

Interview Question

When would you use `bind()`?

Example:

```javascript
button.addEventListener(
    "click",
    user.greet.bind(user)
);
```

Without binding:

`this` changes — a DOM event handler is called with `this` set to **the element that the listener is attached to**, not to `user`.

---

## Three things about `bind` worth knowing

**1. The binding is permanent.** A bound function cannot be re-bound:

```javascript
function show() { console.log(this.name); }

const bound = show.bind({ name: "A" });

bound.call({ name: "B" });   // "A" — the call() is ignored
```

**2. It does partial application.** Arguments passed to `bind` are locked in ahead of the ones passed at call time:

```javascript
function multiply(a, b) { return a * b; }

const double = multiply.bind(null, 2);

double(5);   // 10
```

**3. It creates a new function every time.** This matters in React:

```jsx
<Child onClick={this.handleClick.bind(this)} />
```

Every render produces a fresh function, so `Child` sees a new prop reference and `React.memo` never prevents a re-render. Same reference-equality rule as [Module 4](objects-arrays.md). Bind once in the constructor, or use a class field arrow.

---

# Rule 4 — Constructor Binding

When using `new`:

```javascript
function User(name){

    this.name=name;

}

const u=new User("Rocky");
```

JavaScript creates:

1. New object
2. Links prototype
3. Sets `this`
4. Returns object

---

## What `new` really does — the senior answer

Written out precisely, `new User("Rocky")`:

1. Creates a brand-new empty object.
2. Sets that object's internal `[[Prototype]]` to `User.prototype`.
3. Calls `User` with `this` bound to the new object.
4. Returns the new object — **unless the constructor explicitly returns an object**, in which case that object wins.

Step 4 is the part most candidates miss:

```javascript
function User(name) {
    this.name = name;
    return { name: "Override" };
}

new User("Rocky").name;   // "Override"
```

Returning a **primitive** is ignored — `return 42` still gives you the new object. Only objects override.

You can write the whole thing yourself, which is the classic whiteboard question:

```javascript
function myNew(Constructor, ...args) {
    const obj = Object.create(Constructor.prototype);
    const result = Constructor.apply(obj, args);
    return typeof result === "object" && result !== null ? result : obj;
}
```

---

# Binding Precedence

When several rules could apply, this is the order — highest wins:

1. **`new`** — `new Fn()`
2. **Explicit** — `call`, `apply`, `bind`
3. **Implicit** — `obj.fn()`
4. **Default** — `fn()`

Arrow functions sit outside this list entirely. They have no `this` of their own, so none of the four rules apply to them — no amount of `call`, `apply`, or `bind` will change an arrow's `this`:

```javascript
const arrow = () => console.log(this);

arrow.call({ name: "Rocky" });   // the `call` is ignored
```

Being able to state this precedence order out loud is a strong senior signal.

---

# Arrow Functions

This is where many developers get confused.

---

## Definition

Arrow functions **do not have their own `this`**.

Instead,

they inherit `this` from the surrounding lexical scope.

---

Normal function

```javascript
const user={

name:"Rocky",

greet:function(){

console.log(this.name);

}

};

user.greet();
```

Output

```text
Rocky
```

---

Arrow

```javascript
const user={

name:"Rocky",

greet:()=>{

console.log(this.name);

}

};

user.greet();
```

Why?

Arrow functions never create their own `this`. An object literal is not a scope, so the arrow reaches straight past `user` to whatever `this` is at the top level of the file.

The output follows exactly the same table as the "losing `this`" trap above — `TypeError` in an ES module, `""` in a browser script, `undefined` in Node CommonJS. [Module 3](functions-closures.md) works through this case in full.

The rule to take away: **never use an arrow function for a method that needs `this`.** Use shorthand method syntax.

---

# Why React Uses Arrow Functions

Example

```javascript
class User{

handleClick=()=>{

console.log(this);

}

}
```

Without arrow functions:

You often needed

```javascript
this.handleClick=this.handleClick.bind(this);
```

in the constructor.

Arrow functions simplified this.

---

## The trade-off

A class field arrow is created **per instance**, not shared on the prototype. Ten thousand instances means ten thousand copies of that function. A prototype method is one function shared by all of them — which is the whole point of Part 2 below.

For React components, where you have a handful of instances, this cost is irrelevant and the ergonomics win. For a class you instantiate in bulk, prefer a prototype method and bind once.

---

# Losing `this`

Common interview question.

```javascript
const user={

name:"Rocky",

greet(){

console.log(this.name);

}

};

setTimeout(user.greet,1000);
```

Why?

Because `setTimeout` executes the function later as a standalone callback. Only the function was passed — the `user.` in front of it was never part of the value.

Correct

```javascript
setTimeout(()=>user.greet(),1000);
```

or

```javascript
setTimeout(user.greet.bind(user),1000);
```

---

## Important Interview Point

`setTimeout` is not quite the same as a bare `fn()` call, because the host environment supplies its own `this`:

* **In a browser**, timer callbacks are invoked with `this` set to `window` — even for a function defined in strict mode, because strict mode only stops *substitution*, it can't stop a caller from passing a `this` explicitly. So this logs `""` (`window.name` again), not `undefined`.
* **In Node.js**, `this` is the `Timeout` object that `setTimeout` returns, so `this.name` is `undefined`.

Either way the teaching point holds — `this` is not `user` — but "it prints `undefined`" is only right in Node.

---

# Part 2 — Prototype

---

# What is a Prototype?

Definition:

Every JavaScript object has an internal link to another object called its **prototype**.

Instead of copying methods into every object,

JavaScript shares them.

---

Without prototype

Imagine

1000 users.

```javascript
const user1={
greet(){...}
};

const user2={
greet(){...}
};

const user3={
greet(){...}
};
```

Memory wasted.

---

With prototype

```
user1

 \
user2 -----> shared greet()

 /

user3
```

One function.

Many objects.

Much more efficient.

---

# Constructor Function

```javascript
function User(name){

this.name=name;

}
```

Every constructor automatically gets:

```javascript
User.prototype
```

---

Adding methods

```javascript
User.prototype.greet=function(){

console.log("Hello");

};
```

Now

```javascript
const u1=new User("Rocky");

const u2=new User("Alex");
```

Both use

the same `greet()`.

```javascript
u1.greet === u2.greet;   // true — one function, not two
```

---

## `prototype` vs `[[Prototype]]`

Two different things with confusingly similar names, and interviewers probe this:

* **`User.prototype`** — a property on the *constructor function*. It is the object that will become the prototype of instances created with `new User()`.
* **`u1.__proto__`** (properly: `Object.getPrototypeOf(u1)`) — the *instance's* actual internal link.

The relationship:

```javascript
Object.getPrototypeOf(u1) === User.prototype;   // true
```

Only functions that can be constructed have a `.prototype` property. Arrow functions and shorthand object methods do not — which is exactly why neither can be called with `new` ([Module 3](functions-closures.md)).

---

# Prototype Chain

Suppose

```javascript
const arr=[1,2,3];
```

When JavaScript sees

```javascript
arr.map()
```

Where does

`map`

come from?

Not from the array itself.

It searches.

```
arr

↓

Array.prototype

↓

Object.prototype

↓

null
```

This search path is called the **Prototype Chain**.

`null` at the end is what terminates the search — that's why an unknown property returns `undefined` instead of looping forever.

---

# Property Lookup

Example

```javascript
const user={

name:"Rocky"

};

console.log(user.name);
```

Found immediately.

---

But

```javascript
user.toString();
```

There is no

```javascript
toString
```

inside

user.

JavaScript searches

```
user

↓

Object.prototype

↓

toString()
```

Found.

---

## Shadowing — and one important asymmetry

Reading walks the chain. **Writing does not.** An assignment always creates an own property on the object itself:

```javascript
u1.greet = function () { console.log("Custom"); };

u1.greet();   // "Custom"  — own property shadows the prototype
u2.greet();   // "Hello"   — untouched
```

This is why `this` matters so much in prototype methods: the method lives on the shared prototype, but `this.name` resolves to the individual instance that called it.

---

# Object.create()

Creates an object with a chosen prototype.

```javascript
const animal={

eat(){

console.log("Eating");

}

};

const dog=Object.create(animal);

dog.eat();
```

Output

```text
Eating
```

This is the prototype chain with no constructors and no classes involved — just one object delegating to another.

---

# ES6 Classes

Most developers think

classes introduced inheritance.

Not true.

Classes are syntactic sugar.

This

```javascript
class User{

greet(){

console.log("Hello");

}

}
```

Internally behaves almost like

```javascript
function User(){}

User.prototype.greet=function(){

console.log("Hello");

};
```

---

## Where "almost" matters

"Syntactic sugar" is the right headline, but a senior answer names the differences:

| | Constructor function | `class` |
| --- | --- | --- |
| Called without `new` | Runs, silently pollutes global or throws | **`TypeError`**, always |
| Hoisting | Fully hoisted, callable before definition | In the TDZ — `ReferenceError` before its line |
| Strict mode | Follows the surrounding file | Class bodies are **always** strict |
| Methods in `for...in` | Enumerable, so they show up | **Non-enumerable**, so they don't |
| Private state | Convention only (`_name`) | Real `#private` fields |

The last two are observable in code, not just theory — which is what makes this a good interview question rather than trivia.

---

# Inheritance

```javascript
class Animal{

eat(){

console.log("Eat");

}

}

class Dog extends Animal{

bark(){

console.log("Woof");

}

}
```

Dog inherits

eat()

through the prototype chain.

The chain for `new Dog()`:

```
dog → Dog.prototype → Animal.prototype → Object.prototype → null
```

`extends` does nothing magic — it links those prototypes together.

---

## `super`

If a subclass declares its own constructor, it **must** call `super()` before touching `this`:

```javascript
class Dog extends Animal {

    constructor(name) {
        super();          // required first
        this.name = name;
    }

}
```

Skipping it throws `ReferenceError: Must call super constructor ... before accessing 'this'`. The reason is that in a derived class, `this` doesn't exist until the parent constructor creates it.

`super.method()` also calls the parent's version from an override — the standard way to extend behaviour rather than replace it.

---

# instanceof

Checks prototype chain.

```javascript
const arr=[];

console.log(arr instanceof Array);
```

Output

```text
true
```

Because

```
Array.prototype

exists

inside

arr's prototype chain.
```

---

## Where `instanceof` fails

It compares prototype **identity**, so it breaks across realms. An array created inside an `<iframe>` or a worker has a *different* `Array.prototype`:

```javascript
iframeArray instanceof Array;    // false
Array.isArray(iframeArray);      // true
```

That is the real reason `Array.isArray` exists ([Module 4](objects-arrays.md)). Same caution applies to `instanceof Error` across module or realm boundaries.

---

# Object.getPrototypeOf()

Useful interview topic.

```javascript
Object.getPrototypeOf(arr);
```

Returns

```javascript
Array.prototype
```

Use this rather than `__proto__`, which is a legacy accessor kept only for web compatibility. There is a setter too, `Object.setPrototypeOf()` — but changing an object's prototype after creation forces engines to deoptimise it, so build the object with the right prototype instead.

---

# Common Interview Questions

## Beginner

* What is `this`?
* Difference between normal and arrow functions?
* What is a prototype?

---

## Intermediate

* Explain prototype chain.
* Difference between `call`, `apply`, and `bind`.
* Why does `setTimeout` lose `this`?
* Explain constructor functions.

---

## Senior

* Explain how `new` works internally.
* Why are classes syntactic sugar?
* Explain property lookup.
* Explain method sharing through prototypes.
* Explain prototype inheritance.
* Explain lexical `this`.
* What is the precedence order of the binding rules?
* Why does `instanceof` fail across iframes?

---

# React Connection

Even though modern React primarily uses functional components, understanding `this` is still valuable because:

* You'll encounter class components in legacy codebases.
* Third-party libraries and plain JavaScript often rely on `this`.
* Understanding lexical scoping helps explain why arrow functions behave the way they do.

However, for modern React interviews, **closures, hooks, state management, rendering, and asynchronous behavior** are tested much more frequently than deep prototype questions. You should know prototypes well, but don't spend disproportionate time memorizing obscure prototype edge cases.

One thing worth noticing: hooks exist partly *because* `this` was the main source of friction in class components. `useState` replaces `this.state`, and a closure over a `const` replaces `this.handleClick.bind(this)`. Function components trade a dynamic-`this` problem for a stale-closure problem — the trade is real, and being able to describe it that way lands well in an interview.

---

# Best Practices

✅ Prefer arrow functions for callbacks when you want lexical `this`.

✅ Use `class` syntax rather than constructor functions for new object-oriented code.

✅ Avoid manually modifying built-in prototypes like `Array.prototype` or `Object.prototype` in application code.

✅ Understand the prototype chain, but don't overuse inheritance. Modern JavaScript often favors composition over inheritance.

❌ Never use an arrow function as an object method or a prototype method that needs `this`.

---

# Senior-Level Summary

By the end of this module, you should be able to explain:

* `this` is determined by **how a function is called**, not where it's defined.
* The four binding rules and their precedence: `new` > explicit > implicit > default.
* Arrow functions inherit `this` from their surrounding lexical scope and cannot be rebound.
* `call()`, `apply()`, and `bind()` provide explicit control over `this`.
* Every object has an internal prototype link, and `Constructor.prototype` is not the same thing as an instance's `[[Prototype]]`.
* JavaScript resolves properties by walking the prototype chain, but assignment always writes an own property.
* ES6 classes are built on top of the existing prototype system — with real differences around `new`, hoisting, strict mode, and enumerability.
* `new` creates an object, links its prototype, binds `this`, and returns the new object unless the constructor returns a different object.
* Method sharing through prototypes improves memory efficiency compared to duplicating methods on every object.

---

# Practice Questions

Answer these without running them:

### 1. What are the three possible outputs, and what decides which one you get?

```javascript
const user = {
    name: "Rocky",
    greet() { console.log(this.name); }
};

const fn = user.greet;

fn();
```

---

### 2. What does this log?

```javascript
function Person(name) {
    this.name = name;
    return { name: "Someone else" };
}

console.log(new Person("Rocky").name);
```

---

### 3. Predict the output and explain the rule that produces it:

```javascript
const obj = {
    name: "Rocky",
    regular() { return this.name; },
    arrow: () => this?.name
};

console.log(obj.regular.call({ name: "A" }));
console.log(obj.arrow.call({ name: "B" }));
```

---

### 4. Why is `u1.greet === u2.greet` true for a prototype method but false for a class field arrow? When would that difference actually matter?

---

### 5. Explain:

A team lead says "just use arrow functions everywhere, they fix `this`." Give one concrete case where that advice produces a bug.

---

## Up Next — Module 6: Asynchronous JavaScript, Event Loop & Concurrency

This is one of the most important modules for frontend interviews and will cover:

* Synchronous vs asynchronous execution
* Web APIs
* Call Stack
* Callback Queue
* Microtask Queue
* Event Loop
* Promises
* `async`/`await`
* `Promise.all`, `allSettled`, `race`, `any`
* Fetch API
* `AbortController`
* Error handling
* Retry strategies
* Debouncing & throttling
* Common async interview questions
* React-specific async patterns (effects, cleanup, race conditions)

This is also the module where many "predict the output" interview questions come from, so we'll go into considerable depth.

---

| ← Previous | Index | Next → |
| ---------- | ----- | ------ |
| [Module 4 — Objects & Arrays](objects-arrays.md) | [Study Plan](../README.md) | [Module 6 — Async JavaScript & the Event Loop](async-event-loop.md) |
