# task-do reference

## Difficulty heuristics

**Prefer simple when:**

- Single-file or known-location edit (typo, copy, small config)
- Clear acceptance criteria already in the TODO line / notes
- No new product behavior or architecture decision

**Prefer complex when:**

- New feature or multi-file / cross-area change
- Requirements or success criteria unclear
- Needs a design doc or Speckit artifacts before coding

Always propose a difficulty; the user may override.

## Complex pipeline (default)

1. `brainstorming` → approved design/requirements (often `docs/superpowers/specs/YYYY-MM-DD-*-design.md`)
2. `speckit-specify`
3. `speckit-plan`
4. `speckit-implement`

Optional only if stuck: `speckit-clarify`, `speckit-tasks`, `speckit-analyze`, `speckit-converge`.

## Mode file

- Primary path: `.todo-mode` (gitignored)
- Contents: one line, `parallel` or `serial`
- Missing file ⇒ `parallel`
- Change via `/task-mode`

## Worktree path

Default sibling of primary: `../notes-{ID}` (e.g. `../notes-T-001`).

## TODO.md entry shapes

```markdown
## Task

- [ ] T-001 short title
- [ ] T-002 short title — @claimed task/T-002 ../notes-T-002
- [x] T-003 short title — done task/T-003
- [ ] T-004 short title — blocked: waiting on API keys

## Bug

- [ ] B-001 title — @claimed main
- [x] B-002 title — done bug/B-002
```

| Tail | Meaning |
|------|---------|
| (none) | open, unclaimed |
| `— @claimed {branch} {worktree}` | parallel in progress |
| `— @claimed main` | serial in progress |
| `— done {branch}` | finished; branch kept, not auto-merged |
| `— blocked: {reason}` | stopped; still open checkbox |

Match entries by `{ID}` on the checkbox line. `[ ]` = open · `[x]` = done.

Branch: `T-*` → `task/{ID}` · `B-*` → `bug/{ID}`.

## Git cheat sheet

```bash
# parallel provision (from primary, on main)
git worktree add "../notes-T-001" -b task/T-001

# list / remove
git worktree list
git worktree remove "../notes-T-001"

# serial
git switch -c task/T-001
# … commits …
git switch main
```
