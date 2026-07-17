---
name: progress-merge
description: Use when PROGRESS.md has merge conflicts (after `git merge` / `git rebase` / `git cherry-pick` involving PROGRESS.md), or when user wants to compare/merge progress state between two branches (合并 PROGRESS 冲突, merge progress across branches, 两个分支的 PROGRESS 怎么合, PROGRESS conflict 怎么解决, 解决 progress 冲突, dry-run compare branches)
version: 1.2.0
---

# Progress Merge

Merge PROGRESS.md across two git branches semantically. Use during merge conflicts on PROGRESS.md, or for comparing progress state between branches.

## Global Rules

Find project root (upward to `.git`/`PROGRESS.md`); target `PROGRESS.md` in root; language follows user input > commit history > locale (en/zh); preserve existing format, never restructure either side's section naming style; output follows ours (or user-chosen side). If PROGRESS.md does not exist on either side: do NOT initialize (merge is not the right place for that) — both sides missing → exit and direct user to `/progress-save`; one side missing → propose degenerate merge (take the other side as-is), user confirms before write. Sequencing Awareness: see dedicated section below.

## Execution Modes

### Mode A: Conflict Resolution

Triggered by:
- explicit `/progress-merge` while git index has PROGRESS.md as `UU`
- `/progress-save` sentinel redirect

Input: `git show :2:PROGRESS.md` + `git show :3:PROGRESS.md`.
**CRITICAL — stage mapping depends on git mode**: in `git rebase` the stage 2/3 semantics are REVERSED vs `git merge`. Always run Step 0 (detect mode via `<GITDIR>/MERGE_HEAD` / `rebase-merge/` / `CHERRY_PICK_HEAD`) FIRST to determine which stage is "ours" and which is "theirs".

| Trigger | `:2:` = | `:3:` = | User's "ours" | User's "theirs" |
|---|---|---|---|---|
| `git merge X` | HEAD | X | `:2:` | `:3:` |
| `git rebase X` | X (base) | feature | **`:3:`** (reversed!) | **`:2:`** (reversed!) |
| `git cherry-pick X` | HEAD | X commit | `:2:` | `:3:` (narrow scope) |

### Mode C: Compare (Dry-run)

Triggered by:
- `/progress-merge --compare <ref1> <ref2>`
- `/progress-merge --compare <path1> <path2>` (non-git inputs)

Input: read from two refs or two file paths.

## Execution Flow

### Step 0: Mode Detection + Sequencing Check (Mode A only)

**Step 0a**: Get real git directory (worktree compat):
```
git rev-parse --git-dir
```
Store result as `<GITDIR>`. Use `<GITDIR>/...` instead of `.git/...` in all subsequent commands.

**Step 0b**: Detect git mode (run sequentially, first match wins):

| # | Command | If exit code 0 | Mode |
|---|---|---|---|
| 1 | `test -d <GITDIR>/rebase-merge` | rebase (interactive/merge) | rebase |
| 2 | `test -d <GITDIR>/rebase-apply` | rebase (apply) | rebase |
| 3 | `test -f <GITDIR>/CHERRY_PICK_HEAD` | cherry-pick | cherry-pick |
| 4 | `test -f <GITDIR>/MERGE_HEAD` | merge | merge |
| 5 | all failed | not in conflict | → Error Handling: "Not in unmerged state" |

**Step 0c**: Sequencing check — if other unmerged files exist besides PROGRESS.md, surface warning (see Sequencing Awareness Layer 2).

### Step 1: Read Both Sides

Read ours and theirs versions per Step 0's mode mapping. Also record working tree PROGRESS.md content hash for concurrency check in Step 7.

### Step 2: Parse (Section + Task granularity)

Identify sections using the detection table below, then parse tasks within each section.

### Step 3: Collect Git Context

```bash
# Core: relative diffs
git log --oneline -20 OURS..THEIRS
git log --oneline -20 THEIRS..OURS

# Fork point + time anchors
git merge-base OURS THEIRS
git log -1 --format='%H %ai %s' OURS
git log -1 --format='%H %ai %s' THEIRS
```

OURS/THEIRS are branch refs (not stage numbers). Map per mode:

| Mode | `OURS` ref | `THEIRS` ref |
|---|---|---|
| merge | `git symbolic-ref --short HEAD` | `cat <GITDIR>/MERGE_HEAD` |
| rebase | `cat <GITDIR>/rebase-merge/head-name` | `cat <GITDIR>/rebase-merge/onto` |
| cherry-pick | `git symbolic-ref --short HEAD` | `cat <GITDIR>/CHERRY_PICK_HEAD` |

