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
# Install progressing skill
npx skills add wuruofan/agent-skills --skill progressing -g -y

# Install web-fetch-as-markdown skill
npx skills add wuruofan/agent-skills --skill web-fetch-as-markdown -g -y
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

### Progressing

A todo with three superpowers a plain todo doesn't have — git auto-reconciliation, explicit closure conditions, and stale-entry review on read. Single `PROGRESS.md` in the project root; git already syncs it across devices, no external archive to maintain.

**What it does:**

| Context | Action |
|---|---|
| New session / switched machine / "接着干 / 继续 / 之前干到哪了" | Load — read PROGRESS.md, reconcile against git, surface stale entries, output recovery report |
| Mid-session, an entry's `Done when:` was just satisfied | Close — delete the entry in-place, refresh timestamp |
| Session interrupted mid-way | Save — write unfinished state. Force with `/progressing save` |

**When NOT to use:** routine commits with no unfinished work — if nothing is mid-way, there is nothing to persist.

**PROGRESS.md format:**

```markdown
# Progress

> Last updated: 2026-09-09

## Open
- [2026-09-09] compact live/resume display — L4 run interrupted. Done when: `bun run test:l4` exits 0

## Verify
- [2026-09-04] /resume rendering after compact — commit `f00dbabe`. Awaiting: user TTY check

## Paused
- Tool/Thinking display — blocked until v0.7.1. Restart: docs/specs/tool-display.md
```

Limits: entry ≤ 2 lines; `## Paused` ≤ 10; file ≤ 60 lines; omit empty sections. Default `STALE_DAYS=7`.

**Recommended AGENTS.md Configuration:**

> **Critical**: This section must be installed in your project's AGENTS.md / CLAUDE.md / GEMINI.md to make AI sequencing constraints effective.

```markdown
## Progress Tracking (Critical)

Single PROGRESS.md in the project root. Not a per-commit ritual. Pick the right action by context:
- New session / returning to a project / "接着干 / 继续 / 之前干到哪了" / switched machine → `/progressing` (load)
- Mid-session, an entry's `Done when:` was just satisfied → `/progressing` (close)
- Session interrupted mid-way (network drop, switching machines, explicit handoff, pre-compaction) → `/progressing` (save) or `/progressing save`

### Reading project state files (anti-thrashing)
When reading CLAUDE.md, AGENTS.md, or any project doc > 300 lines:
- First read only the first 50 lines (frontmatter + section headings) using Read with `offset`/`limit`
- Then read only the specific sections you need by line range
- Never call Read on a > 300-line file without `offset`/`limit` — this is the #1 cause of autocompact thrashing
```

This lets LLM autonomously trigger the skill at appropriate times. PROGRESS.md stays small by design (≤ 60 lines) and is read in full.
