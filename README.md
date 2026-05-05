# agent-skills

Universal AI Agent skills collection, compatible with Claude Code, OpenCode, and other major Agent frameworks.

[中文版本 (Chinese Version)](README_CN.md)

## Installation

### Install all skills

```bash
npx skills add wuruofan/agent-skills -g -y
```

### Install specific skills

```bash
# Install progress skills
npx skills add wuruofan/agent-skills --skill progress-save -g -y
npx skills add wuruofan/agent-skills --skill progress-restore -g -y
npx skills add wuruofan/agent-skills --skill progress-archive -g -y
npx skills add wuruofan/agent-skills --skill progress-summary -g -y

# Install web-fetch-as-markdown skill
npx skills add wuruofan/agent-skills --skill web-fetch-as-markdown -g -y

# Install multiple skills
npx skills add wuruofan/agent-skills --skill progress-save --skill progress-restore -g -y
```

## Included Skills

### web-fetch-as-markdown

Fetches any web URL and converts it to clean, structured Markdown — stripping ads, navigation, and clutter to leave only readable content.

**Use Cases:**
- Bypass domain safety restrictions in Agents
- Convert webpages to clean markdown for easier parsing
- When encountering "Unable to verify if domain" errors

**Conversion Services:**
1. `https://markdown.new/` - Primary (Cloudflare, optimized for token reduction)
2. `https://r.jina.ai/<url>` - Fallback 1 (Jina AI Reader API)
3. `https://markdownforagents.com/r?url=<url>` - Fallback 2 (requires user consent)

**Trigger Words:** "fetch webpage", "look up", "search online", "parse data from"

---

### Progress Skills Suite

Track project development progress, save and restore work sessions across devices, archive development history.

**第一步：配置 AGENTS.md（重要）**

在项目的 AGENTS.md 添加以下规则，让 LLM 自动触发技能：

```markdown
## 开发进度追踪

提交代码前：调用 `/progress-save` 更新 PROGRESS.md
恢复工作时：调用 `/progress-restore` 恢复会话上下文
大任务完成时：调用 `/progress-archive` 归档历史记录
```

配置后，LLM 会在合适时机自主调用技能，无需手动记忆。

**4 个核心技能：**

| Skill | Purpose | When to Use |
|-------|---------|-------------|
| `progress-save` | Update PROGRESS.md | Before commit, stash, or switching branches |
| `progress-restore` | Restore session context | After break, on new device, or after branch switch |
| `progress-archive` | Archive history records | Major task completed |
| `progress-summary` | Generate session summary | Start new session to continue previous work |

**Quick Decision Guide:**
- About to commit? → `/progress-save`
- Coming back to work? → `/progress-restore`
- Major task done? → `/progress-archive`
- Continuing in new session? → `/progress-summary`

**自动化提示：** `/progress-save` 会检测已完成的大任务，自动提示归档建议。

---

More skills coming soon.