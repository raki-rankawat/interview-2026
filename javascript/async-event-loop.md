[← Study Plan](../README.md)

# Module 6 — Asynchronous JavaScript, Event Loop & Concurrency

> **Difficulty:** ⭐⭐⭐⭐⭐
> **Interview Frequency:** ⭐⭐⭐⭐⭐
> **Importance for React Developers:** ⭐⭐⭐⭐⭐

---

# Table of Contents

```text
1. Synchronous vs Asynchronous JavaScript
2. Single-Threaded Nature of JavaScript
3. Blocking vs Non-Blocking Code
4. Runtime Environment
5. Call Stack
6. Web APIs
7. Callback Queue
8. Microtask Queue
9. Event Loop
10. Callbacks
11. Callback Hell
12. Promises
13. Promise States
14. Promise Chaining
15. async / await
16. Error Handling
17. Promise Combinators
18. Fetch API
19. AbortController
20. Retry & Exponential Backoff
21. Debouncing & Throttling
22. React Applications
23. Common Interview Questions
```

---

# Part 1 — Synchronous JavaScript

## What is Synchronous Execution?

### Definition

Synchronous code executes **one statement at a time**, in the exact order it appears.

JavaScript does not move to the next line until the current one has finished executing.

Example:

```javascript
console.log("A");

console.log("B");

console.log("C");
```

Output:

```text
A
B
C
```

Execution flow:

```text
Line 1

↓

Line 2

↓

Line 3
```

Very straightforward.

---

# Why Does JavaScript Behave This Way?

JavaScript was designed to be:

* Simple
* Predictable
* Easy to embed inside browsers

It executes using a **single Call Stack**.

Only one piece of JavaScript code can execute at a time.

This means:

```text
Task A

↓

Task B

↓

Task C
```

Never

```text
Task A
      \
Task B  Task C
```

No true parallel execution on the main thread.

---

# Single-Threaded JavaScript

## Definition

JavaScript has **one main execution thread**.

It executes only one task at a time.

Imagine a cashier.

```text
Customer 1

↓

Customer 2

↓

Customer 3
```

Only one customer is served at once.

---

## Why Is This Important?

Suppose JavaScript had multiple threads modifying the DOM simultaneously.

Example:

Thread A:

```javascript
button.textContent = "Save";
```

Thread B:

```javascript
button.remove();
```

Which happens first?

Race conditions become common.

Keeping JavaScript single-threaded avoids many synchronization problems.

---

## Important Interview Point — single-threaded ≠ one thing at a time

Be precise about what is single-threaded. **Your JavaScript** runs on one thread. The browser itself is heavily multi-threaded — networking, timers, rendering, and compositing all run elsewhere. That's why ten `fetch` calls genuinely happen in parallel even though your code never runs in parallel.

If you truly need parallel *JavaScript*, that's what **Web Workers** are for. They run on separate threads with no shared memory and no DOM access, communicating by message passing — which is exactly how the language keeps its "no data races" guarantee.

---

# Blocking Code

## Definition

Blocking code prevents JavaScript from executing the next statement.

Example:

```javascript
function heavyTask() {

    for(let i=0;i<1000000000;i++){}

}

heavyTask();

console.log("Done");
```

Output:

```text
Done
```

But only after the loop finishes.

During that loop:

* UI freezes
* Buttons don't respond
* Animations stop
* Scrolling becomes laggy

The reason is one sentence long, and it's worth saying in an interview: **rendering is a task on the same thread.** The browser cannot paint while your function is on the stack.

---

# Non-Blocking Code

Example:

```javascript
setTimeout(() => {

    console.log("Finished");

},1000);

console.log("Running");
```

Output:

```text
Running
Finished
```

The timer does not block JavaScript.

---

# Important Question

If JavaScript is single-threaded...

How can timers work?

How can network requests happen?

How can animations continue?

Answer:

JavaScript itself doesn't do those things.

The **runtime environment** does.

---

