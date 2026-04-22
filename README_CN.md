# agent-skills

通用 AI Agent skills 集合，支持 Claude Code、OpenCode 等主流 Agent。

## 安装

```bash
npx skills add wuruofan/agent-skills -g -y
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

更多 skill 持续添加中。
