---
name: progress-show
description: Display project status overview from PROGRESS.md - quick daily check of current focus, next steps, recent completions
version: 1.5.0
---

# Progress Show

Display a quick overview of project status from PROGRESS.md.

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

1. Locate `PROGRESS.md`.
2. Check if the file exists:
   - If it doesn't exist: Prompt to run `/progress-checkpoint` to initialize.
   - If it exists and old format detected:
     - Create a backup: `PROGRESS.md.bak.<timestamp>`
     - Convert old content to new structure
     - Inform user: "PROGRESS.md has been upgraded to the new format. A backup has been created at PROGRESS.md.bak.<timestamp>."
   - If already using new format: Proceed normally
3. Read and parse the file, extracting the following sections:
   - 🎯 Current Focus tasks
   - 📥 Todo Queue (first 5 items)
   - ✅ Recently Completed task count
   - 📅 Task History (last 3 days)
   - ⚡ Quick Recovery steps
   - 🏛️ Archive Links

## Output Format

```
📊 **Project Progress Overview**
- **Current Focus**: [List tasks under 🎯]
- **Next Steps**: [List first items under 📥]
- **Recently Completed**: [count] items
- **Last 3 Days Tasks**: [List last 3 days under 📅]
- **Recovery Guide**: [List steps under ⚡]
- **Archive Links**: [List recent links under 🏛️]
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