# Runtime Environment

JavaScript runs inside environments such as:

* Browser
* Node.js
* Deno

These environments provide additional features.

For browsers:

```text
JavaScript Engine

+

Browser APIs
```

Browser APIs include:

* setTimeout
* DOM
* Fetch API
* Local Storage
* Geolocation
* WebSocket
* IndexedDB

These are **not part of the JavaScript language itself**.

A good way to prove this to yourself: `setTimeout` and `fetch` are not in the ECMAScript specification at all. They live in the HTML spec. The language provides Promises and the job queue; the *host* provides everything that can actually wait for something.

---

# Call Stack

## Definition

The Call Stack is a **Last-In, First-Out (LIFO)** data structure used to track function execution.

Example:

```javascript
function one(){

    two();

}

function two(){

    three();

}

function three(){

    console.log("Done");

}

one();
```

Execution:

```text
Global

↓

one()

↓

two()

↓

three()
```

Then it unwinds:

```text
three()

↓

two()

↓

one()

↓

Global
```

The stack must become empty before queued asynchronous callbacks can execute.

Covered in more depth in [Module 3](functions-closures.md).

---

# Browser APIs

Consider:

```javascript
setTimeout(() => {

    console.log("Hello");

},2000);
```

What happens?

1. `setTimeout` is pushed onto the Call Stack.
2. The browser starts a 2-second timer.
3. JavaScript continues immediately.
4. After 2 seconds, the callback is placed into the Callback Queue.
5. The Event Loop waits until the Call Stack is empty.
6. The callback is pushed onto the Call Stack and executed.

---

Visualization:

```text
Call Stack

↓

setTimeout()

↓

Browser Timer

↓

Callback Queue

↓

Event Loop

↓

Call Stack
```

---

## Important Interview Point — the delay is a minimum, not a promise

`setTimeout(fn, 1000)` means "not before 1000ms", never "at exactly 1000ms". If the stack is busy when the timer fires, the callback waits. This is why timers are unsuitable for animation — use `requestAnimationFrame`, which the browser calls right before it paints.

There is also a floor: after five levels of nested timers, browsers clamp the delay to a **minimum of ~4ms**, so `setTimeout(fn, 0)` in a recursive chain is really `setTimeout(fn, 4)`.

---

# Callback Queue (Macrotask Queue)

Stores completed asynchronous tasks like:

* `setTimeout`
* `setInterval`
* DOM events
* MessageChannel

Example:

```javascript
setTimeout(() => {

    console.log("Timer");

},0);
```

Even with `0`, it **does not execute immediately**.

It waits until:

* the timer completes
* the Call Stack is empty
* all Microtasks are finished

---

# Microtask Queue

Higher priority than the Callback Queue.

Contains:

* Promise callbacks (`.then`, `.catch`, `.finally`)
* `queueMicrotask`
* `MutationObserver`

Example:

```javascript
Promise.resolve().then(() => {

    console.log("Promise");

});
```

This goes into the Microtask Queue.

---

# Event Loop

## Definition

The Event Loop continuously checks:

1. Is the Call Stack empty?
2. Are there Microtasks?
3. Execute all Microtasks.
4. Execute one Macrotask.
5. Repeat.

Priority:

```text
Call Stack

↓

Microtasks

↓

Macrotasks
```

---

## The precise version

Stated the way the HTML spec orders it, one turn of the loop is:

```text
1. Take ONE macrotask from the queue and run it to completion
2. Drain the ENTIRE microtask queue
3. Render if it's time to (requestAnimationFrame → style → layout → paint)
4. Repeat
```

Two consequences that explain most trick questions:

**Your initial script is itself a macrotask.** That's why every microtask waits until the whole synchronous file has finished — not until the current line finishes.

**"Drain the entire queue" means microtasks added *by* microtasks also run in the same pass.** One macrotask, then *all* microtasks, however many appear.

---

## Important Interview Point — microtasks can starve the loop

