# /progress brief [daily|weekly] [--format=md|html|pdf]

## Command

Generate project development daily or weekly reports, including task completion status, code change statistics, and progress trend analysis.

## Execution Flow

### Step 1: Data Collection

- Read `📅 Task History` section in `PROGRESS.md`
- Execute `git log --since="1 day ago"` or `git log --since="1 week ago"` to get commit history
- Execute `git diff --stat` to analyze code changes
- Parse `✅ Recently Completed` and `🎯 Current Focus` sections to get latest status

### Step 2: Report Generation

Generate reports according to the specified format:

**Markdown Format**:
- Include task completion status
- Code change statistics
- Progress trend analysis
- Next steps plan

**HTML Format**:
- Include Markdown content
- Embed Chart.js to generate code activity and progress trend charts
- Responsive design, support mobile viewing

**PDF Format**:
- Generated based on HTML format
- Use browser print function or Pandoc conversion

### Step 3: Output and Export

- Default output to terminal
- Support specifying output file path
- Automatically create output directory (if it doesn't exist)

## Command Options

- `daily`：Generate daily report (default)
- `weekly`：Generate weekly report
- `--format=md`：Output Markdown format (default)
- `--format=html`：Output HTML format
- `--format=pdf`：Output PDF format
- `--output=<path>`：Specify output file path
- `--lang=zh`：Generate report in Chinese
- `--lang=en`：Generate report in English (default)

## Examples

```
# Generate daily report (Markdown format, automatic language detection)
/progress brief daily

# Generate weekly report (HTML format, English)
/progress brief weekly --format=html --lang=en

# Generate daily report and export to specified file (Chinese)
/progress brief daily --format=pdf --lang=zh --output=./reports/daily_zh.pdf

# Generate weekly report and export to specified file (English)
/progress brief weekly --format=html --lang=en --output=./reports/weekly_en.html
```

## Report Content

**Chinese Daily Report Example**:

```markdown
# 项目日报 - 2026-04-24

## 完成任务
- ✅ 实现用户认证接口
- ✅ 修复登录页面 bug

## 进行中任务
- 🔄 优化数据库查询性能

## 代码变更
- 新增：120 行
- 删除：35 行
- 修改：56 行

## 进度趋势
- 今日完成：2 项
- 本周累计：8 项
- 本月累计：25 项

## 下一步计划
- 完成数据库查询优化
- 开始实现用户权限管理
```

**Chinese Weekly Report Example**:

```markdown
# 项目周报 - 2026-04-18 至 2026-04-24

## 完成任务
- ✅ 实现用户认证接口
- ✅ 修复登录页面 bug
- ✅ 设计数据库 schema
- ✅ 搭建项目基础架构
- ✅ 实现数据导出功能

## 进行中任务
- 🔄 优化数据库查询性能
- 🔄 实现用户权限管理

## 代码变更
- 新增：450 行
- 删除：120 行
- 修改：230 行

## 进度趋势
- 本周完成：5 项
- 本月累计：25 项
- 项目完成度：45%

## 问题与解决方案
- 数据库性能问题：通过添加索引和优化查询语句解决
- 前端样式问题：使用 CSS Grid 重构布局

## 下一步计划
- 完成数据库查询优化
- 实现用户权限管理
- 开始测试阶段
```

**English Daily Report Example**:

```markdown
# Project Daily Report - 2026-04-24

## Completed Tasks
- ✅ Implement user authentication API
- ✅ Fix login page bug

## In Progress Tasks
- 🔄 Optimize database query performance

## Code Changes
- Added: 120 lines
- Deleted: 35 lines
- Modified: 56 lines

## Progress Trend
- Today's completed: 2 items
- This week's total: 8 items
- This month's total: 25 items

## Next Steps
- Complete database query optimization
- Start implementing user permission management
```

**English Weekly Report Example**:

```markdown
# Project Weekly Report - 2026-04-18 to 2026-04-24

## Completed Tasks
- ✅ Implement user authentication API
- ✅ Fix login page bug
- ✅ Design database schema
- ✅ Set up project infrastructure
- ✅ Implement data export functionality

## In Progress Tasks
- 🔄 Optimize database query performance
- 🔄 Implement user permission management

## Code Changes
- Added: 450 lines
- Deleted: 120 lines
- Modified: 230 lines

## Progress Trend
- This week's completed: 5 items
- This month's total: 25 items
- Project completion: 45%

## Issues and Solutions
- Database performance issue: Resolved by adding indexes and optimizing query statements
- Frontend styling issue: Refactored layout using CSS Grid

## Next Steps
- Complete database query optimization
- Implement user permission management
- Start testing phase
```