# task-do reference

## Difficulty heuristics

**Prefer simple when:**

- Single-file or known-location edit (typo, copy, small config)
- Clear acceptance criteria already in the TODO line / notes
- No new product behavior or architecture decision

**Prefer medium when:**

- Bounded change that still needs intent/design clarification before coding
- Multi-file or small feature, but success criteria become clear after a short design
- Needs brainstorming approval, but **not** Speckit artifacts / full structuring pipeline

**Prefer complex when:**

- New feature or multi-file / cross-area change with unclear or contested requirements
- Needs a durable design doc **and** Speckit artifacts before coding
- Architecture, contracts, or multi-step planning benefit from specify → plan → implement

Always propose a difficulty; the user may override.

## Execution paths (summary)

| Difficulty | After confirmation |
|------------|--------------------|
| **simple** | Implement directly — no brainstorming, no Speckit |
| **medium** | Brainstorming → approved design → implement directly — no Speckit |
| **complex** | Brainstorming → approved design → Speckit specify → plan → implement |

## Medium pipeline

1. `brainstorming` → approved design/requirements (in chat and/or `docs/superpowers/specs/YYYY-MM-DD-*-design.md`) — **source of truth**
2. Implement directly on `{branch}` (`task/{ID}` or `bug/{ID}`)
3. Do **not** run Speckit

## Complex pipeline

1. `brainstorming` → approved design/requirements (often `docs/superpowers/specs/YYYY-MM-DD-*-design.md`) — **source of truth**
2. `speckit-specify` with that design as input + `GIT_BRANCH_NAME={branch}` (structure the approved requirements; do not re-discover or re-decide; reuse `task/{ID}` or `bug/{ID}`)
3. `speckit-plan`
4. `speckit-implement`

Optional only if stuck (missing detail / contradiction / blocked): `speckit-clarify`, `speckit-tasks`, `speckit-analyze`, `speckit-converge`.

Division of labor: brainstorming clarifies intent; Speckit structures and implements. Speckit’s `specs/<prefix>-<short-name>/` directory name is independent of the git branch; only `{branch}` from `/task-do` is the work branch.

## Mode file

- Primary path: `.todo-mode` (gitignored)
- Contents: one line, `parallel` or `serial`
- Missing file ⇒ `parallel`
- Change via `/task-mode`

## Worktree path

- `{project}` = basename of the primary repo root (e.g. `task-skill`).
- **Default:** `{primary}/.worktrees/{ID}` (e.g. `.worktrees/T-001`). Aligns with Superpowers `using-git-worktrees` (project-local `.worktrees/`).
- **Override:** `/task-do {ID} --wt <path>` — absolute, or relative to primary. Write the resolved path into the claim line.
- **Ignore:** `.worktrees/` must be gitignored (`git check-ignore -q .worktrees`). If not, add it to `.gitignore` before `git worktree add`.
- **Legacy compat:** older claims may use sibling `../{project}-{ID}`; discovery still accepts those paths.
- **Task worktree detection** (cwd “looks like” a task worktree): claim path match, **or** cwd under `{primary}/.worktrees/`, **or** basename matches `{project}-T-*` / `{project}-B-*` (legacy sibling).

## Worktree discovery (cleanup / merge / peek)

1. Claimed `{path}` from `@claimed {branch} {path}` when present and listed.
2. Else `git worktree list` entry whose branch is `{branch}`.
3. Else `.worktrees/{ID}` if listed.
4. Else sibling `../{project}-{ID}` if listed (legacy).
5. Else: no worktree.

## TODO.md entry shapes

```markdown
## Task

- [ ] T-001 short title
- [ ] T-002 short title — @claimed task/T-002 .worktrees/T-002
- [x] T-003 short title — done task/T-003
- [x] T-004 short title — merged task/T-004
- [ ] T-005 short title — blocked: waiting on API keys

## Bug

- [ ] B-001 title — @claimed main
- [x] B-002 title — done bug/B-002
```

| Tail | Meaning |
|------|---------|
| (none) | open, unclaimed |
| `— @claimed {branch} {worktree}` | parallel in progress (primary claim) |
| `— @claimed main` | serial in progress (primary claim) |
| `— done {branch}` | finished on **task branch/worktree** `TODO.md`; primary may still show `@claimed` until `/task-merge` |
| `— merged {branch}` | merged into `main` via `/task-merge` |
| `— blocked: {reason}` | stopped on execution checkout; still open checkbox |

Match entries by `{ID}` on the checkbox line. `[ ]` = open · `[x]` = done.

**Where lines are written:** claim → primary; done/blocked → worktree (parallel) or task branch (serial); merged → primary after `/task-merge`.

Branch: `T-*` → `task/{ID}` · `B-*` → `bug/{ID}`.

## Git cheat sheet

```bash
# parallel provision (from primary, on main)
mkdir -p .worktrees
git check-ignore -q .worktrees || echo '.worktrees/' >> .gitignore
git worktree add .worktrees/T-001 -b task/T-001

# override
git worktree add /tmp/my-T-001 -b task/T-001

# list / remove
git worktree list
git worktree remove .worktrees/T-001

# serial
git switch -c task/T-001
# … commits …
git switch main
```
