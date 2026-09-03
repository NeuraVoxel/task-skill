---
name: task-unclaim
description: >-
  Abandons an in-progress TODO claim: restores the open checkbox line, removes
  the task worktree if present, leaves the task branch unless the user asks to
  delete it. Use when the user runs /task-unclaim, unclaims a stuck @claimed ID,
  or frees a claimed task without finishing it.
disable-model-invocation: true
---

# task-unclaim

Unclaim a claimed but unfinished ledger item from the **primary** checkout.

**Spec:** `docs/superpowers/specs/2026-08-31-parallel-todo-worktree-design.md`

**Primary only:** If cwd looks like a task worktree (under `{primary}/.worktrees/`, or basename `{project}-T-*` / `{project}-B-*`, or a claimed task path — see `task-do/reference.md`), stop and tell the user to run `/task-unclaim` on primary.

## Inputs

| Field | Required | Notes |
|-------|----------|-------|
| id | yes | `T-001`, `B-002`, … |
| `--dwt` | no | Remove the task worktree if present (skip ask when discoverable) |
| `--dbr` | no | Delete the local task/bug branch (skip ask) |

If id missing: ask once, then stop.

## Steps

1. On primary, read `TODO.md`. Find the line with `{ID}`.
2. If not found: report and stop.
3. If line is already `- [x]`: report done-item; do not uncheck unless user explicitly asks to reopen.
4. Parse optional `@claimed {branch} {path}` or `@claimed main`. Fallback branch: `T-*` → `task/{ID}`, `B-*` → `bug/{ID}`.
5. Rewrite the line to `- [ ] {ID} {title}` (strip claim / blocked tails; keep original title only).
6. **Worktree cleanup:**
   - Discover path (first match): claimed `{path}` if listed → `git worktree list` on `{branch}` → `.worktrees/{ID}` if listed → legacy `../{project}-{ID}` if listed.
   - If a worktree exists: `--dwt` → remove; else **ask**; remove only on yes (`--force` only if user allows discarding changes).
   - On remove failure: leave TODO unclaimed, report the git error.
7. **Branch cleanup:** `--dbr` → `git branch -d {branch}` (`-D` only if user allows force); else keep unless user asks to delete.
8. Do not merge, push, or open PRs.
9. Reply: ID unclaimed, worktree removed or skipped, branch kept or deleted.

## Rules

- Only edit `TODO.md` on primary.
- Never auto-merge to `main`.
- `--dwt` / `--dbr` only skip cleanup questions — they never force through worktree remove failures.
