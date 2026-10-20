---
name: progressing
description: "Git-backed todo that survives sessions, in one PROGRESS.md at the project root (one line per entry). Four actions, routed by context — pick the right one automatically, or force one with `/progressing load`, `/progressing close`, `/progressing save`, `/progressing repair`. LOAD — new session / switched machine / '接着干 / 继续 / 之前干到哪了 / 恢复进度 / 加载进度 / 现在在干嘛 / 换机器 / 切换机器 / 新会话': read PROGRESS.md, reconcile against git, surface stale entries, report next step. CLOSE — mid-session an entry's `Done when:` / `Awaiting:` is now satisfied / '这条完成了 / 条件满足了 / 可以关掉了': delete that entry in place, refresh the date. SAVE — interrupted mid-way / '中断 / 切机器 / 保存进度 / 交接 / context 压缩前' / network dropping: write unfinished state. REPAIR — PROGRESS.md failed the format gate / '进度太长 / 台账太乱 / 修复进度文件 / 压缩进度': compress oversized entries to one line, move detail into its spec/plan file, link with `Detail:`. NOT a per-commit ritual — routine commits with no unfinished work do nothing."
version: 3.1.0
---

# Progressing

A todo with three superpowers a plain todo doesn't have:

1. **Closure conditions are explicit** — every entry carries `Done when:` / `Awaiting:` / `Restart:` so the definition of "done" is mechanical, not a human judgement call.
2. **Git auto-reconciles** — when you load, cheap closure checks run against git (commit landed? branch merged? file exists?) and matching entries are removed without you asking.
3. **Stale entries die on read** — every entry carries a birth date `[YYYY-MM-DD]`; entries older than `STALE_DAYS` (default 7) are surfaced for review on every load.

Without any of these three, this skill is just a worse todo. Keep all three or kill the skill.

A fourth property makes the first three survivable: **the file stays small.** PROGRESS.md is re-read in full on every load, so its cost is paid per session. One entry = one line, and the total is capped. Detail lives in the file it describes; the ledger only points at it.

## The Closure Test

Before writing any entry, answer **"这条怎么关？"**:

| Closes when... | Section | Entry shape |
|---|---|---|
| a machine check passes | `## Open` | `- [YYYY-MM-DD] <task> — <1-line state>. Done when: <check>` |
| the user confirms | `## Verify` | `- [YYYY-MM-DD] <task> — <shipped evidence>. Awaiting: <user check>` |
| nothing closes it | `## Paused` | `- [YYYY-MM-DD] <task> — <1-line reason>. Restart: <link>` |

Missing suffix = invalid entry. No closure? → `## Paused`, that's the honest admission.

## File Format

```markdown
# Progress

> Last updated: 2026-10-20

## Open
- [2026-10-20] eager terminal session — 落地完成（Task 1–5 done），待真机验证. Done when: `bun run test:gateway` 全绿. Detail: docs/specs/2026-10-13-eager-terminal-session.md

## Verify
- [2026-10-19] ink CR 列漂移修复 — 提交 `04139ea0`. Awaiting: user TTY check

## Paused
- [2026-09-01] Tool/Thinking display — 阻塞于 v0.7.1. Restart: docs/specs/tool-display.md
```

**The whole format, exhaustively:**

- One entry = **one line**. No continuation lines, no indented sub-blocks, no blank line inside an entry.
- The header is a **bare date** — `> Last updated: YYYY-MM-DD`. Never a summary, never a changelog. The ledger is current state, not a diary.
- Exactly three sections: `## Open`, `## Verify`, `## Paused`. Omit empty ones. No fourth section, no "附" appendix.
- Every entry ends with `Done when:` / `Awaiting:` / `Restart:`.
- `Detail: <path>` points at the file holding the detail (spec, plan, handoff). **The path must exist.** If no such file exists yet, write one first, then link it.
- An optional HTML comment block right under the title may carry the managed-by banner and the install command. The gate ignores it; markdown renderers hide it; `grep` finds it.

| Limit | Value |
|---|---|
| entry | 1 line, ≤ 256 chars |
| `## Open` | ≤ 15 entries |
| `## Verify` | ≤ 10 entries |
| `## Paused` | ≤ 10 entries |
| file | ≤ 60 lines, ≤ 12,000 chars |

## The Banner

A ledger a human may open should say who owns it and how to keep it honest. Put the managed-by marker in the **H1 title** (renderers show it) and the format note in a short `>` line right under the date (also rendered, and the gate allows it):

