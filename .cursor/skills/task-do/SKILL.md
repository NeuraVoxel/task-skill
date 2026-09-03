---
name: task-do
description: >-
  Claims a TODO.md item by id (T-NNN or B-NNN), provisions a worktree (parallel)
  or task branch (serial) from .todo-mode, runs simple/medium/complex execution,
  then closes out with a done/blocked ledger line on the task worktree/branch —
  never edit primary/main TODO.md at close-out. No auto-merge to main. Use when
  the user runs /task-do, asks to work a task number, or process an item from
  TODO.md.
disable-model-invocation: true
---

# task-do

Orchestrate one ledger item: **claim → provision → execute → close out** (or **continue** after handoff: skip claim/provision-create → cwd gate → execute → close out).

## 1. Specs

- `docs/superpowers/specs/2026-08-31-todo-task-workflow-design.md`
- `docs/superpowers/specs/2026-08-31-parallel-todo-worktree-design.md`
- Heuristics and git cheat sheet: [reference.md](reference.md)

**Ledger write split**

| Phase | Where to edit `TODO.md` |
|-------|-------------------------|
| **Claim** (§5) | **Primary only** (so other runs see `@claimed`) |
| **Close-out / blocked** (§10, medium/complex blocked) | **Task worktree / task branch only** — never primary/`main` |

In parallel, the executing agent (continue in the worktree) must write done/blocked on the **worktree** `TODO.md` and commit it on `{branch}`. Do **not** open or patch primary's `TODO.md` after claim. Serial marks done/blocked on the task branch **before** any `git switch main`.

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
3. **Parallel status peek (worktree):** if a claimed worktree path is present and that path exists, also read **that worktree's** `TODO.md` for `{ID}`. Prefer the worktree line for **done / blocked** decisions (primary may still show `@claimed` after close-out).
4. Classify this run (mutually exclusive). Parse claimed worktree path from the primary line when present (`— @claimed {branch} {path}`). Serial claims use `— @claimed main` (no worktree path).

| Outcome | When | Next |
|---------|------|------|
| **stop** | Worktree (or primary) line is `- [x] … — done` / `— merged`, and user did **not** ask to **redo** | Report done; stop |
| **stop** | **Parallel** claim (`@claimed` with worktree path) for this `{ID}`, user did **not** ask to **redo**, cwd is **not** that claimed worktree, and worktree (if readable) is **not** already done | Report claimed; stop |
| **stop** | **Serial** claim (`@claimed main`) for this `{ID}`, user did **not** ask to **redo**, and cwd is **not** primary | Report claimed; stop |
| **continue** | **Parallel:** primary `@claimed` for this `{ID}` with worktree path, user did **not** ask to **redo**, cwd **is** that claimed worktree, and worktree line is **not** already `- [x]` done | **Skip §5 Claim and §6 provision-create.** Do not rewrite the claim line on primary. Reuse existing worktree/branch → §7 → difficulty+ |
| **continue** | **Serial:** primary `@claimed main` for this `{ID}`, user did **not** ask to **redo**, and cwd **is** primary | **Skip §5 Claim and §6 provision-create.** Do not rewrite the claim line. Stay on primary (serial has no worktree handoff) → §7 → difficulty+ |
| **redo** | User explicitly asked to **redo** (even if `[x]` or `@claimed`) | §5 Claim (overwrite allowed) → §6 → … |
| **fresh claim** | Open `- [ ]` with no `@claimed` (and no worktree done peek) | §5 Claim → §6 → … |

**Continue invariants:** Claim stays primary-only. A continue run must **not** re-write the primary `@claimed` line and must **not** run Claim (§5). Parallel continue may start from the task worktree; serial continue only from primary. Close-out / blocked edits go to the **execution checkout** (worktree or task branch), never primary/`main`.

## 5. Claim (primary only)

Skip this entire section on **continue**.

1. **cwd must be primary.** If cwd looks like a task worktree (`{project}-T-*` / `{project}-B-*`), stop and tell the user to run claim/redo from primary. (Continue-from-worktree is only for parallel — see §4.)
2. **Project name:** `{project}` = basename of the primary repo root (e.g. primary `/…/task-skill` → `task-skill`). Use it for default worktree directory names.
3. Branch name: `T-*` → `task/{ID}`; `B-*` → `bug/{ID}`.
4. **Serial exclusivity (before any write):** when mode is `serial`, scan primary `TODO.md` for `@claimed` on **other** IDs (not the target `{ID}`). If any other ID is `@claimed`, **stop without mutating** `TODO.md`; ask to finish or `/task-unclaim` first. Same-ID `@claimed` is not a blocker (allows **redo** overwrite).
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

1. Suggest `simple`, `medium`, or `complex` with 2–4 sentences of rationale (see reference).
2. **Wait for user confirmation or override.** Do not implement before this.
3. After confirmation, remember difficulty for the close-out reply (not a separate ledger field).

## 9. Execute

### 9a. Simple path

1. Implement the request directly (edit files / notes as needed).
2. Do **not** invoke brainstorming or Speckit.

