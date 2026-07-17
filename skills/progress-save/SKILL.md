---
name: progress-save
description: Use when user asks to update, save, record, or write to PROGRESS.md (更新进度, 保存进度, 记录进度, 更新 PROGRESS.md), or before git commit/stash/checkout/PR/push, or at end of work session
version: 1.7.0
---

# Progress Save

Update PROGRESS.md to record current work state. No git operations involved - purely document update.

## Global Rules

Find project root (upward to `.git`/`PROGRESS.md`); target `PROGRESS.md` in root; language follows user input > commit history > locale (en/zh); if file missing, ask user to init; **preserve existing format, never restructure** (see Format Preservation Policy below).

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

## Progressive Disclosure Principle

**PROGRESS.md is a state board, not a knowledge base.** It answers "what's the current status?" at a glance — not "how does it work?" or "what did we try?"

### The Three Layers

| Layer | Where | Content |
|-------|-------|---------|
| **L0 — State + Key Context** | PROGRESS.md | Status + key context for restoring the scene (rationale, findings, decisions); link spec/plan if available, summarize briefly if not |
| **L1 — Detail** | spec / plan / design docs | Full design decisions, implementation steps, test methods, alternative approaches |
| **L2 — Reference** | code comments, commit messages, archive files | Full history, alternatives considered, implementation details |

**Core rule: If a detail has a home in L1 or L2, link to it. If not (bug fix, ad-hoc findings, experiment records), summarize briefly in PROGRESS.md.**

### Brevity Guidelines

Principles are the primary guide; Size Budget is the objective backstop. Use principles to judge "is this enough?", then Size Budget to check "is this over?". For each entry, ask: **"Next time I come back, can I tell current state and why from this line alone?"** If yes, stop.

| Section | Principle | Example |
|---------|-----------|---------|
| 🎯 Current Focus | Each item: task name + current status (1-2 sentences). Multiple items OK, but each must be tight. Link spec/plan if available | `Phase 5 TTY validation — code in place, real TTY validation deferred. spec: [link] plan: [link]` |
| 📥 Next Phases | Table form, one phase per row. Goal column short (≤15 chars). Link spec/plan if available | `Phase B \| Command system \| [spec] \| [plan] \| ⏳ pending` |
| ⏸️ Paused Tasks | Each item: task name + pause reason (1 sentence) + restart entry (branch/file). Link design details to docs | `Sandbox shell audit — Plan A covers 80%, remaining 20% needs per-command audit. Entry: \`git checkout wip/sandbox-shell-audit\`. Design: [link]` |
| ✅ Recently Completed | Each item: task name + date + 1-sentence summary. Link archive if available | `Tool output mgmt Phase 1-4 (07-02) — three root causes fixed, 1025 tests pass. [archive]` |
| 🧱 Blockers | Each item: problem + status, 1-2 lines | `Per-call timeout — refactor shipped, long-task manual test pending` |
| 🧠 Context Notes | Key info for restoring context: rationale, dev findings, experiment records, key decisions. Each item 1-3 sentences. Large architecture diagrams / full code flows → link to doc | `Captain-Crew orchestration: only one Captain turn per session, enforced by turnGuard state machine. See [architecture.md]` |
| ⚡ Quick Recovery | Commands first; add ≤1 line comment if needed | `bun run test:gateway  # gateway only, ~12s` |

### Size Budget

Objective ruler. Soft limits and hard-trigger compression per section — no "looks fine" judgment calls.

| Section | Soft limit | Hard-trigger compression |
|---------|------------|--------------------------|
| 🎯 Current Focus | 10 lines / 3 items | >15 lines OR >3 items |
| 📥 Next Phases | Table ≤8 rows | >10 rows |
| ⏸️ Paused Tasks | Each item ≤5 lines / section ≤30 lines | Single item >5 lines |
| ✅ Recently Completed | ≤3 items | >3 items (also triggers archive) |
| 🧱 Blockers & Issues | ≤5 lines | >5 lines OR all strikethrough |
| 🧠 Context Notes | Each item ≤3 sentences / section ≤30 lines | Single item >3 sentences OR total >30 lines |
| ⚡ Quick Recovery | Total ≤10 lines | bash block >5 lines OR open list >3 files |