```markdown
# Progress · managed by `progressing` skill

> Last updated: 2026-10-20

> 一条目一行；详情在各自 spec/plan/debug 文件（见 `Detail:`）。手动编辑会破坏格式契约 —— 改完请跑格式闸门。
```

Both lines are optional — the ledger is valid without them. The gate allows a `>` line only if it is the date header or a short (≤ 200 chars) managed-by / format note; any other `>` line is rejected, so the header cannot drift back into a changelog.

**The install command belongs in the project's AGENTS.md, not in the ledger.** Reasoning: an agent without this skill never reads `PROGRESS.md`, so an install line there reaches nobody. `AGENTS.md` is read by every agent on every session — that is the only place the command can actually do its job.

```markdown
> **换机器 / 新环境**：本仓库的 `PROGRESS.md` 由 `progressing` skill 管理。若该 skill 未安装，先装：
> ```bash
> npx skills add wuruofan/agent-skills --skill progressing -g -y
> ```
```

Both audiences get what they need: the banner tells a human reading the file who owns it, `AGENTS.md` tells an agent how to get the tooling. Never put the install command in the ledger — it is a project-level instruction, and the ledger is state.


## The Gate

The limits above are not advice. One command decides pass/fail, and it must exit 0 before any write to PROGRESS.md stands.

```bash
python3 - <<'PY'
import os, re, sys
PATH, MAX_LINE, MAX_LINES, MAX_CHARS = "PROGRESS.md", 256, 60, 12000
CAPS = {"Open": 15, "Verify": 10, "Paused": 10}

def root():
    p = os.path.abspath(os.getcwd())
    while p != "/":
        if os.path.isdir(os.path.join(p, ".git")): return p
        p = os.path.dirname(p)
    return os.path.abspath(os.getcwd())

if not os.path.exists(PATH):
    print("no PROGRESS.md — nothing to check"); sys.exit(0)
raw = open(PATH, encoding="utf-8").read(); lines = raw.split("\n")
errs, counts, sec = [], {}, None
in_comment = False
for i, line in enumerate(lines, 1):
    s = line.strip()
    if not s: continue
    if s.startswith("<!--"):
        if in_comment: comment_end = i
        else: in_comment = True
        continue
    if in_comment:
        if s.endswith("-->"): in_comment = False
        continue
    if line[:1] in (" ", "\t"):
        errs.append(f"L{i}: indented line — one entry = one line; move detail into a file and link it"); continue
    if s.startswith("# "): continue
    if s.startswith("## "):
        sec = s[3:].strip()
        if sec not in CAPS: errs.append(f"L{i}: unknown section '## {sec}' — only {', '.join(sorted(CAPS))}")
        continue
    if s.startswith(">"):
        if re.fullmatch(r"> Last updated: \d{4}-\d{2}-\d{2}", s): continue
        if len(s) <= 200 and re.fullmatch(r"> (Managed by .*|.*由 .* skill 管理.*|.*一条目一行.*|.*格式契约.*|.*闸门.*)", s): continue
        errs.append(f"L{i}: '>' line must be the date header, a short managed-by banner, or a one-line format note")
        continue
    if s.startswith("- "):
        counts[sec] = counts.get(sec, 0) + 1
        if len(line) > MAX_LINE:
            errs.append(f"L{i}: {len(line)} chars > {MAX_LINE} — keep task+state+closure on one line, link detail")
        if not re.search(r"(Done when|Awaiting|Restart):", line):
            errs.append(f"L{i}: no closure suffix — need 'Done when:' / 'Awaiting:' / 'Restart:'")
        m = re.search(r"Detail: (\S+)", line)
        if m and not (os.path.exists(m.group(1)) or os.path.exists(os.path.join(root(), m.group(1)))):
            errs.append(f"L{i}: Detail: {m.group(1)} does not exist")
        continue
    errs.append(f"L{i}: stray line — one entry = one line; move detail into a file and link it")
for name, cap in CAPS.items():
    c = counts.get(name, 0)
    if c > cap: errs.append(f"## {name}: {c} entries > {cap} — close, confirm, or archive the stale ones")
if len(lines) > MAX_LINES: errs.append(f"file: {len(lines)} lines > {MAX_LINES}")
if len(raw) > MAX_CHARS: errs.append(f"file: {len(raw)} chars > {MAX_CHARS}")
if errs:
    print(f"PROGRESS.md violations ({len(errs)}):")
    for e in errs: print("  " + e)
    sys.exit(1)
print(f"PROGRESS.md clean — {len(raw)} chars, {sum(counts.values())} entries")
PY
```

