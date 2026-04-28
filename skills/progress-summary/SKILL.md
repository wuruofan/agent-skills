---
name: progress-summary
description: Generate compact session summary including project state and chat context for new session continuity
version: 1.6.0
---

# Progress Summary

Generate a compact summary combining project state from PROGRESS.md and current chat session context. Enables seamless new session continuity by capturing essential context without unnecessary verbosity.

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
6. **Session Context Capture**: Capture essential chat session context without bloat:
   - User intent and goals
   - Key technical decisions made
   - Errors encountered and resolutions
   - Files modified and their purpose
   - Keep concise (aim for ~500 tokens max)
   - Format for easy copy-paste into new sessions

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
   - Extract `🧠 Context Notes`
   - Extract `🧱 Blockers & Issues`

2. **Analyze Git Status**:
   - Execute `git status --short` to get uncommitted changes
   - Execute `git diff --stat` to analyze code change statistics
   - Execute `git log -5 --oneline` to get recent commits

3. **Capture Chat Session Context**:
   - Retrieve the current chat conversation history
   - Extract key topics and discussion points
   - Identify decisions made during the session
   - Capture action items and follow-up tasks
   - Summarize the conversation in a concise format for new session continuity

### Step 4: Generate Summary

- Consolidate information from PROGRESS.md, Git, and chat session
- Calculate code change statistics
- Identify WIP tasks and current focus
- Synthesize chat conversation into key points and decisions
- Generate next steps suggestions based on combined context

### Step 5: Output Summary

- Output to terminal in text format
- Include comprehensive session context for new session continuity
- Automatically detect language based on user input and commit history
- Format output for easy copy-paste into new sessions

## Examples

```
# Generate summary
/progress-summary
```

## Output Example

```
📊 Session Summary
=================

🎯 Current Focus:
- Implement user authentication API (WIP)

📋 Session Context:
- User Intent: Build secure authentication system
- Key Decisions: JWT tokens, Redis session storage
- Errors Fixed: JWT signing algorithm mismatch
- Files Modified: src/api/auth.js, src/utils/db.js

🧱 Blockers:
- Waiting for API design review

📥 Next Steps:
1. Complete authentication API
2. Add rate limiting middleware

💻 Uncommitted: 2 files | +37/-7 lines
```

## Usage for New Session Continuity

After running `/progress-summary`, you can use the output to quickly catch up in a new session:

```
# Start new session with context
/new-session
---
📊 Session Summary for Continuity
---
🎯 Current Focus: Implement user authentication API (WIP), Optimize database query performance
📥 Next Tasks: Fix login page bug, Design database schema
💬 Key Decisions: JWT-based auth, Redis session storage, rate limiting needed
💻 Changes: src/api/auth.js (+25/-5), src/utils/db.js (+12/-2)
---
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