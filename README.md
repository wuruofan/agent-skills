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
# Install only progress skill
npx skills add wuruofan/agent-skills --skill progress -g -y

# Install only web-fetch-as-markdown skill
npx skills add wuruofan/agent-skills --skill web-fetch-as-markdown -g -y

# Install multiple skills
npx skills add wuruofan/agent-skills --skill progress --skill web-fetch-as-markdown -g -y
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

### progress

Track project development progress, enable cross-device work session saving and recovery, automatically generate daily/weekly reports, manage history archiving, and provide current progress summaries.

**Use Cases:**
- Track project development progress
- Save and recover work sessions across devices
- Generate daily/weekly reports
- Archive historical records
- Get current progress summaries

**Commands:**
- `show` (abbreviation: `s`) - Display project status overview
- `checkpoint` (abbreviation: `c`) - Save progress and create Git commit
- `restore` (abbreviation: `r`) - Recover work session
- `brief` (abbreviation: `b`) - Generate daily/weekly reports
- `archive` (abbreviation: `a`) - Archive historical records
- `summary` (abbreviation: `sum`) - Generate current progress summary

**Trigger Words:** "track progress", "save session", "generate report", "archive history", "check status"

---

More skills coming soon.
