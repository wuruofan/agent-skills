---
name: progress-archive
description: Archive project progress history - move records older than 7 days to archive files, keep PROGRESS.md at reasonable size
version: 1.5.0
---

# Progress Archive

Archive project progress history records, move task history older than 7 days to archive directory, keep PROGRESS.md at a reasonable size.

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

### Step 3: Read PROGRESS.md

- Read `PROGRESS.md` file content
- Parse `📅 Task History` section, extract date and task information

### Step 4: Identify Records to Archive

- Calculate date difference between current date and each history record
- Identify records older than 7 days

### Step 5: Create Archive Directory Structure

- Create archive directory: `docs/progress/year/month/`
- Calculate current week number of the month (calculated by natural week, starting from the 1st day of the month as week 1)
- Determine archive file path: `docs/progress/year/month/week_weeknumber.md`

### Step 6: Move History Records to Archive File

- Move records older than 7 days to corresponding week's archive file
- Update `📅 Task History` section in PROGRESS.md, keep only last 7 days of records
- Update `🏛️ Archive Links` section in PROGRESS.md, add new archive file links

### Step 7: Commit Archive Files

- Execute `git add` to add archive files and updated PROGRESS.md
- Execute `git commit` to commit archive operation

## Archive Directory Structure

```
docs/
└── progress/
    ├── 2026/
    │   ├── 04/
    │   │   ├── week_1.md
    │   │   ├── week_2.md
    │   │   └── README.md  # Monthly index file
    │   └── 05/
    │       ├── week_1.md
    │       └── README.md
    └── README.md  # Annual index file
```

## Archive File Format

**week_1.md example**:

```markdown
# Project Progress Archive - 2026-04-01 to 2026-04-07

## 2026-04-07
- ✅ Completed: Implement user authentication API
- 🔄 In Progress: Optimize database query performance

## 2026-04-06
- ✅ Completed: Design database schema
- ✅ Completed: Set up project infrastructure

## 2026-04-05
- ✅ Completed: Initialize project
- 🔄 In Progress: Design database schema
```

## Monthly Index File Format

**README.md example**:

```markdown
# 2026-April Archive

## Week Archives
- [Week 1](./week_1.md) - 2026-04-01 to 2026-04-07
- [Week 2](./week_2.md) - 2026-04-08 to 2026-04-14
- [Week 3](./week_3.md) - 2026-04-15 to 2026-04-21
- [Week 4](./week_4.md) - 2026-04-22 to 2026-04-28
```

## Examples

```
# Execute archive operation
/progress-archive

# View archive files
ls -la docs/progress/2026/04/
```

## Notes

1. **Automatic Archive**: `progress-checkpoint` automatically triggers archive operation, no need to execute manually.
2. **Retention Period**: Default to keep last 7 days of history records, can be modified via configuration file.
3. **Directory Creation**: If archive directory does not exist, it will be created automatically.
4. **Git Commit**: Archive operation will be automatically committed to Git, ensuring safe storage of history records.

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