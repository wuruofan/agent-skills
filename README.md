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
npx skills add wuruofan/agent-skills --skill progress-merge -g -y

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

5 core skills:

| Skill | Purpose | When to Use |
|-------|---------|-------------|
| `progress-save` | Update PROGRESS.md | Before commit, stash, or switching branches |
| `progress-restore` | Restore session context | After break, on new device, or after branch switch |
| `progress-archive` | Archive history records | Major task completed |
| `progress-summary` | Generate session summary | Start new session to continue previous work |
| `progress-merge` | Merge PROGRESS.md across branches | After `git merge`/`git rebase`/`git cherry-pick` with PROGRESS.md conflicts |

**Quick Decision Guide:**
- About to commit? → `/progress-save`
- Coming back to work? → `/progress-restore`
- Major task done? → `/progress-archive`
- Continuing in new session? → `/progress-summary`
- Merging branches with PROGRESS.md? → `/progress-merge`

**Recommended AGENTS.md Configuration:**

> **Critical**: This section must be installed in your project's AGENTS.md / CLAUDE.md / GEMINI.md to make AI sequencing constraints effective. Without it, `/progress-merge` falls back to weaker triggers.

```markdown
## Progress Tracking (Critical)

Triggers (call the matching skill):
- Before commit/stash/PR/push → `/progress-save`
- Resuming work, new device, branch switch → `/progress-restore`
- Major task complete, PROGRESS.md too long → `/progress-archive`
- Unverified/待手测 table bloat (✅ items >3 or total >10) → `/progress-archive` (Mode C verify-cleanup)
- New session continuing previous work → `/progress-summary`
- `git merge` / `rebase` / `cherry-pick` touching PROGRESS.md (or conflict markers detected) → `/progress-merge`
- After any git op that left PROGRESS.md in conflict state → `/progress-merge`

### Git merge / rebase / cherry-pick sequencing
When conflicts arise:
1. Resolve non-PROGRESS.md conflicts first
2. Verify (tests pass / merge correctness)
3. Call `/progress-merge` for the PROGRESS.md conflict
4. `git add` + complete the merge

### Reading project state files (anti-thrashing)
When reading PROGRESS.md, CLAUDE.md, AGENTS.md, or any project doc > 300 lines:
- First read only the first 50 lines (frontmatter + section headings) using Read with `offset`/`limit`
- Then read only the specific sections you need by line range
- Never call Read on a > 300-line file without `offset`/`limit` — this is the #1 cause of autocompact thrashing
```

This lets LLM autonomously trigger skills at appropriate times.

**Note:** `/progress-save` will automatically detect completed major tasks and prompt archive suggestion when appropriate. It will also detect PROGRESS.md merge conflicts and redirect to `/progress-merge`, and detect Unverified-table bloat to suggest `/progress-archive` Mode C.

---

More skills coming soon.