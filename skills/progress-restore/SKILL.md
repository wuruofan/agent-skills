---
name: progress-restore
description: Use when resuming work after a break, on a new device, in a new session, after switching branches, or when user asks to read, load, or restore context (恢复进度, 接着干, 继续上次, 加载进度, 读取进度, 之前干到哪了)
version: 1.9.1
---

# Progress Restore

Restore work session context from PROGRESS.md. Read progress state, design documents, and output recovery report. Purely read operation - no git operations.

## Global Rules

1. **Project Root Detection**: Search upward from current directory until finding `.git` or `PROGRESS.md`.
2. **File Path**: All operations target `PROGRESS.md` in project root.
3. **Language Following User**: Analyze commit history and user input to auto-detect language.
4. **If PROGRESS.md does not exist**: Ask user to initialize it.
5. **CRITICAL: Preserve Existing Format**: If PROGRESS.md already exists, preserve its structure. Only read content, never restructure.

## Execution Flow

### Step 1: Read PROGRESS.md

- Check if `PROGRESS.md` exists:
  - If not: Prompt user to initialize it
  - If exists: Read and extract content from existing sections

### Step 2: Extract Context from PROGRESS.md

Read the following sections (detect section names intelligently):

1. **Current Focus**: What's being worked on now
2. **Next Phases**: Planned next steps
3. **Paused Tasks**: Mid-way tasks with blockers and entry points
4. **Recently Completed**: Last completed phases
5. **Blockers & Issues**: Any problems encountered
6. **Context Notes**: Key decisions, design doc links
7. **Quick Recovery**: Commands and key files to open
8. **Archive Links**: Links to archived major tasks

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

## Intelligent Section Detection

When reading PROGRESS.md, detect sections by common patterns:

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

Projects may use any naming convention - adapt extraction logic to match existing sections.

## Large File Reading (Anti-Thrashing Policy)

**Problem**: Reading a long PROGRESS.md in a single Read call fills the context window and triggers autocompact thrashing ("Autocompact is thrashing: context refilled to limit within 3 turns").

**Trigger**: PROGRESS.md > 300 lines.

**Policy — read in segments, not whole-file**:
1. **First pass**: `Read(offset: 1, limit: 50)` — get frontmatter + section headings.
2. **Second pass**: For each section listed in Step 2, `Read(offset: <section_start>, limit: <section_length>)` — only the lines you need to extract.
3. **Skip irrelevant sections**: If user only asks "what was I working on?", read only Current Focus + Context Notes + Quick Recovery — skip Archive Links, Recently Completed, etc.
4. **Hard rule**: Never call `Read` on a > 300-line PROGRESS.md without `offset`/`limit`. Use the structural read to plan targeted reads.
