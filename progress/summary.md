# /progress summary

## Command

Generate a summary of the current progress, including in-progress tasks, uncommitted changes, and code change statistics, without performing a Git commit.

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
/progress summary
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