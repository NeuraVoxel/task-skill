---
name: task-merge
description: >-
  Merges a done TODO.md item's task/bug branch into main on the primary
  checkout; auto-reconciles dirty primary TODO.md (claims); optional --stash
  for other dirty files; pauses on non-TODO conflicts; optionally deletes the
  branch (--dbr) and/or worktree (-dwt). Use when the user runs /task-merge,
  asks to merge a finished T-NNN/B-NNN into main, or clean up after a done
  ledger item.
disable-model-invocation: true
---

# task-merge

Merge a **done** ledger item into `main` from the **primary** checkout. Optional cleanup of branch / worktree.

Done status may live on the **task branch / worktree** `TODO.md` (written by `/task-do` close-out) while primary still shows `@claimed`. Prefer branch/worktree for the done check; after merge, rewrite primary to `— merged`.

Parallel claim lines often sit **uncommitted** on primary — that is normal. This skill **auto-reconciles** dirty primary `TODO.md` (does not discard claims). Non-`TODO.md` dirt is **not** stashed unless `--stash`.

## Inputs

| Field | Required | Notes |
|-------|----------|-------|
| id | yes | `T-001`, `B-002`, … from `/task-merge <ID> …` |
| `--dbr` | no | Delete the task/bug branch after a clean merge (skip ask) |
| `-dwt` | no | Delete the task worktree after a clean merge (skip ask) |
| `--stash` | no | If primary has non-`TODO.md` dirty files: stash them before merge, `stash pop` after ledger finalize. **Default off.** |

If id missing: ask once, then stop.

## Preconditions

1. **cwd / edits:** Run on **primary** only. If cwd looks like a task worktree (under `{primary}/.worktrees/`, or basename `{project}-T-*` / `{project}-B-*`, or a claimed task path — see `task-do/reference.md`), stop and tell the user to run `/task-merge` on primary.
2. On primary, read `TODO.md`. Find the checkbox line matching `{ID}`.
3. If not found: report and stop.
4. **Resolve done status** (must be done, not yet merged):
   - Reject immediately if primary (or branch) already shows `— merged {branch}`.
   - **Accept** if any of these show `- [x] {ID} … — done {branch}`:
     1. Primary `TODO.md` line, or
     2. Claimed worktree path's `TODO.md` (from primary `@claimed {branch} {path}`), or
     3. `git show {branch}:TODO.md` for the resolved `{branch}`
   - If primary still has `@claimed` / open but worktree or `{branch}` has `— done`, that is **valid** (normal after `/task-do` close-out).
   - Reject (report and stop): no done line anywhere; only `@claimed` / open / `blocked` with no done on branch/worktree; `[x]` without `— done {branch}`.
5. Resolve `{branch}` from the `— done` tail when present; else from `@claimed {branch} …`; else fallback `T-*` → `task/{ID}`, `B-*` → `bug/{ID}`.
6. **Primary dirty-tree gate** (see §0) — must pass before merge. Do **not** stash unless `--stash`. Do **not** discard uncommitted `TODO.md` claim/ledger lines.

## Steps

### 0. Dirty-tree prep (primary, before merge)

Inspect `git status` on primary `main`.

| Dirty set | Action |
|-----------|--------|
| Clean | Proceed to §1. |
| **Only** `TODO.md` | **Auto-reconcile** (below). Do not ask the user to clean up. Do **not** pre-commit as a required step (prefer reconcile). |
| Any non-`TODO.md` path, **without** `--stash` | **Stop.** List dirty paths. Tell the user to clean up those files, or re-run with `--stash`. Do **not** stash. Do **not** discard `TODO.md`. |
| Any non-`TODO.md` path, **with** `--stash` | Stash **only** those non-`TODO.md` paths (`git stash push -m "task-merge {ID}" -- <paths>`). Remember `stashed=true` for §3b. Then handle `TODO.md` per reconcile if it is also dirty. |

**TODO.md reconcile (preferred over auto-commit):**

1. If working-tree `TODO.md` differs from `HEAD`: copy it to a backup under the repo git dir, e.g. `{primary}/.git/task-merge-primary-TODO.md` (overwrite OK for this run).
2. `git restore --source=HEAD -- TODO.md` so the index/worktree match `HEAD` for merge (backup holds claims / uncommitted ledger rows).
3. Set `todo_backup=<that path>` for §2. If `TODO.md` was clean, `todo_backup` is empty — §2 only finalizes `{ID}`.

Do **not** use `git stash` for `TODO.md`. Do **not** delete the backup until §2 finishes successfully (then remove it).

### 1. Merge

