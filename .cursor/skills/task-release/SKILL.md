---
name: task-release
description: >-
  Abandons an in-progress TODO claim: restores the open checkbox line, removes
  the task worktree if present, leaves the task branch unless the user asks to
  delete it. Use when the user runs /task-release, cancels a claimed task, or
  frees a stuck @claimed ID.
disable-model-invocation: true
---

# task-release

Release a claimed but unfinished ledger item from the **primary** checkout.

**Spec:** `docs/superpowers/specs/2026-08-31-parallel-todo-worktree-design.md`

## Inputs

- Required: id (`T-001`, `B-002`, …). If missing: ask once, then stop.

## Steps

1. On primary, read `TODO.md`. Find the line with `{ID}`.
2. If not found: report and stop.
3. If line is already `- [x]`: report done-item; do not uncheck unless user explicitly asks to reopen.
4. Parse optional `@claimed {branch} {path}` or `@claimed main`.
5. Rewrite the line to `- [ ] {ID} {title}` (strip claim / blocked tails; keep original title only).
6. If a worktree path was claimed and `git worktree list` shows it:
   - Run `git worktree remove <path>` (add `--force` only if user explicitly allows discarding worktree changes).
   - On failure: leave TODO released, report the git error.
7. Do **not** delete the local branch unless the user explicitly asks.
8. Do not merge, push, or open PRs.
9. Reply: ID released, worktree removed or skipped, branch kept or deleted.

## Rules

- Only edit `TODO.md` on primary.
- Never auto-merge to `main`.
