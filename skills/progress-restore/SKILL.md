---
name: progress-restore
description: Use when resuming work on a new device, after a break, or after switching branches - restores session context from PROGRESS.md
version: 1.7.0
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
3. **Recently Completed**: Last completed phases
4. **Blockers & Issues**: Any problems encountered
5. **Context Notes**: Key decisions, design doc links
6. **Quick Recovery**: Commands and key files to open

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

📥 Next Phases
- [Extracted from PROGRESS.md]

🧱 Blockers & Issues
- [Extracted from PROGRESS.md, if any]

🧠 Context Notes
- Key decisions: ...
- Design docs: ...

⚡ Quick Recovery
- Commands: [Extracted from PROGRESS.md]
- Key files: [Extracted from PROGRESS.md open: section]

💡 Related Design Documents
- <file_path> (modified on <date>)
```

## Intelligent Section Detection

When reading PROGRESS.md, detect sections by common patterns:

| Content Type | Possible Section Names |
|--------------|------------------------|
| Current focus | `🎯`, `Current Focus`, `当前`, `Current` |
| Next steps | `📥`, `Next Phases`, `Todo`, `下一步`, `Next` |
| Paused tasks | `⏸️`, `Paused Tasks`, `暂停`, `Hold` |
| Completed | `✅`, `Completed`, `已完成`, `Recently Completed` |
| Blockers | `🧱`, `Blockers`, `Issues`, `问题` |
| Notes | `🧠`, `Context Notes`, `Notes`, `备注`, `Context` |
| Recovery | `⚡`, `Quick Recovery`, `恢复`, `Recovery` |

Projects may use any naming convention - adapt extraction logic to match existing sections.

## Standard PROGRESS.md Structure (Optional Reference)

```markdown
# Progress

> Last updated: {CURRENT_DATE}

## 🎯 Current Focus
<!-- Major task currently in progress -->

## 📥 Next Phases
<!-- Phases to do next -->

## ⏸️ Paused Tasks
<!-- Tasks paused mid-way -->

## ✅ Recently Completed Phases
<!-- Last 2-3 completed phases -->

## 🧱 Blockers & Issues
<!-- Problems encountered -->

## 🧠 Context Notes
<!-- Key decisions, design doc links -->

## ⚡ Quick Recovery
<!-- Commands and key files for restore -->
- `git pull`
- open: <!-- Key files to open -->

## 🏛️ Archive Links
<!-- Links to archived major tasks -->
```