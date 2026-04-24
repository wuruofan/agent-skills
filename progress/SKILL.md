---
name: progress
description: Project development progress tracking, work session recovery, daily/weekly report generation, history archiving, and current progress summaries
version: 1.4.0
commands:
  - show
  - checkpoint
  - restore
  - brief
  - archive
  - summary
---

# Progress Skill

Track project development progress, enable cross-device work session saving and recovery, automatically generate daily/weekly reports, manage history archiving, and provide current progress summaries.

## Commands Overview

| Command | Description | Use Case |
|---------|-------------|----------|
| `show` | Display a quick overview of project status from PROGRESS.md | Daily quick check of project status |
| `checkpoint` | Save progress, update PROGRESS.md, and create a Git commit | Regularly save work progress |
| `restore` | Recover work session and sync remote progress | Resume work on a new device |
| `brief` | Generate daily/weekly reports with code change statistics | Generate periodic reports |
| `archive` | Archive historical records older than 7 days | Maintain PROGRESS.md size |
| `summary` | Generate a detailed summary of current progress including uncommitted changes | Get detailed current status without committing |

## Global Rules

1. **Project Root Detection**: Search upward from the current directory until finding a directory containing `.git` or `PROGRESS.md` as the project root.
2. **File Path**: All operations target `PROGRESS.md` in the project root directory.
3. **Language Following User**: Analyze commit history and user input to automatically detect language and generate content in the corresponding language. Support for manually specifying language via `--lang` parameter.
   - **Detection Priority**: User input > Recent commit messages > System locale
   - **Supported Languages**: English (en) and Chinese (zh)
4. **If PROGRESS.md does not exist**:
   - Ask user: "Project progress tracking not detected. Would you like to initialize PROGRESS.md?"
   - After user confirmation, write the following standard structure directly in the project root directory:

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
<!-- Key decisions, API snippets, research conclusions -->

## ⚡ Quick Recovery
- `git pull`
-

## 📅 Task History (Last 7 days)
<!-- Automatically generated, sorted by date in descending order -->

## 🏛️ Archive Links
<!-- Automatically generated, pointing to historical archive files -->
```
