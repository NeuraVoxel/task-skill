# task-skill

Cursor Agent skills that turn repo-root `TODO.md` into a claim → work → done → merge workflow. **Claim** is written on primary; **done/blocked** is written on the task worktree/branch (not primary/`main`); **merged** is finalized on primary by `/task-merge`. Parallel work uses git worktrees.

## Skills

| Command | Role |
|---------|------|
| `/task-add` | Append a task (`T-NNN`) or bug (`B-NNN`) to `TODO.md` |
| `/task-remove` | Delete a ledger line by id; optional worktree/branch cleanup |
| `/task-mode` | Show or set `parallel` / `serial` (`.todo-mode`, gitignored) |
| `/task-do` | Claim an item, provision branch/worktree, implement, mark `done` |
| `/task-unclaim` | Unclaim an in-progress item; optional worktree/branch cleanup |
| `/task-merge` | Merge a `done` branch into `main`, mark `merged`; optional cleanup |

## Typical flow

```text
/task-add …          →  open line in TODO.md
/task-do T-001       →  claim + work → — done task/T-001
/task-merge T-001    →  merge to main → — merged task/T-001
```

Drop or unclaim without finishing:

```text
/task-unclaim T-001      # keep the open TODO line
/task-remove T-001       # delete the ledger line
/task-remove T-001 --dwt --dbr
```

## `/task-add`

```text
/task-add fix typo in README
/task-add bug: crash on missing ID
/task-add task "Add landing CTA" notes:"Hero needs primary button"
```

- Default type is `task` → `T-NNN`. Use `bug:` / type `bug` → `B-NNN`.
- Does **not** start work; only appends to `TODO.md`.
- Run on **primary**, not inside a task worktree.

## `/task-remove`

```text
/task-remove T-001
/task-remove B-002 --dwt --dbr
```

- Deletes the `{ID}` checkbox line from primary `TODO.md` (any state: open / claimed / done / merged).
- Does **not** renumber other IDs.
- If a worktree/branch exists: asks before cleanup unless `--dwt` / `--dbr`.
- Prefer `/task-unclaim` when you only want to clear `@claimed` but keep the open line.
- Run on **primary**.

## `/task-mode`

```text
/task-mode              # show current
/task-mode parallel     # worktrees per task (default)
/task-mode serial       # one task branch on primary
```

| Mode | Provision |
|------|-----------|
| `parallel` | `git worktree add .worktrees/{ID} -b task/{ID}` (or `bug/{ID}`) |
| `serial` | `git switch -c task/{ID}` on primary; claim tail is `@claimed main` |

Missing `.todo-mode` ⇒ `parallel`. Only **new** `/task-do` runs pick up a mode change.

## `/task-do`

```text
/task-do T-001
/task-do T-001 redo              # re-claim / re-run even if done or claimed
/task-do T-001 --wt /tmp/my-T-001   # custom worktree path (parallel)
```

1. **Claim** on primary `TODO.md` (`@claimed …`).
2. **Provision** branch (and worktree in parallel under `.worktrees/{ID}` unless `--wt`).
3. **Handoff (parallel):** open Agent in the worktree and re-run `/task-do {ID}` → **continue** (no re-claim).
4. **Difficulty gate:** confirm `simple`, `medium`, or `complex` before coding.
5. **Execute** → close out as `— done {branch}` on the **worktree/task-branch** `TODO.md` (commit with the work; do not edit primary/`main` ledger).

**Simple:** implement directly.  
**Medium:** brainstorming clarifies and approves a design → implement directly (no Speckit).  
**Complex:** brainstorming clarifies and approves a design doc → Speckit structures/implements from that design (not a second discovery pass). Speckit reuses `GIT_BRANCH_NAME={branch}`.

## `/task-unclaim`

```text
/task-unclaim T-001
/task-unclaim T-001 --dwt --dbr   # also remove worktree / delete branch
```

Rewrites the line to open `- [ ] {ID} {title}`. Does not merge. Branch kept unless you ask to delete.

## `/task-merge`

```text
/task-merge T-001
/task-merge T-001 --dbr -dwt     # after clean merge: delete branch + worktree
/task-merge T-001 --stash        # stash non-TODO.md dirty files, pop after
```

- Accepts `- [x] … — done {branch}` on primary **or** on the worktree/task branch (primary may still show `@claimed`).
- Dirty primary **`TODO.md` only** (typical parallel `@claimed` lines): auto-reconcile via backup — preserve other claims; do not ask to discard.
- Dirty **non-`TODO.md`**: stop unless `--stash` (stash those paths, pop after ledger finalize). Never stash `TODO.md`.
- `TODO.md`-only conflicts: prefer the branch done line for `{ID}`, keep other primary claim rows, then set `— merged` and commit ledger.
- Other conflicts: stop; resolve, commit (or abort), then re-run.
- Without `--dbr` / `-dwt`: asks before deleting branch / worktree.

## `TODO.md` tails

| Tail | Meaning |
|------|---------|
| (none) | open |
| `— @claimed {branch} {worktree}` | parallel in progress |
| `— @claimed main` | serial in progress |
| `— done {branch}` | finished on task branch/worktree; primary may still be `@claimed` |
| `— merged {branch}` | merged via `/task-merge` |
| `— blocked: {reason}` | stopped on execution checkout; still open |

Branch names: `T-*` → `task/{ID}`, `B-*` → `bug/{ID}`.  
Worktree default: `.worktrees/{ID}` (gitignored; override with `--wt`). Legacy sibling `../{project}-{ID}` still discoverable.

## Parallel vs merge conflicts

Parallel shines when tasks touch **different** files. Same hotspot file (e.g. one shared `index.html`) → prefer `serial`, or merge each `done` item quickly, or rebase the task branch onto `main` before `/task-merge`.

## Rules of thumb

- Claim on primary; done/blocked on worktree/task branch; merged on primary after `/task-merge`.
- Never auto-merge into `main`; use `/task-merge`.
- Do not push / open PRs unless you ask separately.
- Skills live under `.cursor/skills/task-*/`.
