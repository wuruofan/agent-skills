---
name: progress-archive
description: Use when a major task or all its phases are finished, when PROGRESS.md grows too long, when user asks to archive, clean up, trim, or move completed work to history (归档, 归档进度, 清理已完成, 任务完成, 进度太长, 收尾, 清理, 精简, trim, 太长了, 堆积)
version: 1.9.0
---

# Progress Archive

Archive completed major tasks from PROGRESS.md. Keeps PROGRESS.md concise while preserving full development history for future reference.

## Two Modes

LLM auto-selects mode based on user intent — no CLI flags needed.

### Mode A: Major Task Archive (default)

Triggered by: "归档", "archive", "任务完成", "收尾", or when save detects a fully-completed major task.

Archive an entire completed major task (all phases ✅) to a standalone file.

### Mode B: Trim

Triggered by: "清理", "精简", "trim", "太长了", "堆积", "进度太长", or when save bloat detection triggers.

Lightweight cleanup: trim Recently Completed to recent 2-3 items, remove fixed Blockers, archive overflow items to a monthly trim file. Does NOT require a major task to be fully complete.

## Global Rules

1. **Project Root Detection**: Search upward from current directory until finding `.git` or `PROGRESS.md`.
2. **File Path**: All operations target `PROGRESS.md` in project root.
3. **Language Following User**: Analyze commit history and user input to auto-detect language.
4. **If PROGRESS.md does not exist**: Ask user to initialize it.
5. **CRITICAL: Preserve Existing Format**: Never restructure PROGRESS.md. Only extract completed tasks.

## What to Archive

**Archive when:**
- A major task (e.g., "TS Runtime Rewrite") has all phases marked ✅ completed
- User explicitly requests archiving completed work

**Archive content:**
- Task name + completion date
- All phases with their details
- Design document links
- Code statistics
- Key decisions and context

**Keep in PROGRESS.md:**
- Current in-progress tasks
- Next planned tasks
- Recently completed (transition, max 2-3 items)
- Archive links pointing to archived files

## Execution Flow

### Mode Selection

Read user intent from message context:
- Keywords like "清理", "精简", "trim", "太长", "堆积" → Mode B (Trim)
- Keywords like "归档", "archive", "完成", "收尾" → Mode A (Major Task Archive)
- Ambiguous → ask user which mode

---

### Mode A: Major Task Archive

#### Step 1: Read PROGRESS.md

- Read full content
- Identify completed tasks (marked with ✅ or similar completion indicators)
- Detect task structure: major task → phases → details

### Step 2: Identify Tasks to Archive

**Detection criteria:**
- Task section header contains completion indicator (✅, "已完成", "Completed")
- All phases under this task are marked completed
- Task is in "Recently Completed" section (not "Current Focus")

**Ask user confirmation:**
"Found completed task: [Task Name]. Archive it to history?"

### Step 3: Create Archive File

**Archive directory structure:**
```
docs/progress/archive/
├── ts-runtime-rewrite-2026-05.md
├── extensions-mechanism-2026-06.md
├── cleanup-python-2026-07.md
└── README.md (index of all archived tasks)
```

**Archive file format:**
```markdown
# TS Runtime Rewrite

> Completed: 2026-05-05

## Summary
Major task: Rewrite Gateway + TUI runtime from Python to TypeScript

## Phases
### Phase 1: pi-agent-core Validation ✅
- Verified Agent class instantiation
- Tested event subscription
- File: tui/test/pi-agent-core.test.ts

### Phase 2: Tools Module ✅
- 4 core tools: file_read, file_write, file_list, shell_execute
- 25 tests passing
- File: tui/src/tools/core.ts

### Phase 4.1: Gateway TS ✅
...

### Phase 4.3: Harness Hooks ✅
...

## Design Documents
- docs/design/2026-05-03-gateway-ts-rewrite-plan.md
- docs/design/2026-05-04-phase-4-2-captain-crew-starlight-design.md
- docs/design/harness/2026-05-05-harness-design.md

## Code Statistics
- Gateway: 600 lines
- Session: 420 lines
- Tools: 260 lines
- Tests: ~1400 lines

## Key Decisions
- Terminal → Session 1:1 mapping
- Multi SessionManager parallel execution
- Batch execution mode for Crew
```