```javascript
function loop() {
    Promise.resolve().then(loop);
}

loop();
```

This freezes the tab permanently. The microtask queue never empties, so the browser never reaches the render step.

The macrotask version does **not** freeze:

```javascript
function loop() {
    setTimeout(loop, 0);
}

loop();
```

Only one runs per turn, so rendering still gets its slot between them. If you ever need to yield to the browser mid-work, this is the difference that matters — and it's the reason React's scheduler is built on macrotasks (`MessageChannel`), not promises.

---

# Classic Interview Question

```javascript
console.log("A");

setTimeout(() => {

    console.log("B");

},0);

Promise.resolve().then(() => {

    console.log("C");

});

console.log("D");
```

Output:

```text
A
D
C
B
```

Explanation:

1. `A`
2. Timer registered
3. Promise scheduled as Microtask
4. `D`
5. Call Stack empty
6. Microtasks → `C`
7. Macrotasks → `B`

This exact pattern appears in interviews frequently.

---

## The harder version

Same idea, but with `async`/`await` and a nested microtask:

```javascript
console.log("1");

setTimeout(() => console.log("2"), 0);

(async () => {
    console.log("3");
    await null;
    console.log("4");
})();

Promise.resolve().then(() => {
    console.log("5");
    Promise.resolve().then(() => console.log("6"));
});

console.log("7");
```

Output:

```text
1
3
7
4
5
6
2
```

Walk it through:

* `1` — synchronous.
* `3` — the body of an `async` function runs **synchronously** until it hits the first `await`.
* `7` — synchronous; the script is still one macrotask.
* `4` — `await null` queued the rest of the function as a microtask, and it was queued first.
* `5` — the next microtask.
* `6` — queued *during* microtask processing, and still drained in the same pass.
* `2` — the macrotask, last.

If you can narrate that, you can answer almost any version of this question.

---

# Callbacks

A callback is a function passed into another function to be executed later.

Example:

```javascript
function greet(callback){

    console.log("Hello");

    callback();

}

greet(() => {

    console.log("World");

});
```

Output:

```text
Hello
World
```

Note that this callback is **synchronous** — "callback" and "asynchronous" are not synonyms. `array.map(fn)` takes a callback and never leaves the stack.

---

# Callback Hell

Before Promises, asynchronous code often looked like:

```javascript
login(user, function(){

    getProfile(function(){

        getOrders(function(){

            getPayments(function(){

                // ...

            });

        });

    });

});
```

Problems:

* Deep nesting
* Hard to read
* Hard to debug
* Error handling is difficult

Promises were introduced to solve this.

The error-handling problem is the serious one. `try/catch` cannot catch anything thrown inside an asynchronous callback, because by the time it runs, the `try` block is long gone from the stack:

```javascript
try {
    setTimeout(() => { throw new Error("boom"); }, 0);
} catch (err) {
    // never runs
}
```

Every callback had to handle its own errors, which is where the `function (err, result)` convention came from.

---

# Promises

## Definition

A Promise is an object representing the eventual completion (or failure) of an asynchronous operation.

Think of ordering food.

* Order placed → Pending
* Food delivered → Fulfilled
* Restaurant cancels → Rejected

---

## Promise States

Every Promise has one of three states:

```text
Pending

↓

Fulfilled

or

Rejected
```

A settled promise (fulfilled or rejected) cannot change state again.

---

## Creating a Promise

```javascript
const promise = new Promise((resolve, reject) => {

    const success = true;

    if(success){

        resolve("Success");

    }else{

        reject("Failure");

    }

});
```

---

## Important Interview Point — the executor runs synchronously

The function you pass to `new Promise` executes **immediately**, on the current stack. Only the `.then` callbacks are deferred:

```javascript
console.log("1");

new Promise((resolve) => {
    console.log("2");        // synchronous
    resolve();
}).then(() => console.log("4"));

console.log("3");
```

Output:

```text
1
2
3
4
```

