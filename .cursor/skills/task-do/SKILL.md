---
name: task-do
description: >-
  Claims a TODO.md item by id (T-NNN or B-NNN), provisions a worktree (parallel)
  or task branch (serial) from .todo-mode, runs simple/complex execution, then
  closes out with a done/blocked ledger line on primary — no auto-merge to main.
  Use when the user runs /task-do, asks to work a task number, or process an
  item from TODO.md.
disable-model-invocation: true
---

# task-do

Orchestrate one ledger item: **claim → provision → execute → close out** (or **continue** after handoff: skip claim/provision-create → cwd gate → execute → close out).

## 1. Specs

- `docs/superpowers/specs/2026-08-31-todo-task-workflow-design.md`
- `docs/superpowers/specs/2026-08-31-parallel-todo-worktree-design.md`
- Heuristics and git cheat sheet: [reference.md](reference.md)

**Primary only for claim/ledger:** Edit `TODO.md` only on the primary vault checkout. Task worktrees must not treat their copy of `TODO.md` as ledger truth. From a worktree cwd, edit primary via its absolute path (not the worktree copy).

## 2. Inputs

- Required: task id (`T-001`, `B-002`, …) from `/task-do <ID>` or the message.
- Optional: explicit **redo** (re-claim / re-run a done or already-claimed item).
- If id missing: ask once, then stop until provided.

## 3. Resolve mode

1. On **primary**, read `.todo-mode` if present; otherwise treat mode as `parallel`.
2. Valid values: `parallel` | `serial`. Mode affects provision only for this run.

## 4. Load

1. On primary, read `TODO.md`. Find the checkbox line matching `{ID}` under `## Task` or `## Bug`:
   `- [ ] {ID} …` or `- [x] {ID} …`
2. If not found: report and stop.
3. Classify this run (mutually exclusive). Parse claimed worktree path from the line when present (`— @claimed {branch} {path}`). Serial claims use `— @claimed main` (no worktree path).

| Outcome | When | Next |
|---------|------|------|
| **stop** | `- [x]` and user did **not** ask to **redo** | Report done; stop |
| **stop** | **Parallel** claim (`@claimed` with worktree path) for this `{ID}`, user did **not** ask to **redo**, and cwd is **not** that claimed worktree | Report claimed; stop |
| **stop** | **Serial** claim (`@claimed main`) for this `{ID}`, user did **not** ask to **redo**, and cwd is **not** primary | Report claimed; stop |
| **continue** | **Parallel:** line `@claimed` for this `{ID}` with worktree path, user did **not** ask to **redo**, and cwd **is** that claimed worktree | **Skip §5 Claim and §6 provision-create.** Do not rewrite the claim line. Reuse existing worktree/branch → §7 → difficulty+ |
| **continue** | **Serial:** line `@claimed main` for this `{ID}`, user did **not** ask to **redo**, and cwd **is** primary | **Skip §5 Claim and §6 provision-create.** Do not rewrite the claim line. Stay on primary (serial has no worktree handoff) → §7 → difficulty+ |
| **redo** | User explicitly asked to **redo** (even if `[x]` or `@claimed`) | §5 Claim (overwrite allowed) → §6 → … |
| **fresh claim** | Open `- [ ]` with no `@claimed` | §5 Claim → §6 → … |

**Continue invariants:** Claiming and other ledger edits still happen only on primary. A continue run must **not** re-write the TODO claim line and must **not** run Claim (§5). Parallel continue may start from the task worktree; serial continue only from primary.

## 5. Claim (primary only)

Skip this entire section on **continue**.

1. **cwd must be primary.** If cwd looks like a task worktree (`{project}-T-*` / `{project}-B-*`), stop and tell the user to run claim/redo from primary. (Continue-from-worktree is only for parallel — see §4.)
2. **Project name:** `{project}` = basename of the primary repo root (e.g. primary `/…/task-skill` → `task-skill`). Use it for default worktree directory names.
3. Branch name: `T-*` → `task/{ID}`; `B-*` → `bug/{ID}`.
4. **Serial exclusivity (before any write):** when mode is `serial`, scan primary `TODO.md` for `@claimed` on **other** IDs (not the target `{ID}`). If any other ID is `@claimed`, **stop without mutating** `TODO.md`; ask to finish or `/task-release` first. Same-ID `@claimed` is not a blocker (allows **redo** overwrite).
5. **Double-read race check** — re-read the target line immediately before write:
   - **Fresh claim (not redo):** if another run already set `@claimed` or `[x]`, **stop without overwriting**.
   - **Redo:** race check does **not** block; you **may** overwrite existing `@claimed` or `[x]` and re-claim for this run.