### Counter-Examples (common anti-patterns)

These are patterns LLMs repeatedly make when writing PROGRESS.md. "❌" shows the original form; "✅" shows the correct form.

**Anti-pattern A — Next Phases filled with archived items**

❌ `| Phase 1 | ... | spec | plan | 📦 archived ([archive](...)) |` × 4 rows
✅ Delete them outright. Archive Links section already links them; Next Phases only holds truly pending work.

**Anti-pattern B — Blockers section entirely resolved**

❌ `- ~~streaming output~~ → ✅ implemented (v0.5.x ship)` × 6 lines
✅ Compress whole section to "no active blockers" one line; resolved items go to git log / archive.

**Anti-pattern C — Full test methodology stuffed in table cell**

❌ `⏳ Not hand-tested. (a) \`bun run dev\` send a long task (>5min cargo build / npm install / large file_read) — chat.send should not falsely timeout; (b) 5-min default fallback: gateway truly deadlocked should reject within 5min instead of hanging forever`
✅ `⏳ Not hand-tested (long-task chat.send no false timeout)` — detailed steps in test file comments

**Anti-pattern D — ASCII architecture diagram in Context Notes**

❌ 17-line ASCII diagram + multiple tables + test stats + reference source list (~60 lines)
✅ `Captain-Crew orchestration: only one Captain turn per session, enforced by turnGuard state machine. See [architecture.md](...)`

**Anti-pattern E — Quick Recovery with 30-line bash block + multi-file open list**

❌ 30+ line `bun test --parallel=4 ...` block + 7 `open: src/...` file paths
✅ `bun run test:gateway  # gateway only, ~12s` — full script in CLAUDE.md

**Anti-pattern F — Paused Tasks with multi-paragraph "restart prerequisites"**

❌ 5+ paragraphs "Restart prereqs: ... touches span data layer + render layer + shortcuts + collapse state machine + token stats integration, split across multiple PRs. Blocked on v0.7.0 full unblock..."
✅ 1 sentence: `Tool/Thinking friendly display — perf blocking unblocked in v0.7.0, render layer design pending after reading cc-black trilogy. Design: [link]`

**Anti-pattern G — Recently Completed merges multiple releases into 1 line**

❌ `v0.8.0 ~ v0.8.14 spec+plan refinement / cache→caches / grouped tool collapse / captain_turn_end / ... (21 items total)`
✅ Split into 1 `Trim YYYY-MM` archive; archive README lists index; PROGRESS.md keeps 1 archive link line.

### Handling Items Without Spec/Plan

Many changes (bug fixes, small refactors, config tweaks) don't have a dedicated spec/plan. How to record them:

- **Bug fix**: `Fixed XXX bug — root cause: 1 sentence. commit: \`abc1234\`` (root cause in brief, not the fix process)
- **Small refactor/config**: `XXX refactor — purpose: 1 sentence. commit: \`abc1234\``
- **Ad-hoc findings/experiment records**: Write 1-3 sentence key findings directly in Context Notes; no need to create a spec

### Link Conventions

When referencing L1/L2 details, use relative paths:

```markdown
- **Task name** — status (1 sentence). spec: [spec-name.md](docs/specs/xxx.md) plan: [plan-name.md](docs/plans/xxx.md)
- **Bug fix** — root cause: 1 sentence. commit: `abc1234`
```

If no spec/plan exists, write a concise summary directly — do NOT create a spec file just for a bug fix.

## Pre-Write Self-Check (mandatory)

Before writing or updating any entry, run through this checklist. If any item hits → don't write / rewrite / link out, do not proceed to Step 5.

