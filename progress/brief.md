# /progress brief [daily|weekly] [--format=md|html|pdf]

## 指令

生成项目开发日报或周报，包含任务完成情况、代码变更统计和进度趋势分析。

## 执行流程

### Step 1: 数据收集

- 读取 `PROGRESS.md` 中的 `📅 任务历史` 部分
- 执行 `git log --since="1 day ago"` 或 `git log --since="1 week ago"` 获取提交历史
- 执行 `git diff --stat` 分析代码变更
- 解析 `✅ 最近完成` 和 `🎯 当前聚焦` 部分获取最新状态

### Step 2: 报告生成

根据指定的格式生成报告：

**Markdown 格式**：
- 包含任务完成情况
- 代码变更统计
- 进度趋势分析
- 下一步计划

**HTML 格式**：
- 包含 Markdown 内容
- 嵌入 Chart.js 生成代码活动和进度趋势图表
- 响应式设计，支持移动端查看

**PDF 格式**：
- 基于 HTML 格式生成
- 使用浏览器打印功能或 Pandoc 转换

### Step 3: 输出与导出

- 默认输出到终端
- 支持指定输出文件路径
- 自动创建输出目录（如果不存在）

## 命令选项

- `daily`：生成日报（默认）
- `weekly`：生成周报
- `--format=md`：输出 Markdown 格式（默认）
- `--format=html`：输出 HTML 格式
- `--format=pdf`：输出 PDF 格式
- `--output=<path>`：指定输出文件路径

## 示例

```
# 生成当日日报（Markdown 格式）
/progress brief daily

# 生成当周周报（HTML 格式）
/progress brief weekly --format=html

# 生成日报并导出到指定文件
/progress brief daily --format=pdf --output=./reports/daily.pdf

# 生成周报并导出到指定文件
/progress brief weekly --format=html --output=./reports/weekly.html
```

## 报告内容

**日报示例**：

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

**周报示例**：

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