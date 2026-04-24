# /progress archive

## Command

Archive project progress history records, move task history older than 7 days to archive directory, keep PROGRESS.md at a reasonable size.

## Execution Flow

### Step 1: Detect Project Root Directory

- Search upward from the current directory until finding a directory containing `.git` or `PROGRESS.md` as the project root.

### Step 2: Read PROGRESS.md

- Read `PROGRESS.md` file content
- Parse `📅 Task History` section, extract date and task information

### Step 3: Identify Records to Archive

- Calculate date difference between current date and each history record
- Identify records older than 7 days

### Step 4: Create Archive Directory Structure

- Create archive directory: `docs/progress/year/month/`
- Calculate current week number of the month (calculated by natural week, starting from the 1st day of the month as week 1)
- Determine archive file path: `docs/progress/year/month/week_weeknumber.md`

### Step 5: Move History Records to Archive File

- Move records older than 7 days to corresponding week's archive file
- Update `📅 Task History` section in PROGRESS.md, keep only last 7 days of records
- Update `🏛️ Archive Links` section in PROGRESS.md, add new archive file links

### Step 6: Commit Archive Files

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
/progress archive

# View archive files
ls -la docs/progress/2026/04/
```

## Notes

1. **Automatic Archive**: checkpoint command automatically triggers archive operation, no need to execute manually.
2. **Retention Period**: Default to keep last 7 days of history records, can be modified via configuration file.
3. **Directory Creation**: If archive directory does not exist, it will be created automatically.
4. **Git Commit**: Archive operation will be automatically committed to Git, ensuring safe storage of history records.