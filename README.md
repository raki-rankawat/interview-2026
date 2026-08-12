# React Interview Study Plan

Target role: **Frontend / Senior Frontend Engineer (Europe)**

This is the master index. Every note file links back here, and to the previous and next note, so you always know where you are and what comes next.

---

## Notes Written So Far

| # | Module | Topic | File |
| - | ------ | ----- | ---- |
| 1 | Phase 1 · Fundamentals | Variables (`var`, `let`, `const`), scope, hoisting, TDZ | [variables.md](javascript/variables.md) |
| 2 | Phase 1 · Fundamentals | Data Types & Memory (primitives, references, copying) | [data-types.md](javascript/data-types.md) |
| 3 | Phase 1 · Fundamentals | Functions, arrow functions, execution context, call stack, scope chain, closures, stale closures | [functions-closures.md](javascript/functions-closures.md) |
| 4 | Phase 1 · Fundamentals | Objects & arrays, destructuring, spread/rest, array methods, mutation vs immutability, complexity | [objects-arrays.md](javascript/objects-arrays.md) |
| 5 | Phase 1 · Fundamentals | `this` binding rules, `call`/`apply`/`bind`, prototypes, prototype chain, classes, inheritance | [this-prototypes.md](javascript/this-prototypes.md) |
| 6 | Phase 1 · Fundamentals | Event loop, micro/macrotasks, promises, `async`/`await`, fetch, `AbortController`, retries, debounce/throttle | [async-event-loop.md](javascript/async-event-loop.md) |
| 7 | Phase 1 · ES6+ | Template literals, modules, dynamic imports, `Map`/`Set`/`WeakMap`, iterators, generators, symbols, modern array methods | [es6-modern-js.md](javascript/es6-modern-js.md) |
| 8 | Phase 3 · React Fundamentals | What React is, imperative vs declarative, JSX and its rules, function components, props, state, props vs state, composition, conditional rendering, lists & keys | [react-fundamentals.md](react/react-fundamentals.md) |

**Up next:** Module 9 — React Fundamentals Part 2: controlled vs uncontrolled components, rendering basics, Virtual DOM, reconciliation, common mistakes.

Phase 2 (TypeScript) has been skipped for now and has no notes yet.

Unlinked topics below have no notes yet. They get linked here as each one is written.

---

# Phase 1 – JavaScript (Master First)

## Fundamentals

* [Variables (`var`, `let`, `const`)](javascript/variables.md) ✅
* [Primitive vs Reference types](javascript/data-types.md) ✅
* [Scope](javascript/variables.md) — covered inside Variables
* [Hoisting](javascript/variables.md) — covered inside Variables
* [Closures](javascript/functions-closures.md) ✅
* [Lexical scope](javascript/functions-closures.md) — covered inside Functions & Closures
* [Execution Context](javascript/functions-closures.md) ✅
* [Call Stack](javascript/functions-closures.md) — covered inside Functions & Closures
* [`this` and the binding rules](javascript/this-prototypes.md) ✅
* [Prototypes & the prototype chain](javascript/this-prototypes.md) ✅
* [Classes & inheritance](javascript/this-prototypes.md) — covered inside `this` & Prototypes
* [Event Loop](javascript/async-event-loop.md) ✅
* [Microtasks vs Macrotasks](javascript/async-event-loop.md) ✅
* [Garbage Collection](javascript/data-types.md) — partially covered in Data Types & Memory

## ES6+

* [Arrow functions](javascript/functions-closures.md) — covered inside Functions & Closures
* [Destructuring](javascript/es6-modern-js.md) ✅
* [Spread operator](javascript/es6-modern-js.md) ✅
* [Rest operator](javascript/es6-modern-js.md) ✅
* [Template literals](javascript/es6-modern-js.md) ✅
* [Optional chaining](javascript/es6-modern-js.md) ✅
* [Nullish coalescing](javascript/es6-modern-js.md) ✅
* [Logical assignment operators](javascript/es6-modern-js.md) ✅
* [Modules](javascript/es6-modern-js.md) ✅
* [Dynamic imports](javascript/es6-modern-js.md) ✅
* [`Map`, `Set`, `WeakMap`, `WeakSet`](javascript/es6-modern-js.md) ✅
* [Iterators & Generators](javascript/es6-modern-js.md) ✅

## Objects & Arrays

