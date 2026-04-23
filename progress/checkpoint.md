# /progress checkpoint [备注]

## 指令

保存当前工作进度，智能选择暂存文件，更新 PROGRESS.md，并生成规范的 Git Commit。

## 执行流程

### Step 1: 自动收集与分析

- 读取 `PROGRESS.md` 当前内容。
- 执行 `git status --short` 获取变更文件列表。
- 执行 `git diff --stat` 了解改动范围。
- 执行 `git log -5 --oneline` 学习历史提交风格。
- 结合代码改动和用户输入的 [备注]，推断本次完成/进行中的任务，生成 Commit Message 草稿。

### Step 2: 合理性分析与确认

- **改动分析**：若改动跨越多个不相关模块（如同时改了 auth 和 payment）或单文件改动超 200 行，提示用户："⚠️ 检测到改动范围较大/跨模块，建议拆分提交。是否拆分？"
- **Commit Message 确认**：展示草稿，等待用户修改或确认。
  - 规范：遵循 Conventional Commits (`type(scope): description`)。
  - WIP：未完成的功能自动在 description 中体现 `wip` 状态。
  - Skip CI：若用户输入包含 `--skip` 或 `#skip-ci`，在标题末尾追加 `[skip ci]`。
  - Footer：所有 commit 末尾统一另起一行追加 `[checkpoint]` 标签。

### Step 3: 智能文件选择

根据改动情况分支处理：

**情况 A：改动小且干净（文件 ≤ 3 且无敏感文件）**
- 自动暂存所有变更文件。
- 静默进入 Step 4。

**情况 B：改动大或有可疑文件**
- 展示**待提交文件预览面板**：
  ```
  ✅ **推荐提交** (3):
    [1] src/api/gateway.py       (+45/-2)
    [2] src/tui/error.py         (+12/-0)
  ⚠️ **建议排除** (检测到配置/日志文件):
    [3] .env                     (未跟踪)
    [4] debug.log                (未跟踪)
  ```
- 提示用户输入指令调整（如："排除 1"、"去掉 .env"、"全部提交"、"只提交 2"）。
- 解析用户意图，构建最终的暂存文件列表。

### Step 4: 写入进度与提交

1. 在本地更新 `PROGRESS.md`：
   - 更新头部时间 `> 最后更新: {CURRENT_DATE}`。
   - 状态迁移：将本次完成的工作移至 ✅ 最近完成（只记日期不记 hash），若是 WIP 则保留在 🎯 并更新描述。
   - 清理：合并重复项，保证 ✅ 分区不超过 5 条。
2. 将更新后的 `PROGRESS.md` 写入磁盘。
3. 执行 Git 操作：
   - `git add <最终确认的文件> PROGRESS.md`
   - `git commit -m "<confirmed_message>"`

## Commit Message 标准格式

**普通提交：**
```
<type>(<scope>): <subject>

[可选正文]

[checkpoint]
```

**跳过 CI 提交：**
```
<type>(<scope>): <subject> [skip ci]

[可选正文]

[checkpoint]
```