- [ ] Contains "this update added…/next plan…/next step is to do X" running narrative? → Delete
- [ ] Cell or entry text > 3 sentences? → Compress to 1-2 sentences; link details to spec/doc
- [ ] Contains full test method / expected result / acceptance steps? → Move to spec or test file comments
- [ ] Contains ASCII diagram / multi-layer code flow / full module dependency graph? → Move to ARCHITECTURE.md, keep 1 sentence + link
- [ ] bash/code block > 5 lines? → Move to README/CLAUDE.md, keep 1-line reference
- [ ] Multiple releases / tasks merged into 1 line (using `/` to chain N items)? → Split apart, or merge the whole group into 1 archive
- [ ] In the "wrong section"? Common misplacements:
    - Archived / shipped items appearing in `📥 Next Phases` or `🎯 Current Focus`
    - Resolved items appearing in `🧱 Blockers & Issues` (strikethrough counts)
    - Design sketches / restart prerequisites appearing in `⏸️ Paused Tasks` (should link to spec)
- [ ] Exceeds the section's Size Budget soft limit? → Triggers mandatory compression

## Execution Flow

### Step 0: Conflict State Pre-check

- `git diff --name-only --diff-filter=U` to detect unmerged files
- If PROGRESS.md is unmerged OR contains `<<<<<<<` markers:
    Output: "PROGRESS.md is in merge conflict state. Use /progress-merge to resolve, then re-run /progress-save."
    Halt (do not proceed)
- Otherwise: proceed

### Step 1: Collect Context

- Read `PROGRESS.md`
- `git status --short` — changed file list
- `git diff --stat` — change scope
- `git log -5 --oneline` — historical context
- Analyze changes to infer task progress

### Step 2: Intelligent Section Detection

Match content to existing sections. **Three-layer identification (fallback in order):**

1. **Layer 1 — emoji / exact alias match**: section heading contains the emoji or alias listed in the table below. Covers standard cases (~90%).
2. **Layer 2 — content semantic match**: if no name match, judge by the section's content contract (the "Belongs" column). If a heading's actual content matches a section's contract, treat it as that section — even if the heading name is non-standard (e.g. `## 🧪 手测清单` matches Unverified's contract).
3. **Layer 3 — unresolvable**: if Layers 1-2 can't uniquely identify (e.g. ambiguous between Paused and Unverified), ask user; do not guess.

**Hard rule**: semantic identification is for content processing only — **never rename an existing section**. Format Preservation Policy still applies. A `## 🧪 手测清单` section is processed as Unverified but keeps its original name.

### Section Detection & Contract Table

Unified identification + content rules. Each section's "Belongs" doubles as the Layer 2 semantic predicate.

| Section | Emoji / aliases | Belongs (Layer 2 semantic contract) | Does NOT belong |
|---------|-----------------|--------------------------------------|-----------------|
| `> Last updated` | `> Last updated` / `> 最后更新` | Timestamp header | — |
| 🎯 Current Focus | 🎯 / Current / 当前 | In-progress tasks + 1-2 sentence status; link spec/plan if available | Shipped retrospectives, archive links, multi-paragraph history |
| 📥 Next Phases | 📥 / Next Phases / Todo / 下一步 | Truly pending phases/tasks with spec/plan links | Status = ✅ / 📦 / archived / shipped (→ delete); ad-hoc TODOs |
| ⏸️ Paused Tasks | ⏸️ / Paused / 暂停 | Task name + 1-sentence pause reason + entry point + design link | Full design sketches, multi-paragraph "restart prerequisites" |
| ✅ Recently Completed | ✅ / Completed / 已完成 / Recently Completed | Task name + date + 1 sentence + archive link | Multiple releases merged into 1 line; >3 items (→ triggers archive) |
| 🧱 Blockers & Issues | 🧱 / Blockers / 问题 | Active blockers + status | Resolved (strikethrough / ✅, → delete; git log has record) |
| 🧠 Context Notes | 🧠 / Notes / 备注 | 1-3 sentence key findings/decisions/rationale, large content linked out | ASCII diagrams (→ ARCHITECTURE.md), full test stats, reference lists |
| ⚡ Quick Recovery | ⚡ / Recovery / 恢复 | ≤5 core commands with necessary comments | >5 line bash blocks; file `open:` lists |
| 🏛️ Archive Links | 🏛️ / Archive / 归档 | Links to archived task files | Archive content itself (lives in archive files) |
| 🔍 Unverified | 🔍 / Unverified / 待手测 / 待验证 / Manual Test Pending | Items in code-shipped + tests-pass + manual-test-pending gray zone | Already-✅ verified items; pure TODOs without shipped code |