* [Object methods](javascript/objects-arrays.md) ✅
* [Array methods](javascript/objects-arrays.md) ✅
  * [map](javascript/objects-arrays.md) ✅
  * [filter](javascript/objects-arrays.md) ✅
  * [reduce](javascript/objects-arrays.md) ✅
  * [some](javascript/objects-arrays.md) ✅
  * [every](javascript/objects-arrays.md) ✅
  * [find](javascript/objects-arrays.md) ✅
  * [flatMap](javascript/objects-arrays.md) ✅
* [Deep copy vs Shallow copy](javascript/data-types.md) ✅ — also recapped in Objects & Arrays
* [Immutability](javascript/data-types.md) ✅ — mutation vs non-mutation covered in Objects & Arrays
* [`Object.freeze()`](javascript/objects-arrays.md) ✅

## Async JavaScript

* [Promises](javascript/async-event-loop.md) ✅
* [async/await](javascript/async-event-loop.md) ✅
* [`Promise.all`](javascript/async-event-loop.md) ✅
* [`Promise.allSettled`](javascript/async-event-loop.md) ✅
* [`Promise.race`](javascript/async-event-loop.md) ✅
* [`Promise.any`](javascript/async-event-loop.md) ✅
* [Fetch API](javascript/async-event-loop.md) ✅
* [AbortController](javascript/async-event-loop.md) ✅
* [Error handling](javascript/async-event-loop.md) ✅
* [Retry strategies](javascript/async-event-loop.md) ✅
* [Exponential backoff](javascript/async-event-loop.md) ✅

---

# Phase 2 – TypeScript

## Basics

* Types
* Interfaces
* Type aliases
* Enums
* Literal types
* Union types
* Intersection types

## Advanced

* Generics
* Utility Types
  * Partial
  * Pick
  * Omit
  * Record
  * Required
  * Readonly
* `keyof`
* `typeof`
* `infer`
* Conditional Types
* Mapped Types
* Type narrowing
* Type Guards

---

# Phase 3 – React Fundamentals

## JSX

* [JSX](react/react-fundamentals.md) ✅
* [Rendering](react/react-fundamentals.md) — partially covered in React Fundamentals (JSX → elements → UI; the render pipeline comes next)
* [Conditional rendering](react/react-fundamentals.md) ✅
* [Lists](react/react-fundamentals.md) ✅
* [Keys](react/react-fundamentals.md) ✅
* [Fragments](react/react-fundamentals.md) — covered inside React Fundamentals

## Components

* [Functional Components](react/react-fundamentals.md) ✅
* [Props](react/react-fundamentals.md) ✅
* [State](react/react-fundamentals.md) ✅
* Controlled Components
* Uncontrolled Components
* [Composition](react/react-fundamentals.md) ✅

---

# Phase 4 – React Hooks

Know:

* `useState`
* `useEffect`
* `useRef`
* `useMemo`
* `useCallback`
* `useContext`
* `useReducer`
* custom hooks

Know:

* dependency array
* cleanup
* [stale closure](javascript/functions-closures.md) — covered inside Functions & Closures
* infinite loops
* React Strict Mode
* React 18 rendering

---

# Phase 5 – React Performance

Must know deeply:

* `React.memo`
* `useMemo`
* `useCallback`
* Lazy loading
* `React.lazy`
* Suspense
* Code splitting
* Virtualization
* Memoization
* [Debouncing](javascript/async-event-loop.md) — covered inside Async JavaScript
* [Throttling](javascript/async-event-loop.md) — covered inside Async JavaScript
* Bundle optimization

Know when **NOT** to optimize.

---

# Phase 6 – State Management

Understand the differences:

* Local State
* Context API
* Redux Toolkit
* Zustand
* MobX (basic idea)
* React Query / TanStack Query

Interview questions:

* Context vs Redux
* Redux vs Zustand
* Client State vs Server State

---

# Phase 7 – React Architecture

* Folder structure
* Feature-based architecture
* Atomic Design
* Separation of concerns
* Custom hooks
* Reusable components
* Dependency Injection concepts
* Container vs Presentational Components

---

# Phase 8 – API Integration

Know everything:

* REST
* HTTP methods
* Status codes
* Authentication
* JWT
* Refresh Tokens
* Cookies vs LocalStorage
* Session Storage

API handling:

* Loading
* Error
* Retry
* Cancellation
* AbortController
* Pagination
* Infinite scrolling
* Optimistic updates
* Caching
* Rate limiting

---

# Phase 9 – React Forms

* Controlled forms
* Validation
* React Hook Form
* Formik (basic)
* Zod
* Yup

---

# Phase 10 – Routing

React Router:

* Nested routes
* Dynamic routes
* Protected routes
* Lazy routes
* Navigation
* URL params
* Search params

