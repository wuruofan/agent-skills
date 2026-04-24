# agent-skills

通用 AI Agent skills 集合，支持 Claude Code、OpenCode 等主流 Agent。

[English Version](README.md)

## 安装

### 安装所有技能

```bash
npx skills add wuruofan/agent-skills -g -y
```

### 安装特定技能

```bash
# 仅安装 progress 技能
npx skills add wuruofan/agent-skills --skill progress -g -y

# 仅安装 web-fetch-as-markdown 技能
npx skills add wuruofan/agent-skills --skill web-fetch-as-markdown -g -y

# 安装多个技能
npx skills add wuruofan/agent-skills --skill progress --skill web-fetch-as-markdown -g -y
```

## 收录的 Skills

### web-fetch-as-markdown

将网页 URL 转换为干净的 Markdown，剥离广告、导航栏和干扰内容。

**使用场景：**
- 绕过 Agent 的域名安全检查
- 将网页转换为易解析的干净 markdown
- 遇到 "Unable to verify if domain" 错误时

**转换服务：**
1. `https://markdown.new/` - 首选（Cloudflare，优化 token 消耗）
2. `https://r.jina.ai/<url>` - 备用 1（Jina AI Reader API）
3. `https://markdownforagents.com/r?url=<url>` - 备用 2（需用户授权）

**触发词：** "上网搜索"、"帮我查一下"、"fetch 网页"、"帮我解析"

---

### progress

跟踪项目开发进度，支持跨设备工作会话保存和恢复，自动生成日报/周报，管理历史归档，并提供当前进度摘要。

**使用场景：**
- 跟踪项目开发进度
- 跨设备保存和恢复工作会话
- 生成日报/周报
- 归档历史记录
- 获取当前进度摘要

**命令：**
- `show` (缩写: `s`) - 显示项目状态概览
- `checkpoint` (缩写: `c`) - 保存进度并创建 Git 提交
- `restore` (缩写: `r`) - 恢复工作会话
- `brief` (缩写: `b`) - 生成日报/周报
- `archive` (缩写: `a`) - 归档历史记录
- `summary` (缩写: `sum`) - 生成当前进度摘要

**触发词：** "跟踪进度"、"保存会话"、"生成报告"、"归档历史"、"检查状态"

---

更多 skill 持续添加中。
