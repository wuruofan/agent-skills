---
name: progress-restore
description: Restore work session, sync remote progress, provide context recall - resume work on a new device or after break
version: 1.5.0
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

5. **If PROGRESS.md already exists**:
   - Check if it's using the old format (not matching the standard structure above)
   - If old format detected:
     - Create a backup: `PROGRESS.md.bak.<timestamp>`
     - Convert old content to the new structure where possible
     - Preserve existing task information
     - Write the updated content to PROGRESS.md
     - Inform user: "PROGRESS.md has been upgraded to the new format. A backup has been created at PROGRESS.md.bak.<timestamp>."
   - If already using the new format:
     - Proceed normally
     - No backup needed

## Execution Flow

### Step 1: Check Local Status and PROGRESS.md

- Execute `git status`.
- **If local is dirty (has uncommitted changes)**:
  - Output warning: "⚠️ Local has uncommitted changes, suggest first `/progress-checkpoint` to handle."
  - Abort process, let user decide.
- Check if `PROGRESS.md` exists:
  - If it doesn't exist: Prompt user to initialize it (as per Global Rules)
  - If it exists and old format detected:
    - Create a backup: `PROGRESS.md.bak.<timestamp>`
    - Convert old content to new structure
    - Inform user: "PROGRESS.md has been upgraded to the new format. A backup has been created at PROGRESS.md.bak.<timestamp>."
  - If already using new format: Proceed normally

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

1. **`PROGRESS.md`**: Focus on 🎯, 📥, ⚡ sections, especially identify WIP status tasks.
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
- [Extracted from 🎯, specially marked WIP tasks]

📥 **Next Steps**
- [Extracted from 📥]

💡 **Related Design Documents**
- <file_path> (modified on <date>)

🛠 **Prepare to Start Development Environment**
- Parse commands in `⚡ Quick Recovery`, infer if none (based on Makefile/package.json, etc.).
- If parsing fails or no command can be inferred, prompt user: "No startup command detected, please manually start the development environment or update the ⚡ Quick Recovery section in PROGRESS.md."
- If there is an inferred command, ask: "Execute `[inferred/extracted command]`?"
- **Auto-writeback**: If user corrects the startup command, automatically update it to the `⚡ Quick Recovery` section of `PROGRESS.md` for permanent memory.

🔄 **WIP Task Recovery**
- Identify and list all WIP status tasks
- Provide options for quick WIP environment recovery
```

## Standard PROGRESS.md Structure

```
# Progress

> Last updated: {CURRENT_DATE}

## 🎯 Current Focus
<!-- Core tasks in progress, recommended no more than 2 -->

## 📥 Todo Queue
<!-- Next planned tasks -->

## ✅ Recently Completed
<!-- Keep only the last 3-5 items to avoid infinite file growth -->

## 🧱 Blockers & Issues
<!-- Record sticking points for easy review -->

## 🧠 Context Notes
<!-- Key decisions, API snippets, research conclusions, debug notes and error analysis -->

## ⚡ Quick Recovery
- `git pull`
-

## 📅 Task History (Last 7 days)
<!-- Automatically generated, sorted by date in descending order -->

## 🏛️ Archive Links
<!-- Automatically generated, pointing to historical archive files -->
```