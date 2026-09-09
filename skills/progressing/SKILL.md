---
name: progressing
description: Use as a git-backed todo that survives sessions. New session / switched machine / user says "接着干 / 继续 / 之前干到哪了 / 恢复进度 / 加载进度" → loads PROGRESS.md, reconciles against git, surfaces stale entries. Mid-session work that satisfies a `Done when:` → closes the entry in-place. Session interrupted mid-way (network drop, switching machines, explicit handoff, context compaction before state must survive) → writes unfinished state. NOT a per-commit ritual. PROGRESS.md is a single file in the project root.
version: 2.0.0
---

# Progressing

A todo with three superpowers a plain todo doesn't have:

1. **Closure conditions are explicit** — every entry carries `Done when:` / `Awaiting:` / `Restart:` so the definition of "done" is mechanical, not a human judgement call.
2. **Git auto-reconciles** — when you load, cheap closure checks run against git (commit landed? branch merged? file exists?) and matching entries are removed without you asking.
3. **Stale entries die on read** — every entry carries a birth date `[YYYY-MM-DD]`; entries older than `STALE_DAYS` (default 7) are surfaced for review on every load.

Without any of these three, this skill is just a worse todo. Keep all three or kill the skill.

## The Closure Test

Before writing any entry, answer **"这条怎么关？"**:

| Closes when... | Section | Entry shape |
|---|---|---|
| a machine check passes | `## Open` | `- [YYYY-MM-DD] <task> — <1-line state>. Done when: <check>` |
| the user confirms | `## Verify` | `- [YYYY-MM-DD] <task> — <shipped evidence>. Awaiting: <user check>` |
| nothing closes it | `## Paused` | `- <task> — <1-line reason>. Restart: <link>` |

Missing suffix = invalid entry. No closure? → `## Paused`, that's the honest admission.

## File Format

```markdown
# Progress

> Last updated: 2026-09-09

## Open
- [2026-09-09] compact live/resume display — L4 run interrupted. Done when: `bun run test:l4` exits 0

## Verify
- [2026-09-04] /resume rendering after compact — commit `f00dbabe`. Awaiting: user TTY check

## Paused
- Tool/Thinking display — blocked until v0.7.1. Restart: docs/specs/tool-display.md
```

Limits: entry ≤ 2 lines; `## Paused` ≤ 10; file ≤ 60 lines; omit empty sections.

## Behavior

The skill picks the right action from context — no mode flag needed in 90% of cases. Defaults below; user can force with `/progressing save` or `/progressing load`.

### Load (default when context suggests resumption)

Triggers: new session's first message hints at this project / user says "接着干 / 继续 / 之前干到哪了" / switched branches or machines / user asks "现在在干嘛".

1. Read PROGRESS.md. Missing → if `git status` shows WIP, create one; else say "nothing in progress".
2. Read `git log --oneline -10`, `git status --short`, current branch.
3. Reconcile:
   - `## Open` — run each `Done when:` cheaply (commit / merge / file / grep); closure met → delete. Expensive (test suite, build) → ask the user.
   - `## Verify` — batch-ask the user one question listing all pending entries.
   - Stale sweep — entries older than `STALE_DAYS` (default 7) that survived above: ask continue / abandon / extend. Continue → re-stamp today's date.
   - `## Paused` — leave alone unless > 10.
4. Write back via targeted Edit. Skip writing if Open + Verify are both empty.
5. Output a recovery report ending with a suggested next step.

### Close (auto, mid-session)

When work in this session satisfies an existing `Done when:` / `Awaiting:`, immediately delete that entry and refresh `> Last updated`. This is a low-cost micro-action, not a full save. Run it whenever the closure fires — don't wait for an interruption.

### Save (explicit interruption)

Triggers: user says "中断 / 切机器 / 保存进度 / 交接 / context 压缩前" / network dropping.

1. `git diff --name-only --diff-filter=U` — if PROGRESS.md is unmerged, redirect to `progress-merge` first.
2. Read PROGRESS.md in full.
3. Drop entries this session finished; re-stamp carried-over Open entries with today's date.
4. Add new unfinished items via targeted Edit.
5. Update `> Last updated`.
6. Self-check: if only the timestamp changed, save failed — say so. If Open + Verify would both be empty, skip writing.

## Global Rules

PROGRESS.md lives in the project root (single file, no external archive, no per-event snapshots — git already syncs it across devices). Find root by upward search for `.git`. File language: user input > commit history > locale. Default `STALE_DAYS=7`; user can override per call.

## Trigger Timing

| Context | Action |
|---|---|
| New session / new device / "接着干 / 继续 / 之前干到哪了" | Load |
| Mid-session, a `Done when:` is now satisfied | Close (auto) |
| Network dropping / switching machines / explicit handoff / pre-compaction with unfinished state | Save |
| Routine commit with no unfinished work | Do nothing |
| `git merge` / `rebase` / `cherry-pick` touches PROGRESS.md | Delegate to `progress-merge` |
