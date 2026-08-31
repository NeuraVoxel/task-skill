# task-skill

Cursor Agent skills that turn repo-root `TODO.md` into a claim → work → done → merge workflow. Ledger edits happen on the **primary** checkout only; parallel work uses git worktrees.

## Skills

| Command | Role |
|---------|------|
| `/task-add` | Append a task (`T-NNN`) or bug (`B-NNN`) to `TODO.md` |
| `/task-mode` | Show or set `parallel` / `serial` (`.todo-mode`, gitignored) |
| `/task-do` | Claim an item, provision branch/worktree, implement, mark `done` |
| `/task-release` | Unclaim an in-progress item; optional worktree/branch cleanup |
| `/task-merge` | Merge a `done` branch into `main`, mark `merged`; optional cleanup |

## Typical flow

```text
/task-add …          →  open line in TODO.md
/task-do T-001       →  claim + work → — done task/T-001
/task-merge T-001    →  merge to main → — merged task/T-001
```

Drop or free a claim without finishing:

```text
/task-release T-001
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

## `/task-mode`

```text
/task-mode              # show current
/task-mode parallel     # worktrees per task (default)
/task-mode serial       # one task branch on primary
```

| Mode | Provision |
|------|-----------|
| `parallel` | `git worktree add ../{project}-{ID} -b task/{ID}` (or `bug/{ID}`) |
| `serial` | `git switch -c task/{ID}` on primary; claim tail is `@claimed main` |

Missing `.todo-mode` ⇒ `parallel`. Only **new** `/task-do` runs pick up a mode change.

## `/task-do`

```text
/task-do T-001
/task-do T-001 redo    # re-claim / re-run even if done or claimed
```

1. **Claim** on primary `TODO.md` (`@claimed …`).
2. **Provision** branch (and worktree in parallel).
3. **Handoff (parallel):** open Agent in the worktree and re-run `/task-do {ID}` → **continue** (no re-claim).
4. **Difficulty gate:** confirm `simple` or `complex` before coding.
5. **Execute** → close out as `— done {branch}` (no auto-merge).

**Simple:** implement directly.  
**Complex:** brainstorming → Speckit (`specify` / `plan` / `implement`). Speckit reuses `GIT_BRANCH_NAME={branch}` so it does not create a second branch.

## `/task-release`

```text
/task-release T-001
/task-release T-001 --dwt --dbr   # also remove worktree / delete branch
```

Rewrites the line to open `- [ ] {ID} {title}`. Does not merge. Branch kept unless you ask to delete.

## `/task-merge`

```text
/task-merge T-001
/task-merge T-001 --dbr -dwt     # after clean merge: delete branch + worktree
```

- Only accepts `- [x] … — done {branch}`.
- On conflict: stops; resolve, commit (or abort), then re-run.
- Without flags: asks before deleting branch / worktree.

## `TODO.md` tails

| Tail | Meaning |
|------|---------|
| (none) | open |
| `— @claimed {branch} {worktree}` | parallel in progress |
| `— @claimed main` | serial in progress |
| `— done {branch}` | finished; not yet on `main` |
| `— merged {branch}` | merged via `/task-merge` |
| `— blocked: {reason}` | stopped; still open |

Branch names: `T-*` → `task/{ID}`, `B-*` → `bug/{ID}`.  
Worktree default: `../{project}-{ID}` (sibling of primary).

## Parallel vs merge conflicts

Parallel shines when tasks touch **different** files. Same hotspot file (e.g. one shared `index.html`) → prefer `serial`, or merge each `done` item quickly, or rebase the task branch onto `main` before `/task-merge`.

## Rules of thumb

- Ledger truth = primary `TODO.md` only.
- Never auto-merge into `main`; use `/task-merge`.
- Do not push / open PRs unless you ask separately.
- Skills live under `.cursor/skills/task-*/`.
