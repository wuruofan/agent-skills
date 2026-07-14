---
name: progress-save
description: Use when user asks to update, save, record, or write to PROGRESS.md (更新进度, 保存进度, 记录进度, 更新 PROGRESS.md), or before git commit/stash/checkout/PR/push, or at end of work session
version: 1.5.0
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

### Section Contract

Each section defines both what belongs in it and what doesn't. The core L0 constraint — when a section's name doesn't match its actual function, signals get polluted immediately.

| Section | Belongs | Does NOT belong |
|---------|---------|-----------------|
| 🎯 Current Focus | In-progress tasks + 1-2 sentence status; link spec/plan if available | Shipped retrospectives, archive links (→ Archive Links), multi-paragraph historical changes |
| 📥 Next Phases | Truly pending phases/tasks with spec/plan links | Status = ✅ / 📦 / archived / shipped (→ delete); ad-hoc TODOs like "I'll do X later" (should go in spec first) |
| ⏸️ Paused Tasks | Task name + 1-sentence pause reason + entry point + design link | Full design sketches, multi-paragraph "restart prerequisites", alternative comparisons |
| ✅ Recently Completed | Task name + date + 1 sentence + archive link | Multiple releases merged into one line; >3 items (→ triggers archive) |
| 🧱 Blockers & Issues | Active blockers + status | Resolved (strikethrough / ✅, → delete; git log has record) |
| 🧠 Context Notes | 1-3 sentence key findings/decisions/rationale, large content linked out | ASCII architecture diagrams (→ ARCHITECTURE.md), full test statistics (→ CI dashboard), reference project lists (→ README) |
| ⚡ Quick Recovery | ≤5 core commands with necessary comments | >5 line bash blocks (→ README/CLAUDE.md); file `open:` lists (→ IDE recents) |

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

### Step 0: Conflict State Pre-check (NEW)

- Execute `git diff --name-only --diff-filter=U` to detect unmerged files
- If PROGRESS.md is in unmerged state OR contains `<<<<<<<` markers:
    Output: "PROGRESS.md is in merge conflict state. Use /progress-merge
             to resolve, then re-run /progress-save."
    Halt execution (do not proceed to Step 1-5)
- Otherwise: proceed to Step 1

### Step 1: Collect Context

- Read current `PROGRESS.md` content.
- Execute `git status --short` to get changed file list.
- Execute `git diff --stat` to understand change scope.
- Execute `git log -5 --oneline` to learn historical context.
- Analyze changes to infer task progress.

### Step 2: Intelligent Section Detection

Based on existing PROGRESS.md structure, find appropriate sections for updates:

| Content Type | Possible Section Names |
|--------------|------------------------|
| Last updated time | `> Last updated`, `> 最后更新` |
| Current focus | `🎯`, `Current Focus`, `当前`, `Current` |
| Next steps | `📥`, `Next Phases`, `Todo`, `下一步`, `Next` |
| Paused tasks | `⏸️`, `Paused Tasks`, `暂停`, `Hold` |
| Completed | `✅`, `Completed`, `已完成`, `Recently Completed` |
| Blockers | `🧱`, `Blockers`, `Issues`, `问题` |
| Notes | `🧠`, `Context Notes`, `Notes`, `备注`, `Context` |
| Recovery | `⚡`, `Quick Recovery`, `恢复`, `Recovery` |
| Archive Links | `🏛️`, `Archive Links`, `归档`, `Archive` |

**Matching logic:**
1. Search for section names matching content type patterns
2. If found, update within that section
3. If not found, append to most relevant existing section or ask user

### Step 3: Update Content (Preserve Structure, Apply All Rules)

Update content while preserving the exact structure. **Every item must pass Pre-Write Self-Check + Section Contract + Size Budget + Brevity Guidelines.** Default to compression over completeness — details belong in L1/L2.

