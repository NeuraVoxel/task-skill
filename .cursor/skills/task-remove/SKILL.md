---
name: task-remove
description: >-
  Deletes a TODO.md ledger item by id (T-NNN or B-NNN) from the primary
  checkout. Optionally removes the task worktree (--dwt) and/or deletes the
  task branch (--dbr). Use when the user runs /task-remove, asks to delete or
  drop a TODO task/bug line, or remove an added item from the ledger.
disable-model-invocation: true
---

# task-remove

Remove one ledger entry from **primary** `TODO.md`. Does not implement work, merge, or push.

**Primary only:** If cwd looks like a task worktree (`{project}-T-*` / `{project}-B-*`, `{project}` = primary basename), stop and tell the user to run `/task-remove` on primary.

## Inputs

| Field | Required | Notes |
|-------|----------|-------|
| id | yes | `T-001`, `B-002`, … from `/task-remove <ID> …` |
| `--dwt` | no | Remove the task worktree if present (skip ask when claimed / discoverable) |
| `--dbr` | no | Delete the local task/bug branch if present (skip ask) |

If id missing: ask once, then stop.

## Steps

1. On primary, read `TODO.md`. Find the checkbox line matching `{ID}` under `## Task` or `## Bug`.
2. If not found: report and stop.
3. Parse optional tails for cleanup hints:
   - `@claimed {branch} {path}` or `@claimed main`
   - `— done {branch}` / `— merged {branch}`
   - Fallback branch: `T-*` → `task/{ID}`, `B-*` → `bug/{ID}`
4. **Worktree cleanup** (before deleting the line):
   - Discover path: claimed `{path}`, else `git worktree list` entry on `{branch}`, else sibling `../{project}-{ID}` if listed.
   - If a worktree exists:
     - `--dwt` present → `git worktree remove <path>` (`--force` only if user explicitly allows discarding worktree changes).
     - `--dwt` absent → **ask** whether to remove it; wait; remove only on yes.
   - If remove fails: **stop** without deleting the TODO line; report the git error.
5. **Branch cleanup** (optional):
   - If `--dbr` present and `{branch}` exists locally → `git branch -d {branch}` (`-D` only if user explicitly allows force).
   - If `--dbr` absent and a task branch exists (claimed / done / merged / fallback name) → **ask** whether to delete it; wait; delete only on yes.
   - If no branch: skip.
6. **Delete the ledger line** on primary `TODO.md` (the whole `- [ ]` / `- [x]` line for `{ID}`). Do not renumber other IDs. Leave section headers (`## Task` / `## Bug`) in place even if empty.
7. Do not merge, push, or open PRs.
8. Reply briefly: ID removed, title, worktree removed/kept/skipped, branch deleted/kept/skipped.

## Rules

- Only edit `TODO.md` on primary.
- Never auto-merge to `main`.
- Removing a `merged` or historical `done` line only drops the ledger row; it does not revert commits already on `main`.
- Prefer `/task-unclaim` when the user only wants to free a claim but keep the open TODO line.
- `--dwt` / `--dbr` only skip **cleanup** questions — they never force through worktree remove failures.