If no section matches after Layer 2: append to most relevant section or ask user.

### Step 3: Update Content (Preserve Structure, Apply All Rules)

Update content while preserving the exact structure. **Every item must pass Pre-Write Self-Check + Section Detection & Contract Table + Size Budget + Brevity Guidelines.** Default to compression over completeness — details belong in L1/L2.

1. **Last updated time**: Update header timestamp if exists
2. **Completed tasks**: Add to appropriate completed section — concise per Brevity Guidelines
3. **In-progress**: Update status — link to spec/plan if available, otherwise concise summary
4. **Key files**: If "Recovery" section exists, add key file paths
5. **Context Notes**: Record key findings, decisions, recovery context per Brevity Guidelines. Link to docs for large content; keep concise summaries inline.

**Cleanup rules (mandatory — unconditional, not advisory):**

Every item is a hard trigger, not "compress if it has grown beyond…". Hit = change, no subjective judgment needed.

- **Run Pre-Write Self-Check per item** — any hit → change / delete / link out
- **Run Size Budget per section** — exceeding hard trigger → mandatory compression
- **Misplaced items** (archived / shipped in Next Phases or Current Focus, design sketches in Paused Tasks) → **move to correct section or delete**
- **Merge duplicates**
- **Do NOT add new sections unless explicitly needed**

### Step 4: Detect Archive Needs

After updating content, check three conditions (in priority order):

**Condition A — Major task completion (higher priority):**
- Major task header contains ✅ or "已完成"
- All phases/subtasks under it marked completed
- Task is in "Recently Completed" (not "Current Focus")

→ Prompt: "Task '[Name]' appears complete. Archive it with `/progress-archive`?"
  (If PROGRESS.md > 300 lines OR Recently Completed > 5 items, append: "Also: PROGRESS.md is getting long — consider trimming after archiving.")

**Condition B — Bloat without completed major task:**
- No completed major task, BUT PROGRESS.md > 300 lines OR Recently Completed > 5 items

→ Output (non-blocking): "PROGRESS.md 已有 N 行 / Recently Completed 有 N 条。Run `/progress-archive` and say '清理' or 'trim' to archive overflow completed items."

**Condition C — Unverified / 待手测 table bloat:**
- Detect Unverified / 待手测 table (section names: "Unverified", "待手测", "TTY 待手测", "Manual Test Pending", etc.)
- Count ✅ verified items in Unverified → if > 3: trigger
- Count total Unverified items → if > 10: trigger

→ Output (non-blocking): "Unverified table has N items (M already ✅ verified). Run `/progress-archive` with 'verify-cleanup' or '待手测清理' to triage — verified items don't belong in Unverified anymore."

### Step 5: Write to Disk

Write updated content preserving exact structure and formatting style.

### Step 6: Lint Pass (read-only, post-write)

After writing, run a read-only lint pass over the updated PROGRESS.md. **Only output warnings — never modify content.** User decides whether to act.

Output format (compact, one line per warning; skip rules with no warnings):

```
🔍 Lint warnings:
- [Rule 2] Phase "<name>" plan last updated YYYY-MM-DD (>30d), review if still accurate
- [Rule 3] Blockers/Paused reference "Phase X" which is no longer in Next Phases or Recently Completed — consider updating
- [Rule 4] PROGRESS.md is N lines (>500) — strongly consider /progress-archive trim
```

If no warnings: output "🔍 Lint: clean."

