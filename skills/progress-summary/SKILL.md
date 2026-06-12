---
name: progress-summary
description: Use when starting a new session, handing off to another session, or when context is getting long and user asks for a summary, recap, or handoff (生成总结, 会话总结, 总结一下, 上下文回顾, 接着上次干, 交接, 写个摘要)
version: 1.8.0
---

# Progress Summary

Generate a compact summary combining project state from PROGRESS.md and current chat session context. Enables seamless new session continuity by capturing essential context without unnecessary verbosity.

## Global Rules

1. **Project Root Detection**: Search upward from the current directory until finding a directory containing `.git` or `PROGRESS.md` as the project root.
2. **File Path**: All operations target `PROGRESS.md` in the project root directory.
3. **Language Following User**: Analyze commit history and user input to auto-detect language.
   - **Detection Priority**: User input > Recent commit messages > System locale
   - **Supported Languages**: English (en) and Chinese (zh)
4. **If PROGRESS.md does not exist**: Ask user to initialize it.
5. **CRITICAL: Preserve Existing Format**: If PROGRESS.md already exists, preserve its structure. Only read content, never restructure.

## Execution Flow

### Step 1: Collect Context

1. **Read PROGRESS.md**: Extract current focus, next steps, completed tasks (detect section names intelligently).

2. **Analyze Git Status**:
   - Execute `git status --short` to get uncommitted changes
   - Execute `git diff --stat` to analyze code changes
   - Execute `git log -5 --oneline` to get recent commits

3. **Capture Session Context**:
   - Key decisions made
   - Errors encountered and resolutions
   - Files modified and their purpose
   - Keep concise (≤ 800 tokens, prioritize decisions and errors over verbose descriptions)

### Step 2: Intelligent Section Detection

Detect sections by common patterns:

| Content Type | Possible Section Names |
|--------------|------------------------|
| Last updated time | `> Last updated`, `> 最后更新` |
| Current focus | `🎯`, `Current Focus`, `当前`, `Current` |
| Next steps | `📥`, `Next Phases`, `Todo`, `下一步`, `Next` |
| Paused tasks | `⏸️`, `Paused Tasks`, `暂停`, `Hold` |
| Completed | `✅`, `Completed`, `已完成`, `Recently Completed` |
| Blockers | `🧱`, `Blockers`, `Issues`, `问题` |
| Notes | `🧠`, `Context Notes`, `Notes`, `备注`, `Context` |
| Recovery | `⚡`, `Quick Recovery`, `恢复`, `Recovery` |
| Archive Links | `🏛️`, `Archive Links`, `归档`, `Archive` |

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

## Examples

```
# Generate summary
/progress-summary

# Use in new session
Copy output and paste at start of new session
```

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