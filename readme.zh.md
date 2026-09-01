# task-skill

一组 Cursor Agent Skill：把仓库根目录的 `TODO.md` 变成「认领 → 开发 → 完成 → 合并」工作流。账本只在 **primary（主工作区）** 上改；并行开发用 git worktree。

## Skills

| 命令 | 作用 |
|------|------|
| `/task-add` | 向 `TODO.md` 追加任务（`T-NNN`）或缺陷（`B-NNN`） |
| `/task-mode` | 查看或设置 `parallel` / `serial`（写入 `.todo-mode`，已 gitignore） |
| `/task-do` | 认领条目、准备分支/worktree、实现，并标记为 `done` |
| `/task-unclaim` | 取消进行中的认领；可选清理 worktree / 分支 |
| `/task-merge` | 将 `done` 分支合并进 `main`，标记为 `merged`；可选清理 |

## 典型流程

```text
/task-add …          →  在 TODO.md 新增一行
/task-do T-001       →  认领 + 开发 → — done task/T-001
/task-merge T-001    →  合并到 main → — merged task/T-001
```

不想做完、只想取消认领：

```text
/task-unclaim T-001
```

## `/task-add`

```text
/task-add fix typo in README
/task-add bug: crash on missing ID
/task-add task "Add landing CTA" notes:"Hero needs primary button"
```

- 默认类型为 `task` → `T-NNN`。写 `bug:` 或指定类型 `bug` → `B-NNN`。
- **不会**开始实现，只往 `TODO.md` 追加。
- 在 **primary** 上执行，不要在任务 worktree 里跑。

## `/task-mode`

```text
/task-mode              # 查看当前模式
/task-mode parallel     # 每任务一个 worktree（默认）
/task-mode serial       # 在 primary 上使用任务分支
```

| 模式 | 准备方式 |
|------|----------|
| `parallel` | `git worktree add ../{project}-{ID} -b task/{ID}`（缺陷为 `bug/{ID}`） |
| `serial` | 在 primary 上 `git switch -c task/{ID}`；认领尾注为 `@claimed main` |

没有 `.todo-mode` 时视为 `parallel`。模式变更只影响**之后新的** `/task-do`。

## `/task-do`

```text
/task-do T-001
/task-do T-001 redo    # 即使已 done / 已认领，也重新认领并执行
```

1. 在 primary 的 `TODO.md` 上 **认领**（`@claimed …`）。
2. **准备**分支（parallel 时再建 worktree）。
3. **交接（parallel）：** 在 worktree 里打开 Agent，再跑一次 `/task-do {ID}` → **continue**（不再认领）。
4. **难度确认：** 编码前确认 `simple`、`medium` 或 `complex`。
5. **执行** → 收尾为 `— done {branch}`（不会自动合并进 `main`）。

**simple：** 直接改代码。  
**medium：** brainstorming 澄清并批准 design → 直接实现（不走 Speckit）。  
**complex：** brainstorming 澄清并批准 design → Speckit 以该 design 为输入做结构化与实现（不再重新发现需求）。Speckit 通过 `GIT_BRANCH_NAME={branch}` 复用已有分支。

## `/task-unclaim`

```text
/task-unclaim T-001
/task-unclaim T-001 --dwt --dbr   # 同时删除 worktree / 分支
```

把该行改回开放的 `- [ ] {ID} {title}`。不做合并。除非你明确要求，否则保留分支。

## `/task-merge`

```text
/task-merge T-001
/task-merge T-001 --dbr -dwt     # 干净合并后：删分支 + 删 worktree
```

- 只接受 `- [x] … — done {branch}`。
- 发生冲突时立刻停下；解决后 `commit`（或 `merge --abort`），再重新执行。
- 未带参数时：删除分支 / worktree 前会先询问。

## `TODO.md` 尾注

| 尾注 | 含义 |
|------|------|
| （无） | 开放、未认领 |
| `— @claimed {branch} {worktree}` | parallel 进行中 |
| `— @claimed main` | serial 进行中 |
| `— done {branch}` | 已完成，尚未进 `main` |
| `— merged {branch}` | 已通过 `/task-merge` 合并 |
| `— blocked: {reason}` | 受阻；复选框仍为开放 |

分支命名：`T-*` → `task/{ID}`，`B-*` → `bug/{ID}`。  
默认 worktree：`../{project}-{ID}`（与 primary 同级）。

## 并行与合并冲突

任务改动的**文件互不重叠**时，parallel 最合适。若都改同一热点文件（例如共用一个 `index.html`），建议改用 `serial`，或每个 `done` 尽快 `/task-merge`，或在合并前先把任务分支 rebase/merge 到最新 `main`。

## 使用约定

- 账本唯一真相源 = primary 上的 `TODO.md`。
- 不要自动合并进 `main`；用 `/task-merge`。
- 除非另行要求，否则不 push、不开 PR。
- Skill 位于 `.cursor/skills/task-*/`。

英文版见 [readme.md](readme.md)。