1. `git switch main`
2. `git merge {branch}` (default merge; allow fast-forward or merge commit).
3. **On conflict:**
   - If **only** `TODO.md` conflicts (or `TODO.md` plus easily resolvable noise):
     - Resolve with **ledger merge rules** (§2a): `{ID}` → prefer branch/`theirs` `- [x] … — done {branch}` over primary `@claimed`; **preserve** other IDs' lines from `HEAD`/ours and from `todo_backup` (especially `@claimed` and rows not on the task branch).
     - `git add TODO.md`, continue the merge (`git commit` if merge stopped for conflicts). Then proceed to §2.
   - Otherwise:
     - Stop immediately.
     - List conflicted paths.
     - Leave the in-progress merge as git left it.
     - Keep `todo_backup` for the user; if `stashed=true`, leave the stash in place and say how to `stash pop` after they abort or finish.
     - Do **not** finalize ledger as `merged`, do **not** delete branch/worktree, do **not** push.
     - Tell the user to resolve, `git add`, `git commit` (or `git merge --abort`), then re-run `/task-merge {ID}` if still `— done` (or finish manually and set `— merged` themselves).
4. **On success:** continue to §2.

### 2. Ledger (primary after merge)

#### 2a. Ledger merge rules

When combining post-merge `TODO.md`, `todo_backup` (if any), and conflict sides:

1. **`{ID}` (the merge target):** end state after this section is always  
   `- [x] {ID} {title} — merged {branch}`  
   Keep the original title; strip `done` / `@claimed` / other tails.
2. **Every other ID** present in `todo_backup` but missing or weaker on post-merge `TODO.md`: **re-apply the backup line** (typical: other tasks' `@claimed …`, or open rows added on primary but not yet on the task branch). Prefer backup `@claimed` / open over dropping the row. Do not overwrite another ID that already shows `— merged` or `— done` on post-merge with an older backup open line.
3. Keep section structure (`## Task` / `## Bug`); insert restored rows under the correct section by ID prefix (`T-*` → Task, `B-*` → Bug).

If there was no `todo_backup`, only rewrite `{ID}` to `— merged {branch}`.

#### 2b. Persist ledger

After rewriting:

1. If `TODO.md` differs from `HEAD`, commit it on `main` (message like `chore(todo): mark {ID} as merged`). Include preserved claim lines for other IDs in that commit so parallel ledger state is not left only in the worktree.
2. Delete `todo_backup` if it exists.
3. Do **not** uncheck `[x]` or reopen `{ID}`.

### 3. Cleanup prompts

Discover worktree (if any):

1. Prefer a path from `git worktree list` whose branch is `{branch}`.
2. Else claimed `{path}` from primary `@claimed …` if listed (use pre-merge claim path from backup/memory if the merged ledger already dropped it).
3. Else `.worktrees/{ID}` if listed.
4. Else legacy sibling `../{project}-{ID}` if listed.
5. Else: no worktree.

**Branch delete (`--dbr`):**

- If `--dbr` present: `git branch -d {branch}` (use `-D` only if user explicitly allows force).
- If `--dbr` absent: **ask** whether to delete `{branch}`; wait for answer; delete only on yes.

**Worktree delete (`-dwt`):**

- If no worktree: report skipped.
- If `-dwt` present and worktree exists: `git worktree remove <path>` (`--force` only if user explicitly allows discarding worktree changes).
- If `-dwt` absent and worktree exists: **ask** whether to delete it; wait for answer; remove only on yes.

If both flags present: delete both without asking (still only after a clean merge + ledger update).

If only one flag present: auto-do that one; still **ask** about the other when applicable.

### 3b. Stash restore

If `stashed=true` from §0:

1. `git stash pop` (the `task-merge {ID}` stash).
2. On pop conflict: stop; report; leave conflicts for the user. Do **not** drop the stash if pop failed mid-way without applying.
3. On success: continue to §4.

If merge was conflict-paused in §1 without finishing: do **not** pop here (already instructed in §1).

### 4. Reply

Briefly: ID, branch, merge result (merged / conflict-paused), ledger (`merged` + whether other claims were restored), dirty-tree handling (TODO reconciled / `--stash` used / clean), branch kept or deleted, worktree removed or skipped/kept, stash pop result if any.

## Rules

- Never push, open a PR, or merge anything that is not this `{ID}`'s done branch.
- Never auto-merge without this skill being invoked.
- Do not uncheck `[x]` or reopen the item.
- Do not delete branch/worktree on unresolved non-TODO conflicts.
- **Dirty `TODO.md`:** auto-reconcile via backup; never discard claim/ledger lines; do not require the user to delete uncommitted TODO content.
- **Dirty non-`TODO.md`:** default stop; stash **only** with explicit `--stash`.
- Never `git stash` for `TODO.md` (backup file + restore + re-apply only).
- `--dbr` / `-dwt` only skip **cleanup** questions after a **successful** merge — they never force through non-TODO conflicts or skip the dirty-tree gate.
- `--stash` never forces through non-TODO merge conflicts.
