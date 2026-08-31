---
name: task-add
description: >-
  Registers a new task or bug in the repo-root TODO.md ledger with the next
  T-NNN or B-NNN id. Use when the user runs /task-add, asks to add a TODO item,
  log a task/bug, or append to TODO.md. Does not execute the task.
disable-model-invocation: true
---

# task-add

Append one entry to the project root `TODO.md`. Do not run `/task-do` or implement work.

**Primary only:** Run in the primary vault checkout. Do not append to `TODO.md` inside a task worktree (`{project}-T-*` / `{project}-B-*`, where `{project}` is the primary repo basename); if cwd looks like a task worktree, stop and tell the user to run `/task-add` on primary.

## Inputs

From the user message (ask only for what is missing):

| Field | Required | Default |
|-------|----------|---------|
| type | no | `task` (`task` \| `bug`) |
| title | yes | — |
| notes | no | empty (appended after `—` if present) |

## Steps

1. Resolve repo-root `TODO.md`. If missing, create it with this skeleton:

```markdown
# TODO

Project task ledger for `/task-add` and `/task-do`.
IDs: `T-NNN` (task) · `B-NNN` (bug). `[ ]` open · `[x]` done.
Claim: `— @claimed {branch} {worktree}` or `— @claimed main`.
Done: `— done {branch}` (branch kept; do not auto-merge `main`).
Merged: `— merged {branch}` via `/task-merge` (optional `--dbr` / `-dwt`).
Mode: `.todo-mode` → `parallel` (default) | `serial` via `/task-mode`.

## Task

## Bug
```

2. Prefix: `task` → `T`, `bug` → `B`. Scan the whole file for `\b{Prefix}-(\d+)\b` and take max + 1 (start at `001`, zero-pad to 3 digits).

3. Under `## Task` (for task) or `## Bug` (for bug), append one line:

```markdown
- [ ] {ID} {title}
```

If notes are provided:

```markdown
- [ ] {ID} {title} — {notes}
```

4. Reply with the new ID, type, and title. Stop. Do not start execution.

## Examples

- `/task-add fix typo in CLAUDE.md` → `- [ ] T-001 fix typo in CLAUDE.md`
- `/task-add bug: task-do crashes on missing ID` → `- [ ] B-001 task-do crashes on missing ID`
- `/task-add task "Add landing CTA" notes:"Hero needs primary button"` → `- [ ] T-002 Add landing CTA — Hero needs primary button`
