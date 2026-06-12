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

### Situation-based triggers
- Before commit/stash/PR: Call `/progress-save` to update PROGRESS.md
- When resuming work after a break: Call `/progress-restore` to restore session context
- When a major task or all its phases are finished: Call `/progress-archive` to archive history
- When starting a new session to continue previous work: Call `/progress-summary` to get session context
- Before `git merge` / `git rebase` / `git cherry-pick` involving PROGRESS.md (or detected PROGRESS.md conflict markers): Call `/progress-merge`
- After resolving any git operation that left PROGRESS.md in conflict state: Call `/progress-merge` before continuing

### Sequencing during `git merge` operations
When AI executes `git merge` / `git rebase` / `git cherry-pick` and conflicts arise:
1. FIRST resolve non-PROGRESS.md conflicts in the working tree
2. Verify tests pass or merge correctness is confirmed
3. THEN call /progress-merge to handle PROGRESS.md conflict
4. Finally `git add` + complete the merge

### User-phrase triggers (defense-in-depth)
- User says "update/save/record PROGRESS.md", "更新进度", "保存进度", "记录进度" → `/progress-save`
- User says "restore/resume/load context", "恢复进度", "接着干", "继续上次", "之前干到哪了" → `/progress-restore`
- User says "archive/clean up/trim/cleanup", "归档", "归档进度", "任务完成", "进度太长", "清理", "精简", "太长了", "堆积", "收尾" → `/progress-archive`
- User says "summarize/recap/handoff", "总结一下", "会话总结", "交接", "写个摘要" → `/progress-summary`
- User says "merge progress", "合并进度", "PROGRESS 冲突", "两个分支的 PROGRESS 怎么合" → `/progress-merge`
```

This lets LLM autonomously trigger skills at appropriate times.

**Note:** `/progress-save` will automatically detect completed major tasks and prompt archive suggestion when appropriate. It will also detect PROGRESS.md merge conflicts and redirect to `/progress-merge`.

---

More skills coming soon.