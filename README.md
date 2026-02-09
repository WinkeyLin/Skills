# Skills

Kanro 自用的 Agent Skills 仓库，由两部分组成：

- 自编写 Skills（本仓库长期维护）
- 社区维护 Skills（通过 GitHub Actions 定时/手动同步）

## Skills 分类

### 自编写 Skills

- `commit-message`：基于 `git diff --cached` 自动生成 Conventional Commits 风格提交信息
- `explain`：按结构化流程解释代码语法、逻辑、概念与复杂度

### 社区维护 Skills（自动同步）

| 本地目录          | 来源仓库                   | 来源路径                       | 来源 ref |
| ----------------- | -------------------------- | ------------------------------ | -------- |
| `frontend-design` | `anthropics/skills`        | `skills/frontend-design`       | `main`   |
| `skill-creator`   | `anthropics/skills`        | `skills/skill-creator`         | `main`   |
| `code-reviewer`   | `google-gemini/gemini-cli` | `.gemini/skills/code-reviewer` | `main`   |

## 目录结构

```text
.
├─ README.md
├─ .github/
│  └─ workflows/
│     └─ sync-external-skills.yml
├─ commit-message/
│  └─ SKILL.md
├─ explain/
│  └─ SKILL.md
└─ (synced)
   ├─ frontend-design/
   ├─ skill-creator/
   └─ code-reviewer/
```

## Skill: commit-message

### 核心功能

- 读取 `git diff --cached` 获取 staged 变更
- 自动分析变更类型并生成 Conventional Commits 风格 commit message
- 给出变更要点列表，便于直接确认并提交

### 支持的 type

`feat` / `fix` / `refactor` / `style` / `docs` / `test` / `chore` / `perf` / `ci`

### 输出格式

```text
<type>(<scope>): <short description>

- <change point 1>
- <change point 2>
- ...
```

### 触发方式

当用户请求“生成 commit message”“写 commit msg”或使用 `/commit` 时触发。

### 工作流程

1. 获取 staged diff 和文件变更概览
2. 查看最近 commit 风格
3. 分析 diff 并分类 type 与 scope
4. 生成格式化 commit message
5. 用户确认后执行 `git commit`

## Skill: explain

`explain` 采用 **Workflow + Template** 混合结构，共 7 个解释模块：

| 模块                           | 说明                       | 必选 |
| ------------------------------ | -------------------------- | ---- |
| 1. Purpose Summary             | 一两句话说明代码目的       | 是   |
| 2. Syntax Breakdown            | 逐一解释语法元素           | 是   |
| 3. Execution Logic Walkthrough | 按执行顺序逐步追踪         | 是   |
| 4. Key Concepts                | 设计模式 / 语言特性 / 算法 | 是   |
| 5. Data Flow                   | 数据流向（多函数交互时）   | 按需 |
| 6. Error Handling & Edge Cases | 异常处理和边界情况         | 按需 |
| 7. Complexity & Performance    | 时间 / 空间复杂度          | 按需 |

### 自适应规则

会根据代码规模自动调整解释粒度：

- 短片段：聚焦语法与意图
- 中等函数：覆盖执行路径和关键状态变化
- 大文件：强调模块职责、依赖关系和性能风险

并附带 Fibonacci memoization 的完整示例用于参考。

## 推荐使用顺序

1. 先使用 `explain` 理解变更代码
2. 完成并 `git add` 后使用 `commit-message` 生成提交信息
3. 确认 message 后执行提交

## 自动同步外部 Skills（GitHub Actions）

仓库已提供自动同步工作流：`.github/workflows/sync-external-skills.yml`

### 触发方式

- 定时触发：`on.schedule`
- 手动触发：`on.workflow_dispatch`

工作流会记录上一次执行时 `SYNC_TARGETS` 的哈希值；只有当 `SYNC_TARGETS` 发生变化时，才会继续拉取、比较并提交。

手动触发时可设置 `dry_run = true`，只执行同步不提交。`dry_run` 不会更新该哈希记录。

### 如何配置定时任务（cron）

在 workflow 中修改 `schedule`：

```yaml
on:
  schedule:
    - cron: "0 3 * * *"
```

- `cron` 使用 UTC 时区
- 上例表示每天 UTC 03:00 运行一次

### 如何编辑自动同步规则（`SYNC_TARGETS`）

`SYNC_TARGETS` 的每一行是**自定义规则格式**（不是 GitHub Actions 官方语法），会在脚本中按 `|` 分隔解析。

解析逻辑：

```bash
IFS='|' read -r source_repo source_ref source_path destination_path
```

规则语法：

```text
<source_repo>|<source_ref>|<source_path>|<destination_path>
```

字段说明：

- `source_repo`：来源仓库（`owner/repo`）
- `source_ref`：来源分支 / Tag / SHA（如 `main`）
- `source_path`：来源仓库内要拉取的目录
- `destination_path`：同步到当前仓库的目标目录

当前默认规则：

```text
anthropics/skills|main|skills/frontend-design|frontend-design
anthropics/skills|main|skills/skill-creator|skill-creator
google-gemini/gemini-cli|main|.gemini/skills/code-reviewer|code-reviewer
```

### 规则编写建议

1. 每行必须正好 4 段，并使用 `|` 分隔
2. `destination_path` 会被覆盖更新（同步时会先删除再拷贝）
3. `destination_path` 请使用仓库内相对路径，不要使用绝对路径或 `..`
4. 修改规则后建议先手动运行一次 `dry_run = true` 进行验证

### 首次启用建议

1. 在仓库设置中确认 Actions workflow permissions 为 **Read and write permissions**
2. 手动运行一次 workflow 并使用 `dry_run = true` 检查结果
3. 确认目录变更符合预期后，再使用 `dry_run = false` 或等待定时任务自动提交
