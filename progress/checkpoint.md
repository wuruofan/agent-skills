# /progress checkpoint [comment]

## Command

Save current work progress, intelligently select staging files, update PROGRESS.md, and generate a standardized Git commit.

## Execution Flow

### Step 1: Automatic Collection and Analysis

- Read current content of `PROGRESS.md`.
- Execute `git status --short` to get changed file list.
- Execute `git diff --stat` to understand change scope.
- Execute `git log -5 --oneline` to learn historical commit style.
- Combine code changes and user input [comment] to infer completed/in-progress tasks, generate commit message draft.

### Step 2: Rationality Analysis and Confirmation

- **Change Analysis**: If changes span multiple unrelated modules (e.g., modified both auth and payment) or single file changes exceed 200 lines, prompt user: "⚠️ Detected large scope/cross-module changes,建议 split commit. Split?"
- **Commit Message Confirmation**: Display draft, wait for user modification or confirmation.
  - Standard: Follow Conventional Commits (`type(scope): description`).
  - WIP: Use `wip(scope): description` format for incomplete features.
  - Skip CI: If user input contains `--skip` or `#skip-ci`, append `[skip ci]` at the end of the title.
  - Footer: All commits end with `[checkpoint]` tag on a new line.

### Step 3: Intelligent File Selection

Process based on change situation:

**Case A: Small and clean changes (files ≤ 3 and no sensitive files)**
- Automatically stage all changed files.
- Silently proceed to Step 4.

**Case B: Large changes or suspicious files**
- Display **pending commit file preview panel**:
  ```
  ✅ **Recommended Commit** (3):
    [1] src/api/gateway.py       (+45/-2)
    [2] src/tui/error.py         (+12/-0)
  ⚠️ **Suggested Exclusion** (detected config/log files):
    [3] .env                     (untracked)
    [4] debug.log                (untracked)
  ```
- Prompt user for adjustment instructions (e.g., "exclude 1", "remove .env", "commit all", "only commit 2").
- Parse user intent, build final staging file list.

### Step 4: Write Progress and Commit

1. Update `PROGRESS.md` locally:
   - Update header time `> Last updated: {CURRENT_DATE}`.
   - Status migration: Move completed work to ✅ Recently Completed (only record date, not hash), if WIP keep in 🎯 and update description.
   - Update 📅 Task History: Add today's task records, sorted by date in descending order.
   - Cleanup: Merge duplicates, ensure ✅ section has no more than 5 items, 📅 Task History keeps last 7 days.
   - Update 🏛️ Archive Links: Add links to archive files.
2. Write updated `PROGRESS.md` to disk.
3. Execute Git operations:
   - `git add <final confirmed files> PROGRESS.md`
   - `git commit -m "<confirmed_message>"`
4. Trigger archive operation:
   - Call `/progress archive` command to automatically archive history records older than 7 days.

## Commit Message Standard Format

**Regular Commit:**
```
<type>(<scope>): <subject>

[optional body]

[checkpoint]
```

**WIP Commit:**
```
wip(<scope>): <subject>

[optional body]

[checkpoint]
```

**Skip CI Commit:**
```
<type>(<scope>): <subject> [skip ci]

[optional body]

[checkpoint]
```

**WIP and Skip CI Commit:**
```
wip(<scope>): <subject> [skip ci]

[optional body]

[checkpoint]
```
