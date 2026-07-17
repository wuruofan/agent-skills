---
name: progress-archive
description: Use when a major task or all its phases are finished, when PROGRESS.md grows too long, when user asks to archive, clean up, trim, or move completed work to history, when an Unverified/待手测 table needs verify-cleanup (归档, 归档进度, 清理已完成, 任务完成, 进度太长, 收尾, 清理, 精简, trim, 太长了, 堆积, verify-cleanup, 验证清理, 待手测清理, unverified 表清理)
version: 2.0.1
---

# Progress Archive

Archive completed major tasks from PROGRESS.md. Keeps PROGRESS.md concise while preserving full development history for future reference.

## Three Modes

LLM auto-selects mode based on user intent — no CLI flags needed.

- **Mode A: Major Task Archive** — triggered by "归档", "archive", "任务完成", "收尾", or when save detects a fully-completed major task. Archive an entire completed major task (all phases ✅) to a standalone file.
- **Mode B: Trim** — triggered by "清理", "精简", "trim", "太长了", "堆积", "进度太长", or when save bloat detection triggers. Lightweight cleanup of Recently Completed to recent 2-3 items; archive overflow to a monthly trim file. Does NOT require a major task to be fully complete. Next Phases / Blockers cleanup is handled automatically by /progress-save. Also covers Unverified-table overflow (see Mode B Step 2 rules).
- **Mode C: Verify-cleanup** — triggered by "verify", "verify-cleanup", "待手测", "unverified", "验证清理", "待手测清理", "unverified 表清理", or when save detects an Unverified/待手测 table with bloat (✅ items > 3 or total items > 10). Clean up the "Unverified / 待手测" table — items in code-shipped + tests-pass + manual-test-pending state. This gray zone is neither Mode A (not a completed major task) nor Mode B (not Recently Completed overflow). Mode C triages items into buckets:
  - Already ✅ verified items → move to corresponding archive file (verified long ago, no reason to stay in Unverified)
  - Items explicitly marked "code shipped" / "won't verify" (e.g. shipped to production, no longer testable) → close directly (remove from Unverified, optionally note in Recently Completed)
  - Long-pending ⏳ items impossible or no longer worth verifying → move to Paused Tasks or archive

## Global Rules

Find project root (upward to `.git`/`PROGRESS.md`); target `PROGRESS.md` in root; language follows user input > commit history > locale (en/zh); if file missing, ask user to init; preserve existing format, never restructure.

## What to Archive

**Archive when:** A major task has all phases marked ✅, OR user explicitly requests archiving.

**Archive content:** Task name + completion date · all phases (1 line per phase + link) · design document links · key decisions (1 line each).

**Keep in PROGRESS.md:** Current in-progress tasks · next planned tasks · Recently Completed (max 2-3) · archive links.

## Execution Flow

### Mode Selection

Read user intent from message context:
- "verify", "verify-cleanup", "待手测", "unverified", "验证清理", "待手测清理" → Mode C
- "清理", "精简", "trim", "太长了", "堆积", "进度太长" → Mode B
- "归档", "archive", "完成", "收尾" → Mode A
- Ambiguous → ask user which mode

**Ambiguity rule**: If user says "清理" (Mode B keyword) but the largest bloat source is an Unverified table rather than Recently Completed, ask whether to run Mode B (Recently Completed trim) or Mode C (Unverified verify-cleanup). Same for the reverse.

---

### Mode A: Major Task Archive

#### Step 1: Read PROGRESS.md

- Read full content
- Identify completed tasks (✅ or similar completion indicators)
- Detect task structure: major task → phases → details

#### Step 2: Identify Tasks to Archive

**Detection criteria:**
- Task section header contains completion indicator (✅, "已完成", "Completed")
- All phases under this task are marked completed
- Task is in "Recently Completed" section (not "Current Focus")

**Ask user confirmation:** "Found completed task: [Task Name]. Archive it to history?"

#### Step 3: Create Archive File

**Directory structure:**
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
Rewrite Gateway + TUI runtime from Python to TypeScript. 4 phases, 1025 tests pass.

