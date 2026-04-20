# agent-skills

A collection of universal AI Agent skills, compatible with Claude Code, OpenCode, and other major Agent frameworks.

## Installation

```bash
npx skills add wuruofan/agent-skills -g -y
```

## Included Skills

### web-fetch-markdown

Converts web URLs to Markdown, bypassing domain security checks in Agents.

**Use Cases:**
- Websites that proxy services cannot directly access
- When you need to fetch external webpage content
- When encountering "Unable to verify if domain" errors

**Proxy Services:**
1. `https://markdown.new/` - Preferred, supports Cloudflare-hosted sites
2. `https://markdownforagents.com/r?url=` - Alternative

**Trigger Words:** "fetch webpage", "look up", "search online"

---

More skills coming soon.
