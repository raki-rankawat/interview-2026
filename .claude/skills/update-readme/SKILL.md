---
name: update-readme
description: Re-sync README.md and the note files' navigation links after notes are added, renamed, or reordered. Use when the user says "update the readme", "update readme", "re-sync the index", "fix the nav links", or runs /update-readme.
---

# update-readme

Keep `README.md` and every note file's navigation in agreement after notes are added, renamed, reordered, or deleted.

## Source of truth

**The note files are the source of truth. `README.md` is derived from them.**

Each note file declares its own identity in its H1:

```markdown
# Module 2 — Data Types & Memory in JavaScript
```

That module number sets the position in one **global linear sequence** that runs across the whole study plan. `README.md` and every prev/next link are rebuilt from that sequence. Never reorder notes by editing the README table — edit the H1s, then run this skill.

## Workflow

### 1. Inventory

Find every note file:

```powershell
Get-ChildItem -Path . -Recurse -Filter *.md -File |
  Where-Object { $_.FullName -notmatch '\\\.claude\\' -and $_.Name -ne 'README.md' } |
  Select-Object -ExpandProperty FullName
```

Read the H1 of each one. Build a table in your head of: module number, title, repo-relative path, containing directory.

### 2. Validate the sequence

Before writing anything, check:

- **Gaps** (1, 2, 4) — ask the user whether a note is missing or the numbers should be closed up.
- **Duplicates** (two Module 3s) — ask which one comes first. Do not guess.
- **No module number in the H1** — a newly added note. Assign it the next free number at the end of the sequence, and say so in your report. If the user's message indicates it belongs elsewhere ("this goes after closures"), insert it there and renumber the notes after it.

Stop and ask if anything is ambiguous. A wrong renumber silently rewrites every nav link in the repo.

### 3. Update each note file

For every note, in sequence order, set these three things and **nothing else**:

**a. Top back-link** — the first line of the file, followed by a blank line:

```markdown
[← Study Plan](<depth>/README.md)
```

`<depth>` is one `..` per directory level below the repo root. A file at `javascript/fundamentals/x.md` is two levels deep, so `../../README.md`. Compute this per file — do not copy it from a sibling at a different depth.

**b. H1** — `# Module <N> — <Title>`. Keep the existing title wording; only correct the number.

**c. Bottom nav table** — the last thing in the file, after a `---` rule:

```markdown
---

| ← Previous | Index | Next → |
| ---------- | ----- | ------ |
| [Module 1 — Variables](variables.md) | [Study Plan](../../README.md) | [Module 3 — Closures](closures.md) |
```

- Prev/next links are **relative to the current file**. Same directory is a bare filename; a different directory needs the relative hop (`../es6/arrow-functions.md`).
- First module: prev cell is `— (first module)`.
- Last module written so far: next cell names the planned next topic in italics with `*(not written yet)*`, taken from the study plan order. Do not link it.
- Link text is `Module <N> — <short title>`. Shorten a long H1 for the cell; the full title stays in the file.

**Do not touch the body of any note.** This skill maintains the top link, the H1 number, and the bottom table. Study content, headings, and code samples are the user's — leave them exactly as they are.

### 4. Update README.md

**a. "Notes Written So Far" table** — one row per note, in sequence order:

```markdown
| # | Module | Topic | File |
| - | ------ | ----- | ---- |
| 1 | Phase 1 · Fundamentals | Variables (`var`, `let`, `const`), scope, hoisting, TDZ | [variables.md](javascript/fundamentals/variables.md) |
```

The Topic cell is a short summary of what the note actually covers — read the note's section headings, don't just restate its title.

Directory-to-phase mapping for the Module column:

| Directory | Phase label |
| --------- | ----------- |
| `javascript/fundamentals/` | Phase 1 · Fundamentals |
| `javascript/es6/` | Phase 1 · ES6+ |
| `javascript/objects-arrays/` | Phase 1 · Objects & Arrays |
| `javascript/async/` | Phase 1 · Async JavaScript |
| `typescript/` | Phase 2 · TypeScript |
| `react/` | Phase 3–7 · React (pick the matching phase) |

For a directory not in this table, infer the phase from the study plan headings and add the row to this table so the mapping sticks.

**b. "Up next" line** under the table — the next unwritten topic in study-plan order.

**c. Inline links in the phase lists** — for every plan bullet the new note covers:

- The note's primary topic gets a link plus ` ✅`.
- A topic covered *inside* another note gets a link plus a short ` — covered inside <Note>` note, no checkmark.
- A topic only partly covered gets ` — partially covered in <Note>`. Be honest here; an overstated checkmark is worse than no link.
- Leave every unwritten topic as plain text. **Never create a link to a file that does not exist.**

**d. Repo structure block** at the bottom of the README — add the new file with its module number.

### 5. Verify

Check that every markdown link in every touched file resolves to a real path:

```powershell
Get-ChildItem -Recurse -Filter *.md -File |
  Where-Object { $_.FullName -notmatch '\\\.claude\\' } |
  ForEach-Object {
    $dir = $_.DirectoryName
    Select-String -Path $_.FullName -Pattern '\]\(([^)#][^)]*\.md)[^)]*\)' -AllMatches |
      ForEach-Object { $_.Matches } |
      ForEach-Object {
        $target = Join-Path $dir $_.Groups[1].Value
        if (-not (Test-Path $target)) { "BROKEN: $($_.Groups[1].Value)" }
      }
  }
```

Any output here is a bug you just introduced — fix it before reporting.

Then report: which files were added to the index, which nav links changed, and any renumbering that happened.

## Rules

- Never invent a link to a file that does not exist. An unwritten topic stays plain text.
- Never edit note content — only the top link, H1 number, and bottom nav table.
- Never delete a README section. If a note is gone, remove its row and downgrade its inline links back to plain text; leave the phase lists otherwise intact.
- If a note's H1 title changed, update every place that title appears: the README table, the inline links, and the prev/next cells in its two neighbours.
- Renumbering is destructive across many files. When in doubt, ask first.
