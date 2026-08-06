# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repository is

A personal study repository for React / Senior Frontend Engineer interview preparation. The deliverable is **written study notes**, not an application. Most work here is authoring or revising markdown notes, not shipping code.

`README.md` is the master study plan (21 phases, JavaScript through behavioral) and the index of which notes exist so far.

## No build tooling

There is no `package.json`, no test runner, no linter, and no build step. Do not invent commands or suggest running `npm test` / `npm run build` — nothing is wired up. Code files exist only as scratch space for trying out the concepts being studied.

The repo is not a git repository yet (`git init` has not been run). The `commit-msg` skill will correctly refuse until it is.

## The linking invariant

This is the part that requires reading several files to see, and the part most easily broken.

Notes form **one global linear sequence** across the whole study plan. Each note declares its position in its own H1:

```markdown
# Module 2 — Data Types & Memory in JavaScript
```

Everything else is derived from those H1 numbers:

- Each note opens with `[← Study Plan](../../README.md)` (one `..` per directory level below root).
- Each note closes with a `| ← Previous | Index | Next → |` table linking its neighbours.
- `README.md` carries a "Notes Written So Far" table, an "Up next" pointer, and inline links from plan bullets to the notes that cover them.

**Adding a note therefore means editing its neighbour too** — the previous module's "Next" cell points at the new file. Missing this is how the nav silently goes stale.

Two rules that keep the index trustworthy:

- **Never link to a file that does not exist.** Unwritten topics in the README stay plain text. A dead link is worse than no link.
- **Coverage markers must be honest.** ` ✅` only for a note's primary topic; use ` — covered inside <Note>` or ` — partially covered in <Note>` otherwise. Overstating coverage tells the user a topic is done when it is not, which costs them in a real interview.

Run `/update-readme` rather than hand-editing the index; it rebuilds all of the above from the H1s and verifies no link is broken.

## Skills

- **`/update-readme`** — re-syncs `README.md` and every note's nav after notes are added, renamed, or reordered. Note files are the source of truth; the README is derived. To reorder notes, edit the H1 numbers and run this, never the reverse.
- **`/commit-msg`** — writes a conventional-commit message from the staged diff and commits. **Never adds a `Co-Authored-By` trailer** — this is an explicit user instruction that overrides the default.

## Note style

Match the existing notes (`javascript/fundamentals/`) rather than a generic docs voice:

- Teach a concept, then immediately show a runnable snippet, then show its output in a bare fenced block labelled `Output:`.
- `---` horizontal rules between major sections.
- Interview framing throughout: "Interview Question" / "Important Interview Point" callouts, a summary of what a strong candidate should be able to say, and unanswered practice questions at the end. **Leave practice questions unanswered** — they are for self-testing.
- Tie concepts back to React wherever the connection is real (why immutable state updates matter, why `let` fixes loop-handler closures).

## Technical accuracy

These notes are rehearsal for interviews, so a plausible-sounding error is actively harmful — it gets repeated to an interviewer. When the user supplies note content, verify claims before writing them down, and correct errors rather than transcribing them faithfully. Say what was corrected and why.

Precision that has already come up: `typeof NaN === "number"` is IEEE 754 behavior, **not** a bug, while `typeof null === "object"` genuinely is a historical bug. JavaScript is always pass-by-value — the counter-example that proves it is reassigning a parameter, not mutating one.
