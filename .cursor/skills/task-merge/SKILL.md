---
name: task-merge
description: >-
  Merges a done TODO.md item's task/bug branch into main on the primary
  checkout, pauses on conflicts, then optionally deletes the branch (--dbr)
  and/or worktree (-dwt). Use when the user runs /task-merge, asks to merge a
  finished T-NNN/B-NNN into main, or clean up after a done ledger item.
disable-model-invocation: true
---

# task-merge

Merge a **done** ledger item into `main` from the **primary** checkout. Optional cleanup of branch / worktree.

## Inputs

| Field | Required | Notes |
|-------|----------|-------|
| id | yes | `T-001`, `B-002`, … from `/task-merge <ID> …` |
| `--dbr` | no | Delete the task/bug branch after a clean merge (skip ask) |
| `-dwt` | no | Delete the task worktree after a clean merge (skip ask) |

If id missing: ask once, then stop.

## Preconditions

1. **cwd / edits:** Run on **primary** only. If cwd looks like a task worktree (`{project}-T-*` / `{project}-B-*`, `{project}` = primary basename), stop and tell the user to run `/task-merge` on primary.
2. On primary, read `TODO.md`. Find the checkbox line matching `{ID}`.
3. If not found: report and stop.
4. **Must be done, not yet merged:**
   - Accept: `- [x] {ID} {title} — done {branch}`
   - Reject (report and stop): open / `@claimed` / `blocked` / already `— merged …` / `[x]` without `— done {branch}`
5. Resolve `{branch}` from the `— done` tail (prefer the written branch; fallback `T-*` → `task/{ID}`, `B-*` → `bug/{ID}`).
6. Primary working tree on `main` must be clean enough to merge (no conflicting dirty state). If dirty in a way that blocks merge: stop and ask the user to clean up. Do **not** stash.

## Steps

### 1. Merge

1. `git switch main`
2. `git merge {branch}` (default merge; allow fast-forward or merge commit).
3. **On conflict:**
   - Stop immediately.
   - List conflicted paths.
   - Leave the in-progress merge as git left it.
   - Do **not** update `TODO.md`, do **not** delete branch/worktree, do **not** push.
   - Tell the user to resolve, `git add`, `git commit` (or `git merge --abort`), then re-run `/task-merge {ID}` if still `— done` (or finish manually and set `— merged` themselves).
4. **On success:** continue.

### 2. Ledger (primary only)

Rewrite the line to:

```markdown
- [x] {ID} {title} — merged {branch}
```

Keep the original title; strip `done` / other tails. Do not edit a worktree copy of `TODO.md`.

### 3. Cleanup prompts

Discover worktree (if any):

1. Prefer a path from `git worktree list` whose branch is `{branch}`.
2. Else, if `../{project}-{ID}` (absolute sibling of primary) exists in `git worktree list`, use that.
3. Else: no worktree.

**Branch delete (`--dbr`):**

- If `--dbr` present: `git branch -d {branch}` (use `-D` only if user explicitly allows force).
- If `--dbr` absent: **ask** whether to delete `{branch}`; wait for answer; delete only on yes.

**Worktree delete (`-dwt`):**

- If no worktree: report skipped.
- If `-dwt` present and worktree exists: `git worktree remove <path>` (`--force` only if user explicitly allows discarding worktree changes).
- If `-dwt` absent and worktree exists: **ask** whether to delete it; wait for answer; remove only on yes.

If both flags present: delete both without asking (still only after a clean merge + ledger update).

If only one flag present: auto-do that one; still **ask** about the other when applicable.

### 4. Reply

Briefly: ID, branch, merge result (merged / conflict-paused), ledger (`merged`), branch kept or deleted, worktree removed or skipped/kept.

## Rules

- Never push, open a PR, or merge anything that is not this `{ID}`'s done branch.
- Never auto-merge without this skill being invoked.
- Do not uncheck `[x]` or reopen the item.
- Do not delete branch/worktree on conflict.
- `--dbr` / `-dwt` only skip **cleanup** questions after a **successful** merge — they never force through conflicts.