6. Set the claim tail on primary `TODO.md` (keep `- [ ]` and title; clear prior `done` / `blocked` / old claim tails):
   - **parallel:** `— @claimed {branch} ../{project}-{ID}`  
     Default worktree path: `../{project}-{ID}` (relative to primary, or the absolute equivalent; write the path you will use).
   - **serial:** `— @claimed main`
7. Write primary `TODO.md`.

## 6. Provision

On **continue**: skip create; reuse the existing worktree/branch from the claim line → go to §7.

### Parallel

1. **Fresh claim / redo only:** From primary (on `main`), run:
   `git worktree add ../{project}-{ID} -b {branch}`
2. If the worktree or branch already exists: reuse; ensure claim line path matches (update claim path on primary only if this run performed Claim).
3. **Handoff stop:** if cwd is still primary after provision, tell the user to open Agent in the worktree (or `cd` there) and re-run `/task-do {ID}`. That re-run is **continue** (§4) — it must not re-claim. Stop here.
4. If cwd is already the task worktree: continue to §7.

### Serial

1. **Fresh claim / redo only:** `git switch -c {branch}` or `git switch {branch}` if the branch already exists.
2. If switch fails because the working tree is dirty: stop and ask the user to clean up. Do **not** stash.

## 7. cwd gate

- **parallel:** execution (§8 onward) must run with cwd = the task worktree. If not, stop and print the expected path.
- **serial:** execution runs in primary on the task branch.

## 8. Difficulty gate (hard stop)

1. Suggest `simple` or `complex` with 2–4 sentences of rationale (see reference).
2. **Wait for user confirmation or override.** Do not implement before this.
3. After confirmation, remember difficulty for the close-out reply (not a separate ledger field).

## 9. Execute

### 9a. Simple path

1. Implement the request directly (edit files / notes as needed).
2. Do **not** invoke brainstorming or Speckit.

### 9b. Complex path

1. Follow the **brainstorming** skill until the user approves the design/requirements.
2. Write design under `docs/superpowers/specs/` when brainstorming requires a design doc.
3. After approval, in order:
   - Read and follow `.cursor/skills/speckit-specify/SKILL.md`
   - Read and follow `.cursor/skills/speckit-plan/SKILL.md`
   - Read and follow `.cursor/skills/speckit-implement/SKILL.md`
4. Default: skip clarify/tasks. Only add them if blocked.
5. If brainstorming or Speckit skills are unavailable: tell the user, save what you have, leave primary ledger `- [ ]` with `— blocked: {reason}`, stop.

**Reminders while executing**

- Do **not** edit the worktree copy of `TODO.md`; ledger updates go on primary at close-out / blocked (use primary's absolute path if cwd is a worktree).
- Do not commit `.obsidian/` or other local noise; follow `.gitignore`.

## 10. Close out

### Parallel (success)

1. In the task worktree: if there are content changes, commit on the task branch (message like `task({ID}): …` or `bug({ID}): …`). No push, no merge.
2. On **primary**, set: `- [x] {ID} {title} — done {branch}`
3. Keep the worktree (do not remove).

### Serial (success)

1. Commit content on the task branch if there are changes.
2. `git switch main`
3. On primary `TODO.md`, set: `- [x] {ID} {title} — done {branch}`

### Blocked (either mode)

1. Keep `- [ ]` on **primary** ledger; set/replace trailing notes to `— blocked: {reason}`.
2. Do not mark done; do not merge.

### Reply

Briefly: ID, mode, difficulty, path taken (simple/complex), branch, result (done / blocked / handoff).

## 11. Gates

- Stop on missing id, missing entry, claim race (**fresh claim** only — not **redo**), serial exclusivity (other IDs’ `@claimed` before write; same-ID allowed for **redo**), claim/redo from a worktree cwd, wrong cwd (parallel execute), dirty serial switch, unconfirmed difficulty, or user rejecting scope.
- **Continue** must skip Claim and must not rewrite the `@claimed` line; ledger edits remain primary-only (close-out / blocked).
- Close-out **may** commit on the task branch and **may** update the ledger on primary/`main` when finishing.
- Never push, open a PR, or auto-merge task branches into `main`.
- Do not expand scope beyond the TODO entry without asking.