### Step 4: Update PROGRESS.md

- Remove archived task content
- Keep only current in-progress + next planned + recently completed (max 2-3)
- Update Archive Links section with new archive file

### Step 5: Update Archive Index

Update `docs/progress/archive/README.md`:
```markdown
# Progress Archive Index

## Completed Tasks
- [TS Runtime Rewrite (2026-05)](./ts-runtime-rewrite-2026-05.md) - Gateway + TUI Python → TypeScript
- [Extensions Mechanism (2026-06)](./extensions-mechanism-2026-06.md) - Skills system
- [Cleanup Python (2026-07)](./cleanup-python-2026-07.md) - Remove legacy code
```

### Step 6: Commit Archive (Optional)

- Ask: "Commit archive changes?"
- If yes: `git add docs/progress/archive/ PROGRESS.md && git commit -m "archive: [Task Name] completed"`
- If no: Just write files, let user commit manually

---

### Mode B: Trim

#### Step 1: Read PROGRESS.md

- Read full content
- Count items in Recently Completed
- Identify fixed Blockers/Known Issues (marked ✅ fixed)
- Identify completed items in Next Phases (marked ✅)

#### Step 2: Identify Items to Trim

**Trim targets:**
- Recently Completed: keep only the most recent 2-3 items, archive the rest
- Blockers / Known Issues: remove items marked ✅ fixed (no archive needed — commit history has the record)
- Next Phases: remove items marked ✅ (they belong in Recently Completed, not here)

**Ask user confirmation:**
"Recently Completed has N items. Trim to 3 most recent? (Older items archived to `docs/progress/archive/trim-YYYY-MM.md`)"

#### Step 3: Create Trim Archive File

If trimming Recently Completed overflow:

**File**: `docs/progress/archive/trim-YYYY-MM.md` (append to existing if present)

```markdown
# Progress Trim — YYYY-MM

> Archived from Recently Completed

## Trimmed Items

### [Task Name] ✅ (original date)
[Original content preserved verbatim]

### [Another Task] ✅ (original date)
[Original content preserved verbatim]
```

#### Step 4: Update PROGRESS.md

- Remove trimmed items from Recently Completed
- Remove fixed Blockers/Known Issues
- Remove ✅ items from Next Phases
- Update Archive Links section with trim file reference

#### Step 5: Update Archive Index

Append to `docs/progress/archive/README.md`:
```
- [Trim YYYY-MM](./trim-YYYY-MM.md) - N recently completed items trimmed
```

#### Step 6: Commit (Optional)

Same as Mode A Step 6.

## Examples

```
# Archive completed major task (Mode A)
/progress-archive
"归档已完成的任务"

# Trim Recently Completed overflow (Mode B)
"清理已完成项"
"PROGRESS 太长了，精简一下"
"trim"
```

## Archive Trigger Detection in progress-save

When `/progress-save` updates PROGRESS.md, it triggers archive suggestions at two levels:

**Major task completion (→ Mode A):**
1. Find major task sections with ✅ or "已完成" markers
2. Check if all phases under this task are completed
3. If found: Prompt user "Task '[Name]' appears complete. Archive it with /progress-archive?"

**Bloat detection (→ Mode B):**
1. PROGRESS.md > 300 lines → suggest /progress-archive trim
2. Recently Completed > 5 items → suggest /progress-archive trim

This lets save act as a trigger point for archive suggestions.

## Archive vs Save

| Action | progress-save | progress-archive |
|--------|---------------|------------------|
| Update PROGRESS.md | ✓ (update phases) | ✓ (remove completed tasks) |
| Create archive files | ✗ | ✓ |
| Trigger | Before each commit | When major task completes |
| Frequency | High (daily) | Low (weekly/monthly) |

**Recommended workflow:**
- `/progress-save` before each commit
- When save detects completed task → prompts archive
- `/progress-archive` to preserve history and clean PROGRESS.md