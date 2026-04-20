# agent-skills

通用 AI Agent skills 集合，支持 Claude Code、OpenCode 等主流 Agent。

## 安装

```bash
npx skills add wuruofan/agent-skills -g -y
```

## 收录的 Skills

### web-fetch-markdown

将网页 URL 转换为 Markdown，绕过 Agent 的域名安全检查。

**使用场景：**
- 代理服务无法直接访问的网站
- 需要获取外部网页内容时
- 遇到 "Unable to verify if domain" 错误时

**代理服务：**
1. `https://markdown.new/` - 首选，支持 Cloudflare 托管网站
2. `https://markdownforagents.com/r?url=` - 备选

**触发词：** "上网搜索"、"帮我查一下"、"fetch 网页"

---

更多 skill 持续添加中。
