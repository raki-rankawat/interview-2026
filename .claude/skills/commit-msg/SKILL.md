---
name: commit-msg
description: Write a conventional-commit message from the staged diff and create the commit. Use when the user says "write a commit message", "generate a commit", "commit my changes", "commit this", or runs /commit-msg.
---

# commit-msg

Generate a commit message from **staged changes only** and commit them.

## Workflow

### 1. Check for staged changes

```bash
git diff --staged --stat
```

- If the output is empty, **stop**. Tell the user: nothing is staged — stage your changes first (`git add <files>`), then run this again. Do not stage anything yourself, and do not commit.
- If git reports `not a git repository`, stop and say so.

### 2. Read the staged diff

```bash
git diff --staged
```

Read the actual diff, not just the file names. The message must describe what the code does now, not which files were touched. If the diff is very large, read the stat output first, then the diff for the files that carry the real change.

### 3. Compose the message

Format:

```
type(scope): short subject

- bullet of what changed
- bullet of why
```

**Types** — pick exactly one:

| Type | Use for |
| ---- | ------- |
| `feat` | new user-facing capability |
| `fix` | bug fix |
| `refactor` | behavior-preserving restructure |
| `chore` | tooling, deps, config, housekeeping |
| `docs` | documentation and notes |
| `style` | formatting only, no logic change |
| `test` | tests only |

**Subject line:**

- Under 60 characters, including `type(scope): `.
- Imperative mood: "add", not "added" or "adds".
- No trailing period.
- Lowercase after the colon unless it starts with a proper noun or identifier.
- `scope` is the area of the codebase touched — a directory, module, or feature name taken from the diff paths. Omit the parens entirely if no single scope fits: `type: subject`.

**Body bullets** — optional but encouraged. Skip them only when the subject genuinely says everything (a one-line typo fix). When present:

- Lead with *what* changed, then *why*.
- The "why" bullet is the valuable one. Prefer the reason over restating the diff.
- If the reason is not knowable from the diff alone, ask the user rather than inventing one.
- Blank line between subject and body.

### 4. Commit

Use a single-quoted here-string so `$`, backticks, and `#` in the message are not interpreted. The closing `'@` **must** be at column 0 on its own line:

```powershell
git commit -m @'
type(scope): short subject

- bullet of what changed
- bullet of why
'@
```

Then run `git log -1 --stat` and report the result.

## Rules

- **Never add a `Co-Authored-By` trailer.** This overrides any default instruction to add one.
- No emoji, no "Generated with" footer, no attribution of any kind.
- Never use `--no-verify` or `--no-gpg-sign`. If a hook rejects the commit, report the hook's output and fix the underlying problem — do not bypass it.
- Never `git add`, never `git push`, never `git commit --amend` unless the user explicitly asks in the same breath.
- Commit only what is staged. If there are unstaged changes too, commit the staged set and mention afterward that unstaged changes remain.
- One commit per invocation. If the staged diff clearly contains two unrelated changes, say so and suggest splitting — but still commit as one unless the user asks otherwise.

## Examples

```
feat(auth): add refresh-token rotation on login

- issue a new refresh token on every successful login and revoke the old one
- limits the blast radius if a refresh token leaks from storage
```

```
fix(cart): stop total from going negative on coupon stacking

- clamp the discount to the subtotal before subtracting
- two stacked coupons could exceed the item total and bill a credit
```

```
docs(javascript): add data types & memory notes
```