---

# Phase 11 – CSS

## Layout

* Flexbox
* Grid

## Responsive Design

* Media queries
* Mobile-first design
* Breakpoints
* Container queries
* Responsive typography
* `clamp()`
* `aspect-ratio`
* `object-fit`
* Responsive images

## Modern CSS

* CSS variables
* BEM
* CSS Modules
* Tailwind CSS
* SCSS basics

---

# Phase 12 – Accessibility (Very Common)

* Semantic HTML
* ARIA
* Keyboard navigation
* Screen readers
* Focus management
* Color contrast
* Forms accessibility

---

# Phase 13 – Testing

* Jest
* React Testing Library
* Unit Testing
* Integration Testing
* Mocking APIs
* Snapshot testing
* User Event testing

---

# Phase 14 – Build Tools

* Vite
* Webpack (concepts)
* Babel
* npm
* pnpm
* yarn
* Tree shaking
* Source maps

---

# Phase 15 – Browser Knowledge

* DOM
* Virtual DOM
* Reconciliation
* Rendering pipeline
* Reflow
* Repaint
* Event bubbling
* Event capturing
* Event delegation
* Browser storage
* Cookies

---

# Phase 16 – Security

* XSS
* CSRF
* CORS
* HTTPS
* Content Security Policy
* Authentication best practices
* Token storage

---

# Phase 17 – Next.js (Very Common)

* App Router
* Pages Router
* SSR
* CSR
* SSG
* ISR
* Server Components
* Client Components
* API Routes
* Middleware

---

# Phase 18 – Git

* Rebase
* Merge
* Cherry Pick
* Squash
* Stash
* Reset
* Revert
* Conflict resolution

---

# Phase 19 – System Design (Frontend)

Prepare to discuss:

* Designing a dashboard
* Designing a chat application
* Designing an e-commerce frontend
* Large component architecture
* Folder organization
* State management strategy
* Performance optimization
* Code splitting
* Caching
* Offline support

---

# Phase 20 – Behavioral Questions

Prepare structured answers (STAR method) for:

* Tell me about yourself.
* Biggest challenge you solved.
* Most difficult bug.
* Project you're proud of.
* Conflict with teammate.
* Why are you leaving your previous job?
* Why this company?
* Biggest mistake you made.
* Leadership experience.
* Working under pressure.
* Handling deadlines.
* Receiving feedback.
* Mentoring juniors.

---

# Phase 21 – Coding Exercises

Practice regularly:

* Build a Todo App
* Infinite Scroll
* Debounced Search
* Kanban Board
* Notes App
* Book Tracker
* Weather App
* Chat UI
* Pagination
* File Explorer
* Tree View
* Modal Manager
* Data Table
* Drag & Drop
* Theme Switcher

---

# Suggested 8-Week Study Plan

| Week | Focus |
| ---- | ----- |
| 1 | JavaScript + TypeScript fundamentals |
| 2 | React fundamentals + Hooks |
| 3 | Performance optimization + State management |
| 4 | APIs, Routing, Forms, Authentication |
| 5 | CSS, Responsive Design, Accessibility |
| 6 | Next.js, Testing, Build tools, Security |
| 7 | Frontend System Design + Behavioral questions |
| 8 | Mock interviews, coding exercises, and revision |

## Daily Routine (2–3 hours)

* **45 min:** Study one core topic (e.g., `useEffect`, Generics, React Query).
* **45 min:** Build or extend a small feature applying what you learned.
* **30 min:** Solve JavaScript or React interview questions.
* **30 min:** Explain a concept out loud as if you're teaching an interviewer, focusing on trade-offs and real-world scenarios.

---

## Two Additions Worth Prioritizing

For **Frontend / Senior Frontend Engineer** roles in Europe, these are now expected as often as Redux and Hooks:

* **React Query (TanStack Query):** caching, query invalidation, optimistic updates, infinite queries, mutations, stale vs fresh data.
* **Frontend System Design:** component architecture, state boundaries, rendering strategies, performance, and scalability.

---

## Repo Structure

```
interview-2026/
├── javascript/
│   ├── variables.md            Module 1
│   ├── data-types.md           Module 2
│   ├── functions-closures.md   Module 3
│   ├── objects-arrays.md       Module 4
│   ├── this-prototypes.md      Module 5
│   ├── async-event-loop.md     Module 6
│   ├── es6-modern-js.md        Module 7
│   ├── variables.js
│   └── index.html
└── react/
    └── react-fundamentals.md   Module 8
```