People assume the whole thing is deferred and predict `1 3 2 4`. It isn't. A promise doesn't make code asynchronous — it gives you a handle on something that already is.

---

## Consuming a Promise

```javascript
promise
    .then(result => {

        console.log(result);

    })
    .catch(error => {

        console.error(error);

    })
    .finally(() => {

        console.log("Finished");

    });
```

---

# Promise Chaining

Instead of nesting:

```javascript
login()
    .then(getProfile)
    .then(getOrders)
    .then(getPayments)
    .catch(handleError);
```

This is flatter and easier to maintain.

---

## What makes chaining work

`.then()` always returns a **new promise**. Whatever you return from the callback becomes that promise's value — and if you return a promise, the chain waits for it.

Which is why the single most common chaining bug is a missing `return`:

```javascript
// ❌ the chain does not wait; `orders` is undefined
getProfile()
    .then(profile => { getOrders(profile.id); })
    .then(orders => console.log(orders));

// ✅
getProfile()
    .then(profile => getOrders(profile.id))
    .then(orders => console.log(orders));
```

Arrow functions with a body `{ }` swallow the value; the concise form returns it. Two characters, completely different behaviour.

A `.catch()` at the end catches a rejection from **any** earlier step, which is the flat error handling callbacks never had. `.finally()` runs either way and passes the value straight through — it can't change the result.

---

# `async` / `await`

## Definition

`async`/`await` is syntactic sugar over Promises.

It allows asynchronous code to look synchronous.

Example:

```javascript
async function loadUser() {

    const response = await fetch("/api/user");

    const user = await response.json();

    console.log(user);

}
```

Under the hood, it still uses Promises.

---

## Two rules that explain everything else

**1. An `async` function always returns a promise** — even if you return a plain value, and even if you return nothing:

```javascript
async function f() { return 1; }

f();          // Promise { 1 }
await f();    // 1
```

Throwing inside an `async` function rejects that promise rather than throwing to the caller.

**2. `await` suspends the function and queues the rest as a microtask.** The code *after* an `await` is a `.then` callback wearing different clothes:

```javascript
async function run() {
    console.log("1");
    await null;          // yields, even though null isn't a promise
    console.log("3");
}

run();
console.log("2");
```

Output:

```text
1
2
3
```

---

## Sequential vs concurrent — the performance question

`await` inside a loop is **sequential**. Three one-second requests take three seconds:

```javascript
// ❌ 3 seconds
const users = [];

for (const id of ids) {
    users.push(await fetchUser(id));
}
```

Start them all, then wait once — one second:

```javascript
// ✅ 1 second
const users = await Promise.all(ids.map(id => fetchUser(id)));
```

Use the sequential form only when each request genuinely depends on the previous result. Spotting this in a code review is a standard senior interview task.

---

# Error Handling

With Promises:

```javascript
fetch("/api")
    .then(...)
    .catch(err => {
        console.error(err);
    });
```

With `async`/`await`:

```javascript
async function load() {

    try {

        const response = await fetch("/api");

        const data = await response.json();

        console.log(data);

    } catch (err) {

        console.error(err);

    }

}
```

---

## Important Interview Point

`try/catch` only catches promises you actually **`await`**. Forget the keyword and the rejection escapes:

```javascript
try {
    fetchUser();          // ❌ not awaited — catch never fires
} catch (err) { }
```

An unhandled rejection fires the `unhandledrejection` event, logs a console error, and in Node crashes the process by default.

The one to watch for with `Promise.all` is subtler: if a second promise rejects *after* the first rejection has already been handled, nothing is listening to it any more. `Promise.allSettled` avoids this entirely.

---

# Promise Combinators

These are common interview questions.

| Method                 | Purpose                            | Rejects Early?             |
| ---------------------- | ---------------------------------- | -------------------------- |
| `Promise.all()`        | Wait for all promises              | ✅ Yes                      |
| `Promise.allSettled()` | Wait for all, regardless of result | ❌ No                       |
| `Promise.race()`       | First promise to settle wins       | Depends                    |
| `Promise.any()`        | First fulfilled promise wins       | Rejects only if all reject |