Exit 0 = clean. Exit 1 = violations listed. **Never use `awk length`** — it counts bytes, so a legitimate Chinese entry (~130 bytes / 61 chars) gets falsely flagged.

**When it fires:** keep the task, the state, and the closure condition; move the detail into the file it describes (or create that file) and link it with `Detail:`. A section over cap means entries have piled up — close, confirm, or archive them, don't raise the cap. Long prose in a ledger is unreadable at a glance and gets re-read on every load.

**When to run it:** after **every** write to PROGRESS.md, no exceptions — Load, Close, Save, and any ad-hoc edit made mid-session. The gate is the only thing standing between the format and a 100KB file of prose; a gate that runs "usually" doesn't run.

## Behavior

The skill picks the right action from context — no mode flag needed in 90% of cases. Defaults below; user can force any of the four with `/progressing load`, `/progressing close`, `/progressing save`, `/progressing repair`.

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
5. Run the gate. If it fires, repair (see **Repair**) and re-run until clean.
6. Output a recovery report ending with a suggested next step.

### Close (auto, mid-session)

When work in this session satisfies an existing `Done when:` / `Awaiting:`, immediately delete that entry and refresh the header date. This is a low-cost micro-action, not a full save. Run it whenever the closure fires — don't wait for an interruption. Then run the gate: Close is the most frequent writer, so an ungated Close is how a ledger quietly grows past 100KB.

### Save (explicit interruption)

Triggers: user says "中断 / 切机器 / 保存进度 / 交接 / context 压缩前" / network dropping.

1. `git diff --name-only --diff-filter=U` — if PROGRESS.md is unmerged: resolve by union — strip conflict markers, keep BOTH sides' entries (drop exact duplicate lines only, never choose between versions), stamp the header date, `git add PROGRESS.md`, then continue. Never delete at merge time; ambiguous near-duplicates surface at the next Load's Verify pass.
2. Read PROGRESS.md in full.
3. Drop entries this session finished; re-stamp carried-over Open entries with today's date.
4. Add new unfinished items via targeted Edit — one line each, closure suffix, `Detail:` link if there's more to say.
5. Update the header date.
6. Self-check: if only the date changed, save failed — say so. If Open + Verify would both be empty, skip writing.
7. Run the gate. Oversized lines mean detail leaked in — move it to the file it describes, link it, re-run until clean.

### Repair (when the gate fails on an existing file)

A ledger written by an older version of this skill can be enormous. Repair is mechanical, not a rewrite:

1. Run the gate; group the violations by entry.
2. For each oversized entry, find the file the detail already lives in — a spec, plan, handoff, or debug note. **The detail is usually already there**; the ledger was duplicating it.
3. Rewrite the entry to one line: task + current state + closure condition + `Detail: <that file>`.
4. Detail that exists nowhere → write it to a new file (spec/plan/handoff, per the project's convention), then link it.
5. Sections over cap → close what is done, confirm what is pending, move the rest to `## Paused` with a `Restart:` link, or archive it out of the ledger entirely (git history already keeps it).
6. The header → bare date. Prose that was in it belongs in a commit message or a handoff note, not the ledger.
7. Re-run the gate until exit 0.

Repair is lossy by design in exactly one way: prose that lives nowhere else and has no home gets a home, or gets dropped. Never drop information you cannot re-derive — if it is only in the ledger, write it out first.

## Global Rules

PROGRESS.md lives in the project root (single file, no external archive, no per-event snapshots — git already syncs it across devices). Find root by upward search for `.git`. File language: user input > commit history > locale. Default `STALE_DAYS=7`; user can override per call.

Detail files may live anywhere the project already uses (usually `docs/specs/`, `docs/plans/`). Reference them by repo-relative path — an absolute path breaks on the next machine.

## Trigger Timing

| Context | Action |
|---|---|
| New session / new device / "接着干 / 继续 / 之前干到哪了" | Load |
| Mid-session, a `Done when:` is now satisfied | Close (auto) |
| Network dropping / switching machines / explicit handoff / pre-compaction with unfinished state | Save |
| Gate failed on an existing oversized file | Repair |
| Routine commit with no unfinished work | Do nothing |
| `git merge` / `rebase` / `cherry-pick` touches PROGRESS.md | Resolve by union (Save step 1) |
