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
# Progress · managed by `progressing` skill

> Last updated: 2026-10-20

> 一条目一行；详情在各自 spec/plan/debug 文件（见 `Detail:`）。手动编辑会破坏格式契约 —— 改完请跑格式闸门。

## Open
- [2026-10-20] eager terminal session — 落地完成，待真机验证. Done when: `bun run test:gateway` 全绿. Detail: docs/specs/2026-10-13-eager-terminal-session.md

## Verify
- [2026-10-19] ink CR 列漂移修复 — 提交 `04139ea0`. Awaiting: user TTY check

## Paused
- [2026-09-01] Tool/Thinking display — 阻塞于 v0.7.1. Restart: docs/specs/tool-display.md
```

One entry = one line. The header is a bare date. Exactly three sections. Every entry ends with `Done when:` / `Awaiting:` / `Restart:`; `Detail: <path>` points at the file holding the detail.

The H1 carries the managed-by marker and a short `>` line carries the format note — both render, so a human opening the file sees who owns it. The gate allows a `>` line only when it is the date header or a short (≤ 200 chars) managed-by / format note, so the header cannot drift back into a changelog.

**Put the install command in the project's AGENTS.md, not in the ledger.** An agent without this skill never opens `PROGRESS.md`, so a command there reaches nobody; `AGENTS.md` is read by every agent on every session.

Limits: entry ≤ 256 chars; `## Open` ≤ 15, `## Verify` ≤ 10, `## Paused` ≤ 10; file ≤ 60 lines and ≤ 12,000 chars. Default `STALE_DAYS=7`.

Line count alone is not a real constraint — a single line can carry 20,000 chars, and a bloated ledger is re-read in full on every session. Verify the format mechanically after **every** write (Load, Close, Save, and ad-hoc edits); the command is embedded in the skill and must exit 0 before a write stands:

```bash
# run the gate block embedded in skills/progressing/SKILL.md
```

A failed gate means: move the detail into the file it describes and link it with `Detail:`; close, confirm, or archive entries that pushed a section over cap.

**Recommended AGENTS.md Configuration:**

> **Critical**: This section must be installed in your project's AGENTS.md / CLAUDE.md / GEMINI.md to make AI sequencing constraints effective.

```markdown
## Progress Tracking (Critical)

Single PROGRESS.md in the project root. Not a per-commit ritual. Pick the right action by context:
- New session / returning to a project / "接着干 / 继续 / 之前干到哪了" / switched machine → `/progressing` (load)
- Mid-session, an entry's `Done when:` was just satisfied → `/progressing` (close)
- Session interrupted mid-way (network drop, switching machines, explicit handoff, pre-compaction) → `/progressing` (save) or `/progressing save`

> **New machine / fresh environment**: this repo's `PROGRESS.md` is managed by the `progressing` skill. If it is not installed:
> ```bash
> npx skills add wuruofan/agent-skills --skill progressing -g -y
> ```
> Without it the entries are still readable (plain markdown) — you only lose auto-reconciliation and closure checks.

### PROGRESS.md stays small (this is why it is read in full)
PROGRESS.md is re-read completely on every load, so its size is paid for every session.
One entry = one line; the header is a bare date; detail lives in a spec/plan/debug file
linked with `Detail: <path>`. Run the format gate after every write to PROGRESS.md —
not only on load and save. A gate that runs "usually" does not run.

### Reading project state files (anti-thrashing)
When reading any project doc that may be large (CLAUDE.md, AGENTS.md, specs, plans, handoffs):
- If the file is > 300 lines, never call Read without `offset`/`limit`
  — read first 50 lines (frontmatter + section headings), then targeted
  section reads by line range
- For multi-file setups (multiple AGENTS.md / sub-folder CLAUDE.md):
  budget the total per-call Read payload, not the per-file size —
  a batch of medium files is worse than one large one for autocompact thrashing
```

This lets LLM autonomously trigger the skill at appropriate times. PROGRESS.md stays small by design (≤ 60 lines) and is read in full.