### `Promise.all()`

```javascript
const [users, posts] = await Promise.all([
    fetchUsers(),
    fetchPosts()
]);
```

Runs requests concurrently and waits for both.

---

## Details worth knowing

**`Promise.all` does not cancel anything.** On the first rejection it rejects immediately, but the other requests keep running to completion — you just stop listening. There is no cancellation in the Promise API at all; that's what `AbortController` is for.

**`Promise.allSettled` never rejects.** It resolves with a status object per entry, which is what you want for "load six widgets, show whichever succeed":

```javascript
const results = await Promise.allSettled([a(), b(), c()]);

results.forEach(r => {
    if (r.status === "fulfilled") console.log(r.value);
    else console.error(r.reason);
});
```

**`Promise.race` settles on the first *settled* promise** — fulfilled *or* rejected. The classic use is a timeout:

```javascript
await Promise.race([
    fetchData(),
    new Promise((_, reject) =>
        setTimeout(() => reject(new Error("Timeout")), 5000))
]);
```

**`Promise.any` ignores rejections** until they all fail, then rejects with an `AggregateError` whose `.errors` array holds every reason. Naming that error type is the detail interviewers are listening for.

---

# Fetch API

`fetch()` returns a Promise.

Example:

```javascript
const response = await fetch("/api/users");

if (!response.ok) {
    throw new Error("Request failed");
}

const users = await response.json();
```

Important: `fetch` only rejects on network failures. A `404` or `500` still resolves successfully—you must check `response.ok`.

This is the most common `fetch` bug in real code. A 500 with an HTML error page flows straight into `.json()`, which then throws a confusing *parse* error instead of the HTTP error that actually happened. `fetch` rejects only for network failure, CORS rejection, or an abort — never for a status code.

---

# AbortController

Used to cancel requests.

```javascript
const controller = new AbortController();

fetch("/api/users", {
    signal: controller.signal
});

// Later...
controller.abort();
```

React example:

```javascript
useEffect(() => {
    const controller = new AbortController();

    fetch("/api/users", {
        signal: controller.signal
    });

    return () => controller.abort();
}, []);
```

This helps avoid race conditions and unnecessary work.

---

## One thing that example is missing

Aborting **rejects** the fetch promise with an `AbortError`. With no `.catch`, every unmount produces an unhandled rejection in the console. The complete version:

```javascript
useEffect(() => {
    const controller = new AbortController();

    fetch("/api/users", { signal: controller.signal })
        .then(res => {
            if (!res.ok) throw new Error(res.status);
            return res.json();
        })
        .then(setUsers)
        .catch(err => {
            if (err.name === "AbortError") return;   // expected on unmount
            setError(err);
        });

    return () => controller.abort();
}, []);
```

`AbortController` also works for anything else that accepts a signal, including `addEventListener` — which is a tidy way to remove many listeners at once:

```javascript
window.addEventListener("resize", onResize, { signal: controller.signal });
```

---

# Retry with Exponential Backoff

Instead of retrying immediately:

```text
Attempt 1

↓

1 second

↓

Attempt 2

↓

2 seconds

↓

Attempt 3

↓

4 seconds
```

Useful for temporary network failures.

Do **not** blindly retry every error. Validation errors (`400`) usually shouldn't be retried.

---

## Implementation

```javascript
async function retry(fn, { attempts = 3, baseDelay = 1000 } = {}) {

    for (let attempt = 0; attempt < attempts; attempt++) {

        try {
            return await fn();
        } catch (err) {

            const isLast = attempt === attempts - 1;
            if (isLast || !isRetryable(err)) throw err;

            const delay = baseDelay * 2 ** attempt;
            const jitter = Math.random() * 200;

            await new Promise(r => setTimeout(r, delay + jitter));
        }
    }
}
```