## Phases
- Phase 1: pi-agent-core Validation ✅ — Agent class + event subscription. [test](tui/test/pi-agent-core.test.ts)
- Phase 2: Tools Module ✅ — 4 core tools, 25 tests. [code](tui/src/tools/core.ts)
- Phase 4.1: Gateway TS ✅
- Phase 4.3: Harness Hooks ✅

## Design Documents
- [Gateway TS rewrite plan](docs/design/2026-05-03-gateway-ts-rewrite-plan.md)
- [Captain-Crew design](docs/design/2026-05-04-phase-4-2-captain-crew-starlight-design.md)
- [Harness design](docs/design/harness/2026-05-05-harness-design.md)

## Key Decisions
- Terminal → Session 1:1 mapping
- Multi SessionManager parallel execution
- Batch execution mode for Crew
```

> **Note:** Archive files are L2 (Reference) in the Progressive Disclosure model. Each phase 1 line + link, not full implementation notes. Design details belong in the spec/plan docs they link to.

#### Step 4: Update PROGRESS.md

- Remove archived task content
- Keep only current in-progress + next planned + Recently Completed (max 2-3)
- Update Archive Links section with new archive file

#### Step 5: Update Archive Index

Update `docs/progress/archive/README.md`:
```markdown
# Progress Archive Index

