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
# Install specific progress skill
npx skills add wuruofan/agent-skills --skill progress-show -g -y
npx skills add wuruofan/agent-skills --skill progress-checkpoint -g -y
npx skills add wuruofan/agent-skills --skill progress-restore -g -y
npx skills add wuruofan/agent-skills --skill progress-brief -g -y
npx skills add wuruofan/agent-skills --skill progress-archive -g -y
npx skills add wuruofan/agent-skills --skill progress-summary -g -y

# Install web-fetch-as-markdown skill
npx skills add wuruofan/agent-skills --skill web-fetch-as-markdown -g -y

# Install multiple skills
npx skills add wuruofan/agent-skills --skill progress-show --skill progress-checkpoint -g -y
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

Track project development progress, enable cross-device work session saving and recovery, automatically generate daily/weekly reports, manage history archiving, and provide current progress summaries.

Split into 6 independent skills for cross-platform compatibility:

| Skill | Description |
|-------|-------------|
| `progress-show` | Display project status overview - quick daily check |
| `progress-checkpoint` | Save progress, update PROGRESS.md, create Git commit |
| `progress-restore` | Restore work session, sync remote progress |
| `progress-brief` | Generate daily/weekly reports with code statistics |
| `progress-archive` | Archive history records older than 7 days |
| `progress-summary` | Generate detailed current progress summary |

**Use Cases:**
- Track project development progress
- Save and recover work sessions across devices
- Generate daily/weekly reports
- Archive historical records
- Get current progress summaries

**Trigger Words:** "track progress", "save session", "generate report", "archive history", "check status"

---

More skills coming soon.