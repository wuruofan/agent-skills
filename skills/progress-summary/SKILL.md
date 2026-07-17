---
name: progress-summary
description: Use when starting a new session, handing off to another session, or when context is getting long and user asks for a summary, recap, or handoff (生成总结, 会话总结, 总结一下, 上下文回顾, 接着上次干, 交接, 写个摘要)
version: 1.9.0
---

# Progress Summary

Generate a compact summary combining project state from PROGRESS.md and current chat session context. Enables seamless new session continuity by capturing essential context without unnecessary verbosity.

## Global Rules

Find project root (upward to `.git`/`PROGRESS.md`); target `PROGRESS.md` in root; language follows user input > commit history > locale (en/zh); if file missing, ask user to init; preserve existing format, never restructure.

## Execution Flow

### Step 1: Collect Context

1. **Read PROGRESS.md**: Extract all sections — current focus, next steps, completed tasks, paused tasks, blockers, context notes, archive links. Detect section names intelligently.

2. **Analyze Git Status**:
   - `git status --short` — uncommitted changes
   - `git diff --stat` — code changes
   - `git log -5 --oneline` — recent commits

3. **Capture Session Context**:
   - Key decisions made
   - Errors encountered and resolutions
   - Files modified and their purpose
   - Keep concise (≤ 800 tokens, prioritize decisions and errors over verbose descriptions)

### Step 2: Intelligent Section Detection

Match sections using the unified Section Detection table below. **Three-layer identification:** (1) emoji/alias match, (2) content semantic match — if no name match, judge by the "Belongs" contract column, (3) unresolvable → skip or ask user. Never rename existing sections during extraction.

### Section Detection & Contract Table

| Section | Emoji / aliases | Belongs (Layer 2 semantic contract) |
|---------|-----------------|--------------------------------------|
| `> Last updated` | `> Last updated` / `> 最后更新` | Timestamp header |
| 🎯 Current Focus | 🎯 / Current / 当前 | In-progress tasks + 1-2 sentence status |
| 📥 Next Phases | 📥 / Next Phases / Todo / 下一步 | Truly pending phases/tasks with spec/plan links |
| ⏸️ Paused Tasks | ⏸️ / Paused / 暂停 | Task name + 1-sentence pause reason + entry point |
| ✅ Recently Completed | ✅ / Completed / 已完成 / Recently Completed | Task name + date + 1 sentence + archive link |
| 🧱 Blockers & Issues | 🧱 / Blockers / 问题 | Active blockers + status |
| 🧠 Context Notes | 🧠 / Notes / 备注 | 1-3 sentence key findings/decisions/rationale |
| ⚡ Quick Recovery | ⚡ / Recovery / 恢复 | ≤5 core commands with necessary comments |
| 🏛️ Archive Links | 🏛️ / Archive / 归档 | Links to archived task files |
| 🔍 Unverified | 🔍 / Unverified / 待手测 / 待验证 / Manual Test Pending | Items in code-shipped + tests-pass + manual-test-pending gray zone |

### Step 3: Generate Summary

```
📊 Session Summary
=================

🎯 Current Focus:
- [Extracted from PROGRESS.md]

📋 Session Context:
- User Intent: ...
- Key Decisions: ...
- Errors Fixed: ...
- Files Modified: ...

📥 Next Steps:
- [Extracted from PROGRESS.md]

💻 Uncommitted: X files | +Y/-Z lines
```

Format output for easy copy-paste into new sessions.

## Large File Reading (Anti-Thrashing Policy)

**Problem**: Reading a long PROGRESS.md in a single Read call fills the context window and triggers autocompact thrashing ("Autocompact is thrashing: context refilled to limit within 3 turns").

**Trigger**: PROGRESS.md > 300 lines.

**Policy — read in segments, not whole-file**:
1. **First pass**: `Read(offset: 1, limit: 50)` — get frontmatter + section headings.
2. **Second pass**: For sections needed in the summary (Current Focus / Next Phases / Recently Completed / Blockers), `Read(offset: <section_start>, limit: <section_length>)` — only those lines.
3. **Skip secondary sections**: Archive Links and Quick Recovery can be skipped or read with a small limit — they rarely affect summary quality.
4. **Hard rule**: Never call `Read` on a > 300-line PROGRESS.md without `offset`/`limit`.

## Output Example

```
📊 Session Summary
=================

🎯 Current Focus:
- Implement user authentication API (WIP)

📋 Session Context:
- User Intent: Build secure authentication system
- Key Decisions: JWT tokens, Redis session storage
- Errors Fixed: JWT signing algorithm mismatch
- Files Modified: src/api/auth.js, src/utils/db.js

📥 Next Steps:
1. Complete authentication API
2. Add rate limiting middleware

💻 Uncommitted: 2 files | +37/-7 lines
```
