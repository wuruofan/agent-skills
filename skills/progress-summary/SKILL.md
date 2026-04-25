---
name: progress-summary
description: Generate detailed current progress summary including uncommitted changes and code statistics - get status without committing
version: 1.5.0
---

# Progress Summary

Generate a summary of the current progress, including in-progress tasks, uncommitted changes, and code change statistics, without performing a Git commit.

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

### Step 1: Detect Project Root Directory

- Search upward from the current directory until finding a directory containing `.git` or `PROGRESS.md` as the project root.

### Step 2: Check and Process PROGRESS.md

- Check if `PROGRESS.md` exists:
  - If it doesn't exist: Prompt user to initialize it (as per Global Rules)
  - If it exists and old format detected:
    - Create a backup: `PROGRESS.md.bak.<timestamp>`
    - Convert old content to new structure
    - Inform user: "PROGRESS.md has been upgraded to the new format. A backup has been created at PROGRESS.md.bak.<timestamp>."
  - If already using new format: Proceed normally

### Step 3: Collect Information

1. **Read PROGRESS.md**:
   - Extract `🎯 Current Focus` tasks
   - Extract `📥 Todo Queue` items
   - Extract `✅ Recently Completed` tasks

2. **Analyze Git Status**:
   - Execute `git status --short` to get uncommitted changes
   - Execute `git diff --stat` to analyze code change statistics
   - Execute `git log -5 --oneline` to get recent commits

### Step 4: Generate Summary

- Consolidate information from PROGRESS.md and Git
- Calculate code change statistics
- Identify WIP tasks
- Generate next steps suggestions

### Step 5: Output Summary

- Output to terminal in text format
- Automatically detect language based on user input and commit history

## Examples

```
# Generate summary
/progress-summary
```

## Output Example

```
📊 Current Progress Summary
==========================

🎯 Current Focus:
- Implement user authentication API (WIP)
- Optimize database query performance

📥 Todo Queue:
- Fix login page bug
- Design database schema

💻 Uncommitted Changes:
- src/api/auth.js (+25/-5)
- src/utils/db.js (+12/-2)

📈 Code Changes:
- Added: 37 lines
- Deleted: 7 lines
- Modified: 0 lines

🔄 Recent Commits:
- 5e721bf feat: add user authentication endpoint
- 9a34cde fix: resolve database connection issue

💡 Next Steps:
- Complete user authentication API
- Run tests before committing
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