Three things to be able to justify:

* **Retry only what can succeed on a retry** — network errors, `429`, `502`, `503`, `504`. Never a `400` or `401`; the answer will not change.
* **Jitter is not decoration.** Without it, every client that failed during an outage retries at exactly the same moment and knocks the server over again on recovery. This is the "thundering herd", and it's the part candidates forget.
* **Only retry idempotent requests.** Retrying a `POST /payments` after a timeout can charge someone twice — the response was lost, not necessarily the request.

---

# Debouncing

Delays execution until the user stops triggering an action.

Example:

Search input.

Without debounce:

```text
R

Ra

Rak

Rake

Rakesh
```

Five API calls.

With debounce (300 ms):

One API call after typing pauses.

---

## Implementation

```javascript
function debounce(fn, delay) {

    let timeoutId;

    return function (...args) {
        clearTimeout(timeoutId);
        timeoutId = setTimeout(() => fn.apply(this, args), delay);
    };
}
```

Every call cancels the pending timer and starts a new one, so `fn` runs only after `delay` of silence.

Two details that get probed: the returned function is a **regular** function, not an arrow, so it can forward `this` — and `fn.apply(this, args)` is why ([Module 5](this-prototypes.md)). And `timeoutId` survives between calls because of the closure ([Module 3](functions-closures.md)). "Implement debounce" is really a closure question wearing a timer costume.

---

# Throttling

Limits execution to a maximum rate.

Example:

Scroll event.

Without throttle:

```text
100+

events/sec
```

With throttle:

```text
One event every

200ms
```

Common for:

* Scroll
* Resize
* Mouse movement

---

## Implementation

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

This is the **leading edge** version — it fires immediately, then ignores calls for `limit` ms.

---

## The distinction to state in an interview

* **Debounce** — "wait until it stops." Runs **once**, at the end. Search inputs, autosave, resize-then-recalculate.
* **Throttle** — "at most once per interval." Runs **repeatedly** at a fixed rate. Scroll position, drag, progress bars, rate-limited APIs.

Ask yourself whether you need the *final* value or a *steady stream* of values. That answers which one every time.

---

# React Applications

These concepts appear everywhere in React:

* `useEffect` for async data fetching
* Cleaning up requests with `AbortController`
* Preventing race conditions
* Debouncing search inputs
* Running independent requests concurrently with `Promise.all`
* Understanding why state updates don't happen immediately after an `await`

---

## Race conditions

Type "ab" quickly and you fire two requests. If the response for "a" arrives *after* the response for "ab", the stale result wins and the UI shows the wrong list. Nothing errors — the data is just quietly wrong.

React's recommended fix is an ignore flag in the cleanup function:

```javascript
useEffect(() => {

    let ignore = false;

    fetchResults(query).then(data => {
        if (!ignore) setResults(data);
    });

    return () => { ignore = true; };

}, [query]);
```

Cleanup runs before every re-run of the effect, so only the newest request is allowed to write state. `AbortController` does the same job and additionally stops the wasted network work — either is a good answer, and mentioning both is better.

---

## Why state looks stale after an `await`

```javascript
async function handleClick() {
    setCount(count + 1);
    await fetchSomething();
    console.log(count);        // still the OLD value
}
```

`count` is a `const` captured by the closure of the render that created this handler. Awaiting doesn't refresh it — that binding can never change. This is the stale closure from [Module 3](functions-closures.md), and the fix is the same: `setCount(prev => prev + 1)`, or a ref when you need to *read* a current value.

---

## Batching

In React 18+, state updates are batched **automatically everywhere** — including inside promises, `setTimeout`, and native event handlers. Before 18, batching only happened inside React event handlers, so two `setState` calls after an `await` caused two renders.

That's a genuinely good thing to know the version boundary for, because plenty of blog posts still describe the old behaviour.

---

# Common Interview Questions

### Beginner

* What is asynchronous JavaScript?
* What is a callback?
* What is a Promise?
* Difference between synchronous and asynchronous code?

