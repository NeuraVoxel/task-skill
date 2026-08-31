---
name: task-mode
description: >-
  Sets or shows the local TODO workflow mode (parallel|serial) in repo-root
  .todo-mode. Use when the user runs /task-mode, asks to switch parallel vs
  serial task execution, or change worktree behavior for /task-do.
disable-model-invocation: true
---

# task-mode

Persist execution mode for `/task-do` in the **primary** checkout only.

**Spec:** `docs/superpowers/specs/2026-08-31-parallel-todo-worktree-design.md`

## Inputs

- Optional: `parallel` or `serial` from `/task-mode <mode>` or the message.
- If missing: print current mode and stop (do not change).

## Steps

1. Resolve primary repo root (directory that contains `TODO.md` and `.git` or is the main worktree). Prefer the vault path the user considers primary, not a `{project}-T-*` / `{project}-B-*` task worktree (`{project}` = primary basename).
2. Read `.todo-mode` if present; otherwise treat current as `parallel`.
3. If user only asked to show mode: reply with current value and stop.
4. If setting `serial`: scan primary `TODO.md` for any line containing `@claimed` and a worktree path (not `main`). If found, **warn** to finish or `/task-unclaim` those IDs first; still allow the write if user explicitly confirms in the same turn, otherwise stop after the warning.
5. Write `.todo-mode` as a single line: `parallel` or `serial` (no quotes, trailing newline OK).
6. Reply: previous mode → new mode. Remind: only **new** `/task-do` runs use the new mode.

## Rules

- Never commit `.todo-mode` (gitignored).
- Do not create/remove worktrees here.