**Rule 2 — Stale Next Phases:**
- Condition: A line under `📥 Next Phases` (or equivalent) references a plan/spec file via path or link, AND that file's mtime > 30 days old
- Fallback (if no file link): if the Phase line itself contains a date (e.g., "(planned 2026-05-15)") older than 30 days
- Final fallback: if PROGRESS.md's own `> Last updated` header > 30 days old AND Next Phases is non-empty
- Action: report Phase name + last-updated date, prompt review

**Rule 3 — Cross-reference consistency (cheap version):**
- Condition: `⏸️ Paused Tasks` or `🧱 Blockers` section references a "Phase X" / "Phase Y" identifier (by name or number) that does NOT appear in `📥 Next Phases`, `✅ Recently Completed`, or `🎯 Current Focus`
- Action: report the dangling reference, prompt user to update or remove
- Note: this catches "Blockers mention Phase B-E but those phases were archived/trimmed" scenarios

**Rule 4 — File length:**
- Condition: PROGRESS.md > 500 lines
- Action: report line count, strongly suggest `/progress-archive` (stricter than Step 4 Condition B's 300-line advisory — Step 4 prompts at write time; Rule 4 persists every save until addressed)

**Implementation notes:**
- All checks are pure reads on the just-written file (plus optional `stat` for Rule 2 file mtimes)
- No content modification — user decides whether to act
- Warnings are idempotent across saves (same state → same warnings); accept this to avoid drift
- Rule 1 (Unverified bloat) is not needed here — Step 4 Condition C already covers it at write time
- If Rule 4 and Step 4 Condition B both fire, that's fine — Step 4 prompts action at write time, Step 6 reinforces on next save if unaddressed

## Large File Reading (Anti-Thrashing Policy)

**Problem**: Reading a long PROGRESS.md in a single Read call fills the context window and triggers autocompact thrashing ("Autocompact is thrashing: context refilled to limit within 3 turns").

**Trigger**: PROGRESS.md > 300 lines.

**Policy — read in segments, not whole-file**:
1. **First pass**: `Read(offset: 1, limit: 50)` — get frontmatter + section headings to understand layout.
2. **Second pass**: For each section that needs updating, `Read(offset: <section_start>, limit: <section_length>)` — only the relevant lines.
3. **Skip unchanged sections**: Sections not touched by this save don't need to be read at all.
4. **Prefer Edit over Write for large files**: After reading the sections you need, use `Edit` tool for targeted changes. Only use `Write` (full file rewrite) when changes span most of the file.
5. **Hard rule**: Never call `Read` on a > 300-line PROGRESS.md without `offset`/`limit`. A single full-file Read is the most common cause of autocompact thrashing.

## Reference Standard Format (Optional)

This standard format is provided as **reference only**. Projects can adopt it if they find it useful, but it's never enforced:

```markdown
# Progress

> Last updated: {CURRENT_DATE}

## 🎯 Current Focus
<!-- Task name — status (1-2 sentences). spec: [link] plan: [link] (multiple OK) -->

## 📥 Next Phases
<!-- | Phase | Goal | spec | plan | Status | -->

## ⏸️ Paused Tasks
<!-- Task name — pause reason (1 sentence). Entry: `git checkout <branch>`. Design: [link] -->

## ✅ Recently Completed Phases
<!-- Task name (date) — 1-sentence summary. [archive link] -->

## 🧱 Blockers & Issues
<!-- Problem — status -->

## 🧠 Context Notes
<!-- Key findings, decision rationale, experiment records, recovery context. Each item 1-3 sentences. Large content linked to docs -->

## ⚡ Quick Recovery
<!-- Commands first; add ≤1 line comment if needed -->
- `git pull`
- `bun run dev`

## 🏛️ Archive Links
<!-- [version — task name (date)](archive-link) -->
```

## Trigger Timing

- Before `git commit`: Update progress before creating commit
- Before `git stash`: Save current state before stashing
- Before `git checkout`: Update before switching branches
- Manual: User explicitly requests progress update

See README for recommended AGENTS.md configuration to let LLM trigger this skill automatically.