### Intermediate

* Explain the Event Loop.
* Difference between Microtasks and Macrotasks.
* Difference between `async`/`await` and `.then()`.
* Explain `Promise.all()` vs `Promise.allSettled()`.

### Senior

* Why does `fetch()` not reject on HTTP 404?
* How would you prevent duplicate API requests?
* How do you avoid race conditions in React?
* When would you use `AbortController`?
* Explain debouncing vs throttling.
* How would you implement retries safely?
* What happens when multiple Promises resolve at the same time?
* Why does an infinite microtask loop freeze the page when an infinite `setTimeout` loop doesn't?

---

# Senior-Level Summary

By the end of this module, you should be able to explain:

* JavaScript is single-threaded, but browsers provide asynchronous capabilities through runtime APIs.
* The Call Stack executes one task at a time.
* Browser APIs handle timers, network requests, and DOM events.
* The Event Loop coordinates execution between the Call Stack, Microtask Queue, and Macrotask Queue — one macrotask, then the microtask queue drained completely, then render.
* Promise callbacks are Microtasks and run before timer callbacks.
* The `new Promise` executor runs synchronously; only the `.then` callbacks are deferred.
* `async`/`await` is syntax built on top of Promises — an `async` function always returns one, and the code after an `await` is a microtask.
* `await` in a loop is sequential; `Promise.all` is concurrent.
* `fetch()` resolves for HTTP errors and only rejects for network failures.
* `AbortController` is essential for canceling in-flight requests — and its rejection needs handling.
* Debouncing and throttling solve different performance problems.
* Production-ready async code requires cancellation, retries where appropriate, and proper error handling.

---

# Practice Questions

Answer these without running them:

### 1. Predict the exact output order:

```javascript
console.log("start");

setTimeout(() => console.log("timeout"), 0);

new Promise((resolve) => {
    console.log("executor");
    resolve();
}).then(() => console.log("then"));

console.log("end");
```

---

### 2. What is logged, and how long does it take?

```javascript
const ids = [1, 2, 3];

async function load() {
    const results = [];
    for (const id of ids) {
        results.push(await fetchUser(id));   // each takes 1s
    }
    return results;
}
```

Rewrite it to finish in ~1 second, and say when you should *not* make that change.

---

### 3. Why does this `catch` never fire, and what are two ways to fix it?

```javascript
async function load() {
    try {
        fetchUser();
    } catch (err) {
        console.error("failed", err);
    }
}
```

---

### 4. A user's API returns `500` with an HTML error page. Trace what this code does, line by line, and name the error the user actually sees:

```javascript
const res = await fetch("/api/users");
const data = await res.json();
```

---

### 5. Explain:

A search box fires a request per keystroke. Requests for `"a"` and `"ab"` are in flight; `"a"` resolves last. Describe what the user sees, and give two different fixes — one that only guards state, and one that also saves the network request.

---

## Next Module — ES6+ Features & Modern JavaScript

We'll cover everything introduced in modern JavaScript that interviewers expect you to know, including:

* `let`/`const`
* Template literals
* Destructuring
* Spread vs Rest
* Default parameters
* Optional chaining (`?.`)
* Nullish coalescing (`??`)
* Logical assignment operators (`&&=`, `||=`, `??=`)
* Modules (`import`/`export`)
* Dynamic imports
* Iterators & Generators
* `Map`, `Set`, `WeakMap`, `WeakSet`
* Symbols
* BigInt
* Modern array methods (`toSorted`, `toReversed`, `findLast`, etc.)
* Practical interview questions and React use cases

This module will complete your **advanced JavaScript** foundation before we transition into **TypeScript**.

---

| ← Previous | Index | Next → |
| ---------- | ----- | ------ |
| [Module 5 — `this` & Prototypes](this-prototypes.md) | [Study Plan](../README.md) | [Module 7 — ES6+ & Modern JavaScript](es6-modern-js.md) |