1. **Last updated time**: Update header timestamp if exists.
2. **Completed tasks**: Add to appropriate completed section — concise per Brevity Guidelines.
3. **In-progress**: Update status — link to spec/plan if available, otherwise concise summary.
4. **Key files**: If there's a "恢复" or "Recovery" section, add key file paths.
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
- Major task section header contains ✅ or "已完成" marker
- All phases/subtasks under this task are marked completed
- Task is in "Recently Completed" section (not "Current Focus")

**If A detected:**
Prompt user: "Task '[Task Name]' appears complete. Archive it with `/progress-archive`?"
- If PROGRESS.md also > 300 lines or Recently Completed > 5 items, append: "Also: PROGRESS.md is getting long — consider trimming after archiving."

**Condition B — Bloat without completed major task:**
- No completed major task detected, BUT:
- PROGRESS.md > 300 lines, OR Recently Completed > 5 items

**If B detected:**
Output (non-blocking): "PROGRESS.md 已有 N 行 / Recently Completed 有 N 条。Run `/progress-archive` and say '清理' or 'trim' to archive overflow completed items."

**Condition C — Unverified / 待手测 table bloat:**
- Detect Unverified / 待手测 table in PROGRESS.md (section names: "Unverified", "待手测", "TTY 待手测", "Manual Test Pending", etc.)
- Count ✅ verified items in Unverified → if > 3: trigger
- Count total Unverified items → if > 10: trigger

**If C detected:**
Output (non-blocking): "Unverified table has N items (M already ✅ verified). Run `/progress-archive` with 'verify-cleanup' or '待手测清理' to triage — verified items don't belong in Unverified anymore."

**Example detection:**
```
## ✅ TS Runtime Rewrite Phase 1-4
### Phase 1: ✅
### Phase 2: ✅
### Phase 4.1: ✅
...
→ All phases complete → Prompt archive suggestion (Condition A)

## Unverified / 待手测
| # | Item | Status |
| 2 | chat.send timeout | ✅ verified |
| 3 | streaming output | ✅ verified |
| ... (12 more, 9 are ✅) |
→ 9 ✅ items in Unverified (>3) AND 15 total (>10) → Prompt verify-cleanup suggestion (Condition C)
```

### Step 5: Write to Disk

Write updated content preserving exact structure and formatting style.

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

**Section Purpose:**

| Section | Purpose | Used By |
|---------|---------|---------|
| `🎯 Current Focus` | In-progress tasks | restore, save, summary |
| `📥 Next Phases` | Planned phases | restore, save, summary |
| `⏸️ Paused Tasks` | Paused mid-way tasks | restore, save (track paused state) |
| `✅ Recently Completed Phases` | Completed phases (max 2-3) | save, archive (detects completion) |
| `🧱 Blockers & Issues` | Active blockers | restore (quick reference) |
| `🧠 Context Notes` | Key findings, decisions, recovery context (concise; link large content) | restore (context recall) |
| `⚡ Quick Recovery` | Restore commands | restore |
| `🏛️ Archive Links` | Archived major tasks | archive (updates links) |

## Difference from Other Skills

| Feature | progress-save | progress-archive | progress-summary | progress-merge |
|---------|---------------|------------------|------------------|----------------|
| Update PROGRESS.md | ✓ | ✓ (remove completed) | ✗ | ✓ (merge conflicts) |
| Create archive files | ✗ | ✓ | ✗ | ✗ |
| Git operations | ✗ | Optional commit | ✗ | Optional `git add` |
| Generate summary | ✗ | ✗ | ✓ | ✗ |
| Trigger detection | Archive suggestion (Mode A/B/**C**), **Merge redirect** | N/A | N/A | N/A |
| Resolves PROGRESS.md conflicts | ✗ | ✗ | ✗ | ✓ |

**Trigger timing:**
- Before `git commit`: Update progress before creating commit
- Before `git stash`: Save current state before stashing
- Before `git checkout`: Update before switching branches
- Manual: User explicitly requests progress update

See README for recommended AGENTS.md configuration to let LLM trigger this skill automatically.