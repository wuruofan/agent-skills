---
name: progress-save
description: Use when user asks to update, save, record, or write to PROGRESS.md (更新进度, 保存进度, 记录进度, 更新 PROGRESS.md), or before git commit/stash/checkout/PR/push, or at end of work session
version: 1.2.0
---

# Progress Save

Update PROGRESS.md to record current work state. No git operations involved - purely document update.

## Global Rules

1. **Project Root Detection**: Search upward from current directory until finding a directory containing `.git` or `PROGRESS.md`.
2. **File Path**: All operations target `PROGRESS.md` in project root.
3. **Language Following User**: Analyze commit history and user input to auto-detect language.
   - **Detection Priority**: User input > Recent commit messages > System locale
   - **Supported Languages**: English (en) and Chinese (zh)
4. **If PROGRESS.md does not exist**: Ask user to initialize it.
5. **CRITICAL: Preserve Existing Format**: If PROGRESS.md already exists, preserve its structure and sections. Only update content, never restructure.

## Format Preservation Policy

**Do NOT standardize project-specific formats.**

Projects may have custom PROGRESS.md structures tailored to their needs. The skill must:

- ✅ Preserve existing section names and order
- ✅ Update content within existing sections
- ✅ Add content to appropriate existing sections
- ❌ Never add standard sections (🎯, 📥, ✅, etc.) if project uses different format
- ❌ Never reorder or rename existing sections
- ❌ Never trigger "format upgrade" or "backup conversion"

**Examples of valid custom formats:**
- `## 当前分支` + `## 最近提交` (project tracking)
- `## Phase 1-4` (milestone-based tracking)
- `## 测试状态` + `## 代码统计` (technical tracking)
- Any structure the project team finds useful

## Execution Flow

### Step 1: Collect Context

- Read current `PROGRESS.md` content.
- Execute `git status --short` to get changed file list.
- Execute `git diff --stat` to understand change scope.
- Execute `git log -5 --oneline` to learn historical context.
- Analyze changes to infer task progress.

### Step 2: Intelligent Section Detection

Based on existing PROGRESS.md structure, find appropriate sections for updates:

| Content Type | Possible Section Names (examples) |
|--------------|-----------------------------------|
| Last update time | `> Last updated`, `> 最后更新` |
| Current focus | `🎯`, `Current Focus`, `当前`, `Current` |
| Next steps | `📥`, `Next Phases`, `Todo`, `下一步`, `Next` |
| Paused tasks | `⏸️`, `Paused Tasks`, `暂停`, `Hold` |
| Completed tasks | `✅`, `Completed`, `已完成`, `Recently Completed` |
| Issues/Blockers | `🧱`, `Blockers`, `Issues`, `问题` |
| Notes | `🧠`, `Context Notes`, `Notes`, `备注`, `Context` |

**Matching logic:**
1. Search for section names matching content type patterns
2. If found, update within that section
3. If not found, append to most relevant existing section or ask user

### Step 3: Update Content (Preserve Structure)

Update content while preserving the exact structure:

1. **Last updated time**: Update header timestamp if exists.
2. **Completed tasks**: Add to appropriate completed section.
3. **In-progress**: Update status descriptions.
4. **Key files**: If there's a "恢复" or "Recovery" section, add key file paths.

**Cleanup rules:**
- Trim oversized sections only if project format expects it
- Merge duplicates
- Do NOT add new sections unless explicitly needed

### Step 4: Detect Completed Major Tasks

After updating content, check for completed major tasks:

**Detection criteria:**
- Major task section header contains ✅ or "已完成" marker
- All phases/subtasks under this major task are marked completed
- Task is in "Recently Completed" section (not "Current Focus")

**If detected:**
Prompt user: "Task '[Task Name]' appears complete with all phases finished. Archive it with `/progress-archive` to preserve history and keep PROGRESS.md concise?"

**Example detection:**
```
## ✅ TS Runtime Rewrite Phase 1-4
### Phase 1: ✅
### Phase 2: ✅
### Phase 4.1: ✅
...
→ All phases complete → Prompt archive suggestion
```

### Step 5: Write to Disk

Write updated content preserving exact structure and formatting style.

## Reference Standard Format (Optional)

This standard format is provided as **reference only**. Projects can adopt it if they find it useful, but it's never enforced:

```markdown
# Progress

> Last updated: {CURRENT_DATE}

## 🎯 Current Focus
<!-- Major task currently in progress, max 1-2 items -->

## 📥 Next Phases
<!-- Phases to do next -->

## ⏸️ Paused Tasks
<!-- Tasks paused mid-way, with completion %, blockers, and entry points -->

## ✅ Recently Completed Phases
<!-- Last 2-3 completed phases, archive when major task done -->

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
| `⏸️ Paused Tasks` | Paused mid-way tasks | restore, save (track paused state) |
| `✅ Recently Completed Phases` | Completed phases (max 2-3) | save, archive (detects completion) |
| `🧱 Blockers & Issues` | Problems | restore (quick reference) |
| `🧠 Context Notes` | Decisions, design docs | restore (context recall) |
| `⚡ Quick Recovery` | Restore commands | restore |
| `🏛️ Archive Links` | Archived major tasks | archive (updates links) |

## Difference from Other Skills

| Feature | progress-save | progress-archive | progress-summary |
|---------|---------------|------------------|------------------|
| Update PROGRESS.md | ✓ | ✓ (remove completed) | ✗ |
| Create archive files | ✗ | ✓ | ✗ |
| Git operations | ✗ | Optional commit | ✗ |
| Generate summary | ✗ | ✗ | ✓ |
| Trigger detection | Archive suggestion | N/A | N/A |

**Trigger timing:**
- Before `git commit`: Update progress before creating commit
- Before `git stash`: Save current state before stashing
- Before `git checkout`: Update before switching branches
- Manual: User explicitly requests progress update

See README for recommended AGENTS.md configuration to let LLM trigger this skill automatically.