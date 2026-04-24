# /progress summary [--format=text|md] [--output=<path>] [--lang=zh|en]

## Command

Generate a summary of the current progress, including in-progress tasks, uncommitted changes, and code change statistics, without performing a Git commit.

## Execution Flow

### Step 1: Detect Project Root Directory

- Search upward from the current directory until finding a directory containing `.git` or `PROGRESS.md` as the project root.

### Step 2: Collect Information

1. **Read PROGRESS.md**:
   - Extract `🎯 Current Focus` tasks
   - Extract `📥 Todo Queue` items
   - Extract `✅ Recently Completed` tasks

2. **Analyze Git Status**:
   - Execute `git status --short` to get uncommitted changes
   - Execute `git diff --stat` to analyze code change statistics
   - Execute `git log -5 --oneline` to get recent commits

### Step 3: Generate Summary

- Consolidate information from PROGRESS.md and Git
- Calculate code change statistics
- Identify WIP tasks
- Generate next steps suggestions

### Step 4: Output Summary

- Default output to terminal in text format
- Support Markdown format with `--format=md`
- Support output to file with `--output=<path>`
- Support language selection with `--lang=zh|en`

## Command Options

- `--format=text`：Output text format (default)
- `--format=md`：Output Markdown format
- `--output=<path>`：Specify output file path
- `--lang=zh`：Generate summary in Chinese
- `--lang=en`：Generate summary in English (default)

## Examples

```
# Generate summary in text format (default)
/progress summary

# Generate summary in Markdown format
/progress summary --format=md

# Generate summary and save to file
/progress summary --output=./progress_summary.md

# Generate summary in Chinese
/progress summary --lang=zh

# Generate summary in Markdown format and save to file
/progress summary --format=md --output=./progress_summary.md
```

## Output Examples

**Text Format Example**:

```
📊 Current Progress Summary
==========================

🎯 Current Focus:
- Implement user authentication API (WIP)
- Optimize database query performance

📥 Todo Queue:
- Fix login page bug
- Design database schema

💻 Uncommitted Changes:
- src/api/auth.js (+25/-5)
- src/utils/db.js (+12/-2)

📈 Code Changes:
- Added: 37 lines
- Deleted: 7 lines
- Modified: 0 lines

🔄 Recent Commits:
- 5e721bf feat: add user authentication endpoint
- 9a34cde fix: resolve database connection issue

💡 Next Steps:
- Complete user authentication API
- Run tests before committing
```

**Markdown Format Example**:

```markdown
# Current Progress Summary

## 🎯 Current Focus
- Implement user authentication API (WIP)
- Optimize database query performance

## 📥 Todo Queue
- Fix login page bug
- Design database schema

## 💻 Uncommitted Changes
- src/api/auth.js (+25/-5)
- src/utils/db.js (+12/-2)

## 📈 Code Changes
- Added: 37 lines
- Deleted: 7 lines
- Modified: 0 lines

## 🔄 Recent Commits
- 5e721bf feat: add user authentication endpoint
- 9a34cde fix: resolve database connection issue

## 💡 Next Steps
- Complete user authentication API
- Run tests before committing
```

**Chinese Text Format Example**:

```
📊 当前进度摘要
================

🎯 当前聚焦：
- 实现用户认证接口（进行中）
- 优化数据库查询性能

📥 待办队列：
- 修复登录页面 bug
- 设计数据库 schema

💻 未提交的更改：
- src/api/auth.js (+25/-5)
- src/utils/db.js (+12/-2)

📈 代码变更：
- 新增：37 行
- 删除：7 行
- 修改：0 行

🔄 最近提交：
- 5e721bf feat: add user authentication endpoint
- 9a34cde fix: resolve database connection issue

💡 下一步：
- 完成用户认证接口
- 提交前运行测试
```

**Chinese Markdown Format Example**:

```markdown
# 当前进度摘要

## 🎯 当前聚焦
- 实现用户认证接口（进行中）
- 优化数据库查询性能

## 📥 待办队列
- 修复登录页面 bug
- 设计数据库 schema

## 💻 未提交的更改
- src/api/auth.js (+25/-5)
- src/utils/db.js (+12/-2)

## 📈 代码变更
- 新增：37 行
- 删除：7 行
- 修改：0 行

## 🔄 最近提交
- 5e721bf feat: add user authentication endpoint
- 9a34cde fix: resolve database connection issue

## 💡 下一步
- 完成用户认证接口
- 提交前运行测试
```