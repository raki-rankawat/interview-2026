# React Interview Study Plan

Target role: **Frontend / Senior Frontend Engineer (Europe)**

This is the master index. Every note file links back here, and to the previous and next note, so you always know where you are and what comes next.

---

## Notes Written So Far

| # | Module | Topic | File |
| - | ------ | ----- | ---- |
| 1 | Phase 1 · Fundamentals | Variables (`var`, `let`, `const`), scope, hoisting, TDZ | [variables.md](javascript/fundamentals/variables.md) |
| 2 | Phase 1 · Fundamentals | Data Types & Memory (primitives, references, copying) | [data-types.md](javascript/fundamentals/data-types.md) |
| 3 | Phase 1 · Fundamentals | Functions, arrow functions, execution context, call stack, scope chain, closures, stale closures | [functions-closures.md](javascript/fundamentals/functions-closures.md) |
| 4 | Phase 1 · Fundamentals | Objects & arrays, destructuring, spread/rest, array methods, mutation vs immutability, complexity | [objects-arrays.md](javascript/fundamentals/objects-arrays.md) |

**Up next:** Module 5 — `this`, Prototypes & the Prototype Chain.

Unlinked topics below have no notes yet. They get linked here as each one is written.

---

# Phase 1 – JavaScript (Master First)

## Fundamentals

* [Variables (`var`, `let`, `const`)](javascript/fundamentals/variables.md) ✅
* [Primitive vs Reference types](javascript/fundamentals/data-types.md) ✅
* [Scope](javascript/fundamentals/variables.md) — covered inside Variables
* [Hoisting](javascript/fundamentals/variables.md) — covered inside Variables
* [Closures](javascript/fundamentals/functions-closures.md) ✅
* [Lexical scope](javascript/fundamentals/functions-closures.md) — covered inside Functions & Closures
* [Execution Context](javascript/fundamentals/functions-closures.md) ✅
* [Call Stack](javascript/fundamentals/functions-closures.md) — covered inside Functions & Closures
* Event Loop
* Microtasks vs Macrotasks
* [Garbage Collection](javascript/fundamentals/data-types.md) — partially covered in Data Types & Memory

## ES6+

* [Arrow functions](javascript/fundamentals/functions-closures.md) — covered inside Functions & Closures
* [Destructuring](javascript/fundamentals/objects-arrays.md) — covered inside Objects & Arrays
* [Spread operator](javascript/fundamentals/objects-arrays.md) — covered inside Objects & Arrays
* [Rest operator](javascript/fundamentals/objects-arrays.md) — covered inside Objects & Arrays
* Template literals
* [Optional chaining](javascript/fundamentals/objects-arrays.md) — covered inside Objects & Arrays
* [Nullish coalescing](javascript/fundamentals/objects-arrays.md) — covered inside Objects & Arrays
* Modules
* Dynamic imports

## Objects & Arrays

* [Object methods](javascript/fundamentals/objects-arrays.md) ✅
* [Array methods](javascript/fundamentals/objects-arrays.md) ✅
  * [map](javascript/fundamentals/objects-arrays.md) ✅
  * [filter](javascript/fundamentals/objects-arrays.md) ✅
  * [reduce](javascript/fundamentals/objects-arrays.md) ✅
  * [some](javascript/fundamentals/objects-arrays.md) ✅
  * [every](javascript/fundamentals/objects-arrays.md) ✅
  * [find](javascript/fundamentals/objects-arrays.md) ✅
  * [flatMap](javascript/fundamentals/objects-arrays.md) ✅
* [Deep copy vs Shallow copy](javascript/fundamentals/data-types.md) ✅ — also recapped in Objects & Arrays
* [Immutability](javascript/fundamentals/data-types.md) ✅ — mutation vs non-mutation covered in Objects & Arrays
* [`Object.freeze()`](javascript/fundamentals/objects-arrays.md) ✅

## Async JavaScript

* Promises
* async/await
* `Promise.all`
* `Promise.allSettled`
* `Promise.race`
* Fetch API
* AbortController
* Error handling
* Retry strategies
* Exponential backoff

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

* Rendering
* Conditional rendering
* Lists
* Keys
* Fragments

## Components

* Functional Components
* Props
* State
* Controlled Components
* Uncontrolled Components
* Composition

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
* [stale closure](javascript/fundamentals/functions-closures.md) — covered inside Functions & Closures
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
* Debouncing
* Throttling
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
└── javascript/
    └── fundamentals/
        ├── variables.md            Module 1
        ├── data-types.md           Module 2
        ├── functions-closures.md   Module 3
        ├── objects-arrays.md       Module 4
        ├── variables.js
        └── index.html
```
