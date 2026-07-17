---
name: progress-restore
description: Use when resuming work after a break, on a new device, in a new session, after switching branches, or when user asks to read, load, or restore context (恢复进度, 接着干, 继续上次, 加载进度, 读取进度, 之前干到哪了)
version: 1.10.0
---

# Progress Restore

Restore work session context from PROGRESS.md. Read progress state, design documents, and output recovery report. Purely read operation - no git operations.

## Global Rules

Find project root (upward to `.git`/`PROGRESS.md`); target `PROGRESS.md` in root; language follows user input > commit history > locale (en/zh); if file missing, ask user to init; preserve existing format, never restructure.

## Execution Flow

### Step 1: Read PROGRESS.md

- If not exists: prompt user to initialize
- If exists: read and extract content from existing sections

### Step 2: Extract Context from PROGRESS.md

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

Extract: Current Focus, Next Phases, Paused Tasks, Recently Completed, Blockers & Issues, Context Notes, Quick Recovery, Archive Links.

### Step 3: Collect Additional Context

1. **Recent commits**: `git log -3 --stat` (understand last work position)
2. **Latest design documents**:
   - Search in `docs/`, `designs/`, `specs/`, `doc/`, `arch/`
   - Ignore `CHANGELOG.md`, `README.md`, `LICENSE.md`
   - `.md` files modified within 7 days

### Step 4: Output Recovery Report

```
🚀 Work Session Recovery Report

📍 Last Work Position
- Recent commit: <hash> <message>
- Changed files: <list>

🎯 Current Focus
- [Extracted from PROGRESS.md]

⏸️ Paused Tasks
- [Extracted from PROGRESS.md, if any]

📥 Next Phases
- [Extracted from PROGRESS.md]

✅ Recently Completed
- [Extracted from PROGRESS.md, if any]

🧱 Blockers & Issues
- [Extracted from PROGRESS.md, if any]

🧠 Context Notes
- Key decisions: ...
- Design docs: ...

⚡ Quick Recovery
- Commands: [Extracted from PROGRESS.md]
- Key files: [Extracted from PROGRESS.md open: section]

🏛️ Archive Links
- [Extracted from PROGRESS.md, if any]

💡 Related Design Documents
- <file_path> (modified on <date>)
```

Projects may use any naming convention — adapt extraction logic to match existing sections.

## Large File Reading (Anti-Thrashing Policy)

**Problem**: Reading a long PROGRESS.md in a single Read call fills the context window and triggers autocompact thrashing ("Autocompact is thrashing: context refilled to limit within 3 turns").

**Trigger**: PROGRESS.md > 300 lines.

**Policy — read in segments, not whole-file**:
1. **First pass**: `Read(offset: 1, limit: 50)` — get frontmatter + section headings.
2. **Second pass**: For each section listed in Step 2, `Read(offset: <section_start>, limit: <section_length>)` — only the lines you need to extract.
3. **Skip irrelevant sections**: If user only asks "what was I working on?", read only Current Focus + Context Notes + Quick Recovery — skip Archive Links, Recently Completed, etc.
4. **Hard rule**: Never call `Read` on a > 300-line PROGRESS.md without `offset`/`limit`. Use the structural read to plan targeted reads.