### 9b. Medium path

1. Follow the **brainstorming** skill until the user approves the design/requirements.
2. Write design under `docs/superpowers/specs/` when brainstorming requires a design doc. That approved design (or the in-chat approved design) is the **requirements source of truth** for implementation.
3. After approval, **implement directly** on the task branch (edit files / notes as needed).
4. Do **not** invoke Speckit (`speckit-specify` / `plan` / `implement` or related).
5. If brainstorming is unavailable: tell the user, save what you have, leave execution-checkout ledger `- [ ]` with `— blocked: {reason}` (see §10 Blocked), stop.

### 9c. Complex path

1. Follow the **brainstorming** skill until the user approves the design/requirements.
2. Write design under `docs/superpowers/specs/` when brainstorming requires a design doc. That approved design is the **requirements source of truth** for Speckit.
3. After approval, run Speckit **as a structuring / execution pipeline**, not a second discovery pass:
   - **Input:** pass the approved design (path + substance) into `speckit-specify` as the feature description. Prefer quoting or summarizing from the design doc over inventing new scope.
   - **Do not** re-run brainstorming-style discovery, re-ask decisions already settled in the design, or draft a competing design narrative.
   - **May** ask follow-ups only when the design is missing an implementation-critical detail, or contradicts itself / the codebase.
   - Default: skip `speckit-clarify` / `speckit-tasks`. Use them only if blocked.
   - Order: read and follow `speckit-specify` → `speckit-plan` → `speckit-implement`.
4. **Reuse the task branch for Speckit** (no second branch):
   - Provision (§6) already created `{branch}` (`task/{ID}` or `bug/{ID}`). Stay on it for the whole Speckit pipeline.
   - When invoking `speckit-specify`, pass `GIT_BRANCH_NAME={branch}` so any `before_specify` git hook reuses that exact name instead of creating a Speckit-style branch (e.g. `003-short-name`).
   - Spec directory under `specs/` may still be auto-named independently; do not treat that directory name as a git branch to create or switch to.
   - Do **not** `git switch -c` / create another branch during Speckit unless the user explicitly asks.
5. If brainstorming or Speckit skills are unavailable: tell the user, save what you have, leave execution-checkout ledger `- [ ]` with `— blocked: {reason}` (see §10 Blocked), stop.

**Reminders while executing**

- **Parallel:** edit the **worktree** `TODO.md` only at close-out / blocked. Never edit primary/`main` `TODO.md` from the worktree agent.
- **Serial:** edit `TODO.md` on the **task branch** at close-out / blocked; do not edit after switching to `main`.
- Do not commit `.obsidian/` or other local noise; follow `.gitignore`.

## 10. Close out

### Parallel (success)

1. In the task worktree: set local `TODO.md` to `- [x] {ID} {title} — done {branch}`.
2. Commit content changes **and** that `TODO.md` update on `{branch}` (message like `task({ID}): …` or `bug({ID}): …`). No push, no merge.
3. Do **not** edit primary/`main` `TODO.md` (it may still show `@claimed` until `/task-merge`).
4. Keep the worktree (do not remove).

### Serial (success)

1. On the task branch: set `TODO.md` to `- [x] {ID} {title} — done {branch}`.
2. Commit content changes **and** that `TODO.md` update on `{branch}`.
3. `git switch main`
4. Do **not** edit `TODO.md` on `main` after the switch (primary claim line may still show `@claimed` until `/task-merge`).

### Blocked (either mode)

1. On the **execution checkout** (parallel: worktree; serial: task branch): keep `- [ ]`; set/replace trailing notes to `— blocked: {reason}`.
2. Commit that ledger update on `{branch}` when practical.
3. Do **not** edit primary/`main` `TODO.md`. Do not mark done; do not merge.

### Reply

Briefly: ID, mode, difficulty, path taken (simple/medium/complex), branch, result (done / blocked / handoff). Note that primary may still show `@claimed` until merge.

## 11. Gates

- Stop on missing id, missing entry, claim race (**fresh claim** only — not **redo**), serial exclusivity (other IDs’ `@claimed` before write; same-ID allowed for **redo**), claim/redo from a worktree cwd, wrong cwd (parallel execute), dirty serial switch, unconfirmed difficulty, or user rejecting scope.
- **Continue** must skip Claim and must not rewrite the primary `@claimed` line.
- Close-out **may** commit on the task branch (including worktree/`TODO.md` done/blocked). Close-out must **not** patch primary/`main` `TODO.md`.
- Never push, open a PR, or auto-merge task branches into `main`.
- Medium: brainstorming then direct implement; never Speckit.
- Complex + Speckit: keep work on `{branch}`; pass `GIT_BRANCH_NAME={branch}` into `speckit-specify`; never create a second feature branch.
- Complex handoff: Speckit consumes the approved brainstorming design; do not re-discover requirements or contradict settled decisions unless the design is incomplete or inconsistent.
- Do not expand scope beyond the TODO entry without asking.
