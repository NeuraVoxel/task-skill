 TODO

Project task ledger for `/task-add`, `/task-remove`, and `/task-do`.
IDs: `T-NNN` (task) · `B-NNN` (bug). `[ ]` open · `[x]` done.
Claim: `— @claimed {branch} {worktree}` or `— @claimed main`.
Done: `— done {branch}` on worktree/task-branch `TODO.md` (primary may still show `@claimed` until merge).
Merged: `— merged {branch}` via `/task-merge` (optional `--dbr` / `-dwt`).
Mode: `.todo-mode` → `parallel` (default) | `serial` via `/task-mode`.

## Task

- [x] T-001 创建index.html 实现canvas绘制一个矩形. — done task/T-001
- [x] T-002 canvas 中绘制一个圆形 — merged task/T-002
- [x] T-003 canvas 中绘制一个三角形 — merged task/T-003

## Bug

- [x] B-001 优化圆形的大小为2倍 — merged bug/B-001
- [x] B-002 优化圆形大小为1/4 — done bug/B-002
