---
name: progress-restore
description: Use when resuming work on a new device, after a break, or after switching branches - restores session context from PROGRESS.md
version: 1.6.0
---

# Progress Restore

Restore work session, sync remote progress, provide context recall, and start development environment.

## Global Rules

1. **Project Root Detection**: Search upward from the current directory until finding a directory containing `.git` or `PROGRESS.md` as the project root.
2. **File Path**: All operations target `PROGRESS.md` in the project root directory.
3. **Language Following User**: Analyze commit history and user input to automatically detect language and generate content in the corresponding language. Support for manually specifying language via `--lang` parameter.
   - **Detection Priority**: User input > Recent commit messages > System locale
   - **Supported Languages**: English (en) and Chinese (zh)
4. **If PROGRESS.md does not exist**:
   - Ask user: "Project progress tracking not detected. Would you like to initialize PROGRESS.md?"
   - After user confirmation, write the following standard structure directly in the project root directory:

5. **CRITICAL: Preserve Existing Format**: If PROGRESS.md already exists, preserve its structure. Only read and present content, never restructure.

## Execution Flow

### Step 1: Check Local Status and PROGRESS.md

- Execute `git status`.
- **If local is dirty (has uncommitted changes)**:
  - Output warning: "⚠️ Local has uncommitted changes, suggest first `/progress-checkpoint` to handle."
  - Abort process, let user decide.
- Check if `PROGRESS.md` exists:
  - If it doesn't exist: Prompt user to initialize it (as per Global Rules)
  - If it exists: Read and preserve its structure, proceed with content extraction

### Step 2: Check Remote Progress Updates

- Execute `git fetch origin`.
- Compare local and remote `PROGRESS.md`.
- **If remote has updates**:
  - Display remote PROGRESS.md diff summary.
  - Ask: "Remote progress has updates, execute `git pull --rebase`?"
  - Execute pull after user confirmation.
- **If remote has no updates**: Continue to next step.

### Step 3: Read Multi-dimensional Context

Read the following content to build recovery context:

1. **`PROGRESS.md`**: Extract current focus, next steps, and recovery info from existing sections (preserve format, don't assume standard sections).
2. **Recent commits**: `git log -3 --stat` (understand last exit position).
3. **WIP task detection**: Analyze WIP tags in recent commits, identify incomplete features.
4. **Latest design documents**:
   - **Whitelist directories**: Only search in `docs/`, `designs/`, `specs/`, `doc/`, `arch/` and project root directory.
   - **Blacklist exclusion**: Ignore `CHANGELOG.md`, `README.md`, `LICENSE.md`, etc.
   - **Time window**: `.md` files modified within 7 days, read their filenames and summaries.

### Step 4: Output Recovery Report and Start Environment

Structured output context, and interactively start environment:

```
🚀 **Work Session Recovery Report**

📍 **Last Exit Position**
- Recent commit: `<hash>` <message>
- Changed files: <list>

🎯 **Current Focus**
- [Extracted from PROGRESS.md current focus section]

📥 **Next Steps**
- [Extracted from PROGRESS.md next steps section]

💡 **Related Design Documents**
- <file_path> (modified on <date>)

🛠 **Prepare to Start Development Environment**
- Parse recovery commands from PROGRESS.md (look for recovery/start sections).
- If parsing fails or no command can be inferred, prompt user: "No startup command detected, please manually start the development environment."
- If there is an inferred command, ask: "Execute `[inferred/extracted command]`?"

🔄 **WIP Task Recovery**
- Identify and list all WIP status tasks
- Provide options for quick WIP environment recovery
```

## Standard PROGRESS.md Structure (Optional Reference)

This format is **optional reference only**. Projects can use any custom format:

```markdown
# Progress

> Last updated: {CURRENT_DATE}

## 🎯 Current Focus
<!-- Major task currently in progress, max 1-2 items -->

## 📥 Next Phases
<!-- Phases to do next -->

## ✅ Recently Completed Phases
<!-- Last 2-3 completed phases -->

## 🧱 Blockers & Issues
<!-- Problems encountered -->

## 🧠 Context Notes
<!-- Key decisions, design doc links, code statistics -->

## ⚡ Quick Recovery
<!-- Commands and key files for restore -->
- `git pull`
- open: <!-- Key files to open -->

## 🏛️ Archive Links
<!-- Links to archived major tasks -->
```

**Section Purpose:**

| Section | Purpose | Used By |
|---------|---------|---------|
| `🎯 Current Focus` | In-progress major task | restore, save, summary |
| `📥 Next Phases` | Planned phases | restore, save, summary |
| `✅ Recently Completed Phases` | Completed phases | save, archive |
| `🧱 Blockers & Issues` | Problems | restore |
| `🧠 Context Notes` | Decisions, design docs | restore |
| `⚡ Quick Recovery` | Restore commands | restore |
| `🏛️ Archive Links` | Archived major tasks | archive |

## Intelligent Section Detection

When reading PROGRESS.md, detect sections by common patterns:

| Content Type | Possible Section Names |
|--------------|------------------------|
| Current focus | `🎯`, `Current Focus`, `当前`, `Current` |
| Next steps | `📥`, `Todo`, `下一步`, `Next`, `🔜` |
| Recovery commands | `⚡`, `Quick Recovery`, `恢复`, `Recovery` |
| Completed | `✅`, `Completed`, `已完成`, `Recently Completed` |
| Issues | `🧱`, `Issues`, `问题`, `Blockers` |
| Notes | `🧠`, `Notes`, `备注`, `Context` |

Projects may use any naming convention - adapt extraction logic to match existing sections.