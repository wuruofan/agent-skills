---
name: progress-archive
description: Use when a major task is complete (all phases finished) - archives completed task to history files, keeps PROGRESS.md concise
version: 1.7.0
---

# Progress Archive

Archive completed major tasks from PROGRESS.md. Keeps PROGRESS.md concise while preserving full development history for future reference.

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

### Step 1: Read PROGRESS.md

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

## Examples

```
# Archive completed task (auto-detect)
/progress-archive

# Archive specific task
/progress-archive --task="TS Runtime Rewrite"
```

## Archive Trigger Detection in progress-save

When `/progress-save` updates PROGRESS.md, it checks for completed tasks:

**Detection logic:**
1. Find major task sections with ✅ or "已完成" markers
2. Check if all phases under this task are completed
3. If found: Prompt user "Task '[Name]' appears complete. Archive it with /progress-archive?"

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