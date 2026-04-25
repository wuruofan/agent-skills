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
# 安装特定 progress 技能
npx skills add wuruofan/agent-skills --skill progress-show -g -y
npx skills add wuruofan/agent-skills --skill progress-checkpoint -g -y
npx skills add wuruofan/agent-skills --skill progress-restore -g -y
npx skills add wuruofan/agent-skills --skill progress-brief -g -y
npx skills add wuruofan/agent-skills --skill progress-archive -g -y
npx skills add wuruofan/agent-skills --skill progress-summary -g -y

# 安装 web-fetch-as-markdown 技能
npx skills add wuruofan/agent-skills --skill web-fetch-as-markdown -g -y

# 安装多个技能
npx skills add wuruofan/agent-skills --skill progress-show --skill progress-checkpoint -g -y
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

### Progress 技能套件

跟踪项目开发进度，支持跨设备工作会话保存和恢复，自动生成日报/周报，管理历史归档，并提供当前进度摘要。

拆分为 6 个独立技能，以实现跨平台兼容：

| 技能 | 描述 |
|------|------|
| `progress-show` | 显示项目状态概览 - 快速日常检查 |
| `progress-checkpoint` | 保存进度，更新 PROGRESS.md，创建 Git 提交 |
| `progress-restore` | 恢复工作会话，同步远程进度 |
| `progress-brief` | 生成日报/周报，包含代码统计 |
| `progress-archive` | 归档 7 天前的历史记录 |
| `progress-summary` | 生成详细的当前进度摘要 |

**使用场景：**
- 跟踪项目开发进度
- 跨设备保存和恢复工作会话
- 生成日报/周报
- 归档历史记录
- 获取当前进度摘要

**触发词：** "跟踪进度"、"保存会话"、"生成报告"、"归档历史"、"检查状态"

---

更多 skill 持续添加中。