## Completed Tasks
- [TS Runtime Rewrite (2026-05)](./ts-runtime-rewrite-2026-05.md) - Gateway + TUI Python → TypeScript
- [Extensions Mechanism (2026-06)](./extensions-mechanism-2026-06.md) - Skills system
- [Cleanup Python (2026-07)](./cleanup-python-2026-07.md) - Remove legacy code
```

#### Step 6: Commit Archive (Optional)

Ask: "Commit archive changes?"
- Yes: `git add docs/progress/archive/ PROGRESS.md && git commit -m "archive: [Task Name] completed"`
- No: just write files, let user commit manually

---

### Mode B: Trim

#### Step 1: Read PROGRESS.md

- Read full content
- Count items in Recently Completed
- Count items in Unverified / 待手测 table (if present)
- Note: Next Phases / Blockers cleanup is handled by /progress-save, not here

#### Step 2: Identify Items to Trim

**Primary trim target:** Recently Completed.
- Keep only the most recent 2-3 items, archive the rest
- Blockers / Next Phases are NOT handled here — /progress-save cleans them automatically

**Secondary trim target:** Unverified / 待手测 table overflow.
- If Unverified table has > 3 items already marked ✅ verified → suggest redirecting to Mode C (these don't belong in Unverified anymore; trimming them here would lose the verify-state nuance)
- If Unverified table total items > 10 → suggest Mode C for triage
- Mode B itself does NOT touch Unverified items directly — Unverified cleanup always goes through Mode C (it requires per-item bucketing decisions, not bulk trim)

**Ask user confirmation:**
"Recently Completed has N items. Trim to 3 most recent? (Older items archived to `docs/progress/archive/trim-YYYY-MM.md`)"

If Unverified overflow detected, append: "Also: Unverified table has N items (M already ✅). Run `/progress-archive` with 'verify-cleanup' to triage — Mode B doesn't touch Unverified items directly."

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
- Update Archive Links section with trim file reference

#### Step 5: Update Archive Index

Append to `docs/progress/archive/README.md`:
```
- [Trim YYYY-MM](./trim-YYYY-MM.md) - N recently completed items trimmed
```

#### Step 6: Commit (Optional)

Same as Mode A Step 6.

---

### Mode C: Verify-cleanup

#### Step 1: Read PROGRESS.md + Detect Unverified Section

- Read full content
- Locate the Unverified / 待手测 table (section names vary: "Unverified", "待手测", "TTY 待手测", "Manual Test Pending", etc. — use the same intelligent section detection as save/merge)
- If no Unverified section found → exit: "No Unverified/待手测 table detected. Mode C has nothing to do."
- Count total items and tally by status: ✅ verified / ⏳ pending / shipped-won't-verify / other

#### Step 2: Bucket Each Item (3-way triage)

For each item in the Unverified table, classify into one of three buckets:

**Bucket 1 — Already verified ✅**:
- Item has ✅ marker, or description clearly states "verified" / "已验证" / "测试通过"
- Action: Move to corresponding archive file (or Recently Completed if recent enough). These no longer belong in Unverified.

**Bucket 2 — Explicitly closed (won't verify)**:
- Item description contains "代码已 ship" / "shipped" / "不再验证" / "won't verify" / "no longer testable" / "close" etc.
- Action: Close directly — remove from Unverified. Optionally add a 1-line note to Recently Completed with `closed (won't verify)` marker.

**Bucket 3 — Long-pending ⏳ (impossible or not worth verifying)**:
- Item has been ⏳ for a long time (use judgement based on surrounding dates / commits), or verification is no longer meaningful (e.g. feature was replaced, code was refactored away)
- Action: Move to Paused Tasks (if may revisit) or archive (if clearly obsolete)

**Bucket 4 — Keep in Unverified**:
- Items that are genuinely still pending verification and worth doing soon
- Action: Leave in Unverified table. These are the "real" pending items.

**Hard rule**: When unsure whether an item is Bucket 2 vs Bucket 3 (close vs pause), ask the user. Do not silently close items that might still be worth verifying.

#### Step 3: Ask User Confirmation (Batched)

Present the triage result as a single batched question (or multiple batches if > 4 ambiguous items):

```
Unverified table triage (N items):

✅ Move to archive (Bucket 1): #2, #3, #4, #5, #9, #10, #11, #12, #13
🚫 Close directly (Bucket 2):  #18 (code shipped)
⏸️ Move to Paused (Bucket 3): #7 (long-pending)
⏳ Keep in Unverified (Bucket 4): #6, #14, #15

❓ Confirm this triage? (or adjust per-item)
  ◉ Apply as-is              (Recommended)
  ◯ Adjust specific items
  ◯ Cancel
```

If user picks "Adjust specific items", ask which item(s) and which bucket to move them to.

#### Step 4: Execute Bucket Actions

**For Bucket 1 (✅ → archive)**:
- Append items to the most relevant existing archive file (by topic/date), or create a new `verify-cleanup-YYYY-MM.md` file if no good match
- Archive file format: same as Mode B trim file — preserve original item content verbatim, add `### [Item ID/name] ✅ (verified date)` header

**For Bucket 2 (close)**:
- Remove from Unverified table
- If user opted to note in Recently Completed: add 1-line entry like `#18 long-task chat.send timeout — closed (won't verify, code shipped). commit: \`abc1234\``

**For Bucket 3 (→ Paused/archive)**:
- If moving to Paused Tasks: add as new entry with pause reason "verification deferred — long-pending, may revisit"
- If moving to archive: append to `verify-cleanup-YYYY-MM.md` with `### [Item ID/name] ⏸️ (deferred)` header

**For Bucket 4 (keep)**:
- Leave in Unverified table untouched

#### Step 5: Update PROGRESS.md

- Rewrite the Unverified table with only Bucket 4 items remaining
- If Bucket 4 is empty → remove the entire Unverified section (and its heading) from PROGRESS.md
- Update Recently Completed / Paused Tasks sections per Step 4 actions
- Update Archive Links section if new archive file was created

#### Step 6: Update Archive Index

If a `verify-cleanup-YYYY-MM.md` file was created, append to `docs/progress/archive/README.md`:
```
- [Verify-cleanup YYYY-MM](./verify-cleanup-YYYY-MM.md) - N items triaged from Unverified table
```

#### Step 7: Commit (Optional)

Same as Mode A Step 6. Suggested commit message: `archive: verify-cleanup N items from Unverified table`.

## Examples

```
# Archive completed major task (Mode A)
/progress-archive
"归档已完成的任务"

# Trim Recently Completed overflow (Mode B)
"清理已完成项"
"PROGRESS 太长了，精简一下"
"trim"

# Triage Unverified/待手测 table (Mode C)
/progress-archive
"verify-cleanup"
"待手测表清理一下"
"unverified 表里一堆 ✅ 的，移走"
```

## Recommended Workflow

- `/progress-save` before each commit
- When save detects completed task → prompts Mode A archive
- When save detects bloat (Recently Completed overflow or PROGRESS > 300 lines) → prompts Mode B trim
- When save detects Unverified table bloat (✅ items > 3 or total > 10) → prompts Mode C verify-cleanup
- `/progress-archive` to preserve history and clean PROGRESS.md