For display: convert full refs to short names via `git symbolic-ref --short <ref>`.

### Step 4: Apply Merge Strategy Table

### Step 5: Heuristic Arbitration (Batched AskUserQuestion)

### Step 6: Preview

### Step 7: Write + Optional `git add` (Mode A only)

Before writing, re-read working tree PROGRESS.md and compare hash with Step 1. If changed externally, abort and prompt user to re-run.

## Merge Strategy Table

| Section | Default Strategy | Notes |
|---|---|---|
| `> Last updated` | **max** | Take newer time |
| `🎯 Current Focus` | **Ask user** | Options: keep ours / theirs / both (annotate source) / rewrite |
| `📥 Next Phases` | **union + task dedup** | Uncertain tasks → keep both, don't ask (plans are low-stakes) |
| `⏸️ Paused Tasks` | **union** | Paused tasks should survive across branches |
| `✅ Recently Completed` | **union + time sort + cap N** | N = `max(ours count, theirs count, 3)`; truncate oldest if over N |
| `🧱 Blockers & Issues` | **union + dedup** | Never lose a blocker |
| `🧠 Context Notes` | **union (append)** | Notes: more is better |
| `⚡ Quick Recovery` | **union + annotate source** | Theirs items marked `[from <branch>]` |
| `🏛️ Archive Links` | **union** | Pure links, safest |
| `🔍 Unverified / 待手测` | **union + status-priority dedup** | Gray-zone section (code shipped + tests pass + manual test pending). Same-task items: take higher verify-state (✅ verified > shipped-won't-verify > ⏳ pending). Never downgrade a verified item. If union produces > 10 items or > 3 ✅ items → suggest `/progress-archive` Mode C (verify-cleanup) after merge completes. |

## Section Detection Table

Unified identification + content rules. **Three-layer identification:** (1) emoji/alias match, (2) content semantic match — if no name match, judge by the "Belongs" contract column, (3) unresolvable → treat as custom section, ask user during arbitration. Never rename existing sections on either side.

| Section | Emoji / aliases | Belongs (Layer 2 semantic contract) |
|---------|-----------------|--------------------------------------|
| `> Last updated` | `> Last updated` / `> 最后更新` | Timestamp header |
| 🎯 Current Focus | 🎯 / Current / 当前 | In-progress tasks + 1-2 sentence status |
| 📥 Next Phases | 📥 / Next Phases / Todo / 下一步 | Truly pending phases/tasks with spec/plan links |
| ⏸️ Paused Tasks | ⏸️ / Paused / 暂停 | Task name + 1-sentence pause reason + entry point |
| ✅ Recently Completed | ✅ / Completed / 已完成 / Recently Completed | Task name + date + 1 sentence + archive link |
| 🧱 Blockers & Issues | 🧱 / Blockers / 问题 | Active blockers + status |
| 🧠 Context Notes | 🧠 / Notes / 备注 | 1-3 sentence key findings/decisions/rationale |
| ⚡ Quick Recovery | ⚡ / Recovery / 恢复 | ≤5 core commands with necessary comments |
| 🏛️ Archive Links | 🏛️ / Archive / 归档 | Links to archived task files |
| 🔍 Unverified | 🔍 / Unverified / 待手测 / 待验证 / Manual Test Pending | Items in code-shipped + tests-pass + manual-test-pending gray zone |

If both sides use different section naming styles → add to "must-ask" queue.

## Task-Level Merge Rules

### Task Parsing Heuristics

| Format | Detection Rule | Example |
|---|---|---|
| Checkbox | `- [ ]` / `- [x]` / `* [ ]` / `* [x]` | `- [x] Implement OAuth` |
| Bullet | `-` / `*` / `+` (no checkbox) | `- Fix trigger words` |
| Sub-heading | `###` / `####` with `:` or status marker | `### Phase 4.1: Tools Module ✅` |
| Numbered | `1.` / `2.` | `1. Rewrite runtime` |

**Sub-heading body** = heading line → next same-level heading / higher-level heading / `---` / EOF. Entire body merges as one unit (no nested decomposition).

**Task attributes**: Title (strip markers/checkboxes/emojis) · Status (`[x]`/`✅`=done, `[ ]`=pending, `⏸️`=paused, `(WIP)`=in_progress, `📦`=shipped-待验证) · Sub-items (indented bullets) · Description (non-structural text after title)

### "Same task?" Judgment (3 tiers)

1. **Clearly same** (normalized identical or semantically equivalent) → auto-merge
2. **Uncertain** (semantically similar but ambiguous) → must-ask queue
3. **Clearly different** (semantically unrelated) → keep both

**Hard constraint**: LLM MUST push uncertain cases to must-ask queue, never decide alone.

**Exception**: In `Next Phases`, uncertain tasks default to "keep both" without asking — plans are low-stakes.

### Same-task Content Arbitration

| Field | Rule |
|---|---|
| Status | Take higher completion; see Status Completion Order below for gray zones (done > shipped-待验证 > in_progress > pending; paused is gray) |
| Description | If semantically different → must-ask queue; if highly similar → take longer |
| Sub-items | union + dedup |
| Timestamp | max |

### Status Completion Order

**Clear**: `pending < in_progress < ✅ done`

**Gray zone 1 — `⏸️ paused` vs `in_progress`/`pending`**: paused may mean "80% done then paused" or "barely started then paused":
- If paused item has progress description → LLM judges from description
- If cannot judge from description → must-ask queue
- `✅ done` always wins over paused

**Gray zone 2 — `📦 shipped-待验证` vs `✅ done` / `in_progress`** (motelet scenario: code shipped + tests pass + manual/TTY test pending):
- `📦 shipped-待验证` sits between `in_progress` and `✅ done` — code is merged/shipped but verification not complete
- `✅ done` (fully verified) always wins over `📦 shipped-待验证` — never downgrade a verified item
- `📦 shipped-待验证` wins over `in_progress` — shipped is closer to done
- Both sides `📦 shipped-待验证` → keep as `📦 shipped-待验证` (do NOT auto-promote to ✅; verification status must be preserved for tracking)
- One side `📦 shipped-待验证`, other side `⏸️ paused` → must-ask queue (shipped vs paused is genuinely ambiguous: shipped may be "shipped then deprioritized for verification" while paused may be "paused before shipping")
- Detection markers for `📦 shipped-待验证`: "代码已 ship", "shipped", "tests pass", "测试通过", "TTY 待手测", "manual test pending", "待手测", "待验证", absence of ✅ on a shipped item

## Heuristic Arbitration & Recommendation

### Must-ask Scenarios

1. Current Focus both non-empty and different
2. Same-name task status conflict that cannot auto-resolve (including paused gray zone)
3. Task name LLM judges as "uncertain if same" (except in Next Phases)
4. Section naming style difference

### Batching Rules

- ≤ 4 must-ask scenarios → one AskUserQuestion call
- > 4 → batch by priority (Current Focus first), ≤ 4 per batch
  - Header format: `X/Y·<Type>` if batches ≤ 9; `X/Y` if > 9
  - Type abbreviations: `CF` / `Task` / `Name` / `Sect`
- No ambiguity → skip questions, go straight to preview

### Recommendation Design

- First option = recommended, append `(Recommended)` to label
- Each option's `description` = one-line reasoning based on git context + status priority

### Task Keyword Extraction (for git log grep)

1. Remove status markers (✅ / ⏸️ / `(WIP)` etc.)
2. Remove leading sequence numbers (`Phase 4.1:`, `1.`, `- [ ]`)
3. Remove emojis
4. **Prefer English/code tokens**: camelCase, hyphenated, underscored, plain English words
5. If ≥ 1 English/code token → take first 1–2 as keyword
6. If 0 English/code tokens (pure Chinese) → take first 1–2 Chinese words
7. Fallback: if keyword grep returns empty → re-search with full task name

## Large File Budget

If either side > 500 lines:
- **Core sections** (Current Focus / Next Phases / Paused Tasks / Recently Completed / **Blockers** / **Unverified**) → full task-level parsing
- **Secondary sections** (Context Notes / Quick Recovery / Archive Links) → section-level union only
- If > 1000 lines → suggest `/progress-archive` first (non-blocking)

## Large File Reading (Anti-Thrashing Policy)

**Problem**: Reading a long PROGRESS.md (or both sides of a merge) in a single Read call fills the context window. After a few tool calls, the session hits autocompact, which thrashes — the context refills to the limit within 2-3 turns, repeatedly. This is the "Autocompact is thrashing" warning.

**Trigger**: Any side > 300 lines (the same threshold used for preview truncation). At 300 lines × ~10 tokens/line ≈ 3K tokens per side, reading both sides + git context + arbitration quickly approaches context limits.

**Policy — read in segments, not whole-file**:

1. **First pass — structural read**: Read only the first 50 lines (frontmatter + section headings) of each side using `Read` with `offset: 1, limit: 50`. This identifies section layout without loading full content.
2. **Second pass — targeted section reads**: For each section you need to parse (per Merge Strategy Table), read just that section's line range. Use the section heading line numbers from pass 1 to compute offset/limit.
   - Example: if `✅ Recently Completed` starts at line 120 and `🧱 Blockers` starts at line 180 → `Read(offset: 120, limit: 60)` to get just the Completed section.
3. **Skip sections that don't need task-level parsing**: For secondary sections under Large File Budget (Context Notes / Quick Recovery / Archive Links), do NOT read them segment-by-segment — they'll be section-level union'd from the structural read + a single bounded read if needed.
4. **Never read both sides simultaneously in full**: alternate — parse ours fully (in segments), then parse theirs (in segments). This halves the peak context usage.
5. **If context is already near limit** (e.g. resuming from a prior compact): skip pass 1, jump straight to reading only the sections mentioned in user's intent (e.g. if user said "merge the Unverified table", read only that section from both sides).

**Hard rule**: If you catch yourself about to call `Read` on a file > 300 lines WITHOUT offset/limit, stop. Use the segmented approach above. A single full-file Read of a 500+ line PROGRESS.md is the most common cause of autocompact thrashing in this skill.

**Applies to both Mode A and Mode C**: In Mode A, both `:2:` and `:3:` versions are subject to this policy. In Mode C, both ref/path inputs are subject to it. For Mode A, you can use `git show :2:PROGRESS.md | head -50` via RunCommand as the structural read (avoids Read tool on a git-index blob that isn't a real file path).

## Sequencing Awareness

**Layer 1 (Constraint on AI)**: When AI executes `git merge` / `git rebase` / `git cherry-pick`, follow AGENTS.md sequencing rule: resolve non-PROGRESS.md conflicts FIRST, then call /progress-merge.

**Layer 2 (Pre-check, addressed to user)**: If /progress-merge is invoked while other unmerged files exist, surface a warning via AskUserQuestion. AI presents "pause and resolve code first" as recommended option, but user makes final decision.

**Layer 3 (Cautionary note, addressed to user)**: If user opts to continue, embed a cautionary note in the preview so user knows the merged PROGRESS.md is based on a partially-resolved state.

## Interaction Patterns

### Preview

Show (a) change summary + (b) full PROGRESS.md preview.

If merged result > 300 lines → show section-level diff summary only (not full file).

### Confirm (Mode A)

Single AskUserQuestion with 4 options:
1. Write + git add (Recommended)
2. Write only (keep unmerged)
3. Revise (back to preview, max 3 times)
4. Cancel

On 4th revision attempt, option 3 disappears.

**Concurrency check**: Before writing, re-read working tree PROGRESS.md and compare with Step 1 hash. If changed → abort, prompt re-run.

### Mode C Close

No write questions. Output guidance for next steps (use Mode A for actual merge, etc.)

## Error Handling

| Error | Handling |
|---|---|
| PROGRESS.md missing on one side | Ask "take the other side (degenerate merge)?" |
| Both sides missing | Exit, direct to `/progress-save` |
| Not in git repo (except dry-run with absolute paths) | Error and exit |
| Not in unmerged state but user calls `/progress-merge` | Enter "compare" mode: ask user for both sources |
| In unmerged state but PROGRESS.md not in conflict list | Skip, tell user nothing to do |
| Parse failure (non-markdown / blank / binary) | Error and exit, suggest manual fix |
| User selects "Other" free text in batch questions | Insert raw content into corresponding section |
| User selects "Cancel" after preview | No write, git state unchanged |
| `git add` fails (permissions/lock) | File written, tell user manual `git add` |
| Other unmerged files exist | Sequencing Awareness Layer 2 flow: soft warn + user decides |
| `<<<<<<<` markers but git not in unmerged state | Offer: ① manual cleanup ② Mode C compare ③ cancel |

## Difference from Other Skills

| Feature | save | restore | archive | summary | **merge** |
|---|---|---|---|---|---|
| Writes to disk | ✓ | ✗ | ✓ | ✗ | ✓ |
| Git read depth | shallow | medium | ✗ | shallow | **deepest** |
| Executes git changes | ✗ | ✗ | optional commit | ✗ | **optional `git add`** |
| Sentinel redirect | → archive, **→ merge** | ✗ | ✗ | ✗ | ✗ |
| Resolves PROGRESS.md conflicts | ✗ | ✗ | ✗ | ✗ | ✓ (exclusive) |
