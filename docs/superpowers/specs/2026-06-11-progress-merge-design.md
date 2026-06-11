# Progress Merge Skill 设计文档

> 日期：2026-06-11
> 状态：Design — 待审阅
> 关联 skills：progress-save, progress-restore, progress-archive, progress-summary

## 1. 概述与动机

现有 progress 系列由 save / restore / archive / summary 四个 skill 构成，
覆盖 PROGRESS.md 在**时间轴**上的所有动作（写入、读取、归档、跨会话快照）。

但当用户在不同 git 分支上独立维护 PROGRESS.md 时，分支合并会触发
冲突，而 git 原生三方合并对 PROGRESS.md 是**行级**操作——它不理解
"Current Focus 该保留谁、Completed 该 union"这种**语义级**问题。

`progress-merge` 补齐**分支轴**上的协作动作：跨两个分支语义化地合并
PROGRESS.md，按 section / task 粒度做启发式仲裁，必要时弹问用户决策。

## 2. 范围与非目标

### 2.1 In Scope

- **场景 A**：`git merge` / `git rebase` / `git cherry-pick` 触发的
  PROGRESS.md 冲突解决
- **场景 C**：dry-run 对比两侧 PROGRESS.md 状态，不写盘

### 2.2 Out of Scope

- **N 路合并**（>2 源）：合并永远是 HEAD 与 MERGE_HEAD 二者之间，
  N 路场景不存在
- **跨分支主动拉取进度**（场景 B）：暂不支持。可在用户提出明确需求
  后追加，技术路径与场景 A 同源
- **PROGRESS.md 以外的文件 merge**：本 skill 只处理 PROGRESS.md
- **`git commit` / `git merge --continue`**：合并 PROGRESS.md 后是否
  推进 git 状态由用户/后续 workflow 决定
- **Git hook 自动触发**：项目已在 commit `b28dd81` 明确"反 hook"，
  本 skill 不引入任何 hook 配置

### 2.3 核心定位

skill 主要解决 **AI 主导 merge** 场景中的 PROGRESS.md 合并问题——
即用户让 AI 去 merge 两个分支时，让 AI 能按 skill 设定的顺序与策略
正确更新 PROGRESS.md。次要场景（用户外部 merge 后找 AI 协助）能
覆盖到的部分尽量覆盖，物理上覆盖不到的不强求。

## 3. 触发与三层防御

### 3.1 触发路径能力评估

| 路径 | 触发方 | 能力 |
|---|---|---|
| ① description 匹配 | Claude 解析用户消息 | ✅ 可靠 |
| ② AI 自身执行 git merge 时主动调用 | Claude 在 Bash tool 跑 `git merge` 前后 | ⚠️ 可靠**但前提是项目已安装 AGENTS.md 模板**（否则降级到路径 ①） |
| ③ progress-save 哨兵跳转 | save 检测 conflict marker | ⚠️ 半自动，要求用户先调 save |
| ④ 用户在 AI 会话外手动 merge | （无自动触发） | ❌ AI 看不见，需用户主动开口 |
| ⑤ Git hook 自动注入 | settings.json hook | ❌ 项目反 hook，排除 |

### 3.2 三层防御

**第 1 层 — description 触发（CSO 风格）**

description 仅列触发短语集合，不总结工作流（避免 CSO 反模式）：

```yaml
description: Use when PROGRESS.md has merge conflicts (after `git merge`
  / `git rebase` / `git cherry-pick` involving PROGRESS.md), or when user
  wants to compare/merge progress state between two branches (合并 PROGRESS
  冲突, merge progress across branches, 两个分支的 PROGRESS 怎么合,
  PROGRESS conflict 怎么解决, 解决 progress 冲突, dry-run compare branches)
```

覆盖 EN/ZH 双语 + 短形式触发词。

**第 2 层 — AGENTS.md 情境规则（核心约束）**

> **前置条件**：该层依赖项目方在 AGENTS.md 中安装下方模板片段。若项目
> 未安装，本层失效，约束力降级到仅靠第 1 层 description 匹配 + 第 3 层
> 哨兵兜底。模板的安装方式见 §16。

约束 AI 自身在主导 merge 时的行为顺序。在 README 的 AGENTS.md 模板
中追加：

```markdown
## Progress Tracking (Critical)

### Situation-based triggers
- Before commit/stash/PR: Call `/progress-save`
- When resuming work: Call `/progress-restore`
- When major task done: Call `/progress-archive`
- When starting new session: Call `/progress-summary`
- Before `git merge` / `git rebase` involving PROGRESS.md (or detected
  PROGRESS.md conflict markers): Call `/progress-merge`
- After resolving any git operation that left PROGRESS.md in conflict
  state: Call `/progress-merge` before continuing

### Sequencing during `git merge` operations
When AI executes `git merge` / `git rebase` / `git cherry-pick` and
conflicts arise:
1. FIRST resolve non-PROGRESS.md conflicts in the working tree
2. Verify tests pass or merge correctness is confirmed
3. THEN call /progress-merge to handle PROGRESS.md conflict
4. Finally `git add` + complete the merge

### User-phrase triggers
... (沿用现有四个 skill 的对应短语)
- User says "merge progress", "合并进度", "PROGRESS 冲突",
  "两个分支的 PROGRESS 怎么合" → `/progress-merge`
```

**第 3 层 — progress-save 哨兵**

progress-save 增加 Step 0 conflict pre-check（详见 §12.2）。

**save 哨兵跳转的 wiring**：

哨兵检测条件有两类，跳转后 merge skill 的处理路径**不同**：

| 哨兵检测到 | merge 收到的 git 状态 | merge 的处理路径 |
|---|---|---|
| PROGRESS.md `UU`（git 真冲突）| `git status` 显示 unmerged | 走标准 Mode A（§4.2） |
| 文件含 `<<<<<<<` marker 但 git 不在 unmerged 状态（如用户手工编辑残留）| `git status` 正常 | 走 §11 / T14 路径：提示用户选 ①手工清理 ②按 Mode C 比较两分支 ③取消 |

merge skill 收到调用时必须先用 `git status --short` 判别实际状态，
不能假设"被 save 哨兵调起来 = 一定在 unmerged 状态"。

### 3.3 不可触场景的诚实声明

剩下"用户在 AI 会话外手动 merge 又乱按顺序"这种情况，**任何 skill
都管不了**——这是工程纪律问题，不是技术问题。skill 在 AI 在场时
尽力引导正确顺序。

## 4. 数据流与执行阶段

```
触发
  ↓
Step 0: Mode Detection + Sequencing Check  (仅 Mode A)
  ↓
Step 1: Read Both Sides
  ↓
Step 2: Parse (Section + Task granularity)
  ↓
Step 3: Collect Git Context
  ↓
Step 4: Apply Merge Strategy Table
  ↓
Step 5: Heuristic Arbitration (Batched AskUserQuestion)
  ↓
Step 6: Preview (含可能的 cautionary note)
  ↓
Step 7: Write + Optional `git add`         (仅 Mode A)
```

### 4.1 Mode 区分

> 注：§2.2 已说明场景 B（跨分支主动拉取进度）暂不在 scope，故 Mode 表
> 只列 A 和 C；未来若加 B，会作为独立 Mode 追加。

| Mode | 触发 | 输入来源 | 是否落盘 |
|---|---|---|---|
| **A** Conflict Resolution | 用户在 unmerged 状态调用 `/progress-merge`，或被 save 哨兵跳转 | `git show :2:PROGRESS.md` + `git show :3:PROGRESS.md`，**stage 含义需先做 git 状态判别**（见 §4.2） | ✓ |
| **C** Compare (dry-run) | `/progress-merge --compare <ref1> <ref2>` 或 `/progress-merge --compare <path1> <path2>` | 分别从两个 ref 或两个文件路径读取 | ✗ |

### 4.2 Stage 编号的语义陷阱（merge / rebase / cherry-pick）

git 在 unmerged 时把冲突文件的三个版本放在 stage 1/2/3。语义随触发命令
变化，**spec 必须先检测 git 状态、再按 mode 解释 stage**：

| 触发命令 | `:2:` 含义 | `:3:` 含义 | 用户直觉的 "ours" | 用户直觉的 "theirs" |
|---|---|---|---|---|
| `git merge X`（在 HEAD 分支） | HEAD（当前分支）| X（被 merge 的分支）| `:2:` | `:3:` |
| `git rebase X`（在 feature 分支） | **X（base）** | **feature（被 replay 的）** | **`:3:`**（反转！）| **`:2:`**（反转！）|
| `git cherry-pick X`（在 HEAD 分支） | HEAD | X 那一个 commit | `:2:` | `:3:`，但范围窄（见 §8.3） |

**检测步骤**（按顺序逐条执行，命中即停）：

**Step 0a: 先取真实 git 目录路径**（worktree 兼容）

`.git` 在 worktree 下是文件不是目录（内容是 `gitdir: <path>`），直接
`test -d .git/...` 会误判。先执行：

```
git rev-parse --git-dir
```

把返回值记为 `<GITDIR>`。后续所有 `test` 命令都用 `<GITDIR>/...`
替代硬编码的 `.git/...`。

**Step 0b: 逐条判定模式**

| # | 命令 | 命中条件 | 结论 |
|---|---|---|---|
| 1 | `test -d <GITDIR>/rebase-merge` | 退出码 0 | rebase 模式（交互式或 merge 风格）|
| 2 | `test -d <GITDIR>/rebase-apply` | 退出码 0 | rebase 模式（apply 风格）|
| 3 | `test -f <GITDIR>/CHERRY_PICK_HEAD` | 退出码 0 | cherry-pick 模式 |
| 4 | `test -f <GITDIR>/MERGE_HEAD` | 退出码 0 | merge 模式 |
| 5 | 全部失败 | — | 不在冲突状态 → 走 §11 的"无 conflict 上下文"分支 |

LLM 应**逐条执行**这些 `test` 命令（每条都是独立命令），第一个返回 0
的就是当前模式。不要把这些组合成 shell 脚本——它们是 LLM 的判断
步骤，不是可执行脚本。

**Spec 内的"ours"和"theirs"约定**：本设计文档中所有提及 "ours" / "theirs"
的地方一律指**用户直觉语义**（即"我的版本" / "对方的版本"），实现时
按上表把 stage 编号映射到正确的一边。**严禁**在 spec 或实现中混用
git 内部 stage 编号和用户直觉语义。

### 4.3 各 Step 详述

详见 §5（策略表）、§6（task 规则）、§7（仲裁）、§8（git 上下文）、
§9（时序）、§10（交互）。

## 5. 合并策略表

按 progress-save 现有的 9 类 section 各自定策略：

| Section | 默认策略 | 推荐依据信号源 | 说明 |
|---|---|---|---|
| `> Last updated` | **max** | — | 取较新时间，无歧义 |
| `🎯 Current Focus` | **问用户** | 两边 commit 时效性、活跃度 | 选保留 ours / theirs / 全部并存（标注分支来源） |
| `📥 Next Phases` | **union + task 去重** | task 名相似度、commit grep | 不同分支可能计划重叠；LLM 判断为"不确定是否同一个"的 task → 默认当成两个不同 task（都保留），**不进必问队列**——计划项保留两个近似计划比让用户决策更省事 |
| `⏸️ Paused Tasks` | **union** | — | 暂停任务跨分支都该留 |
| `✅ Recently Completed` | **union + 时间排序 + 截取 N 条** | task 状态优先级 | N = `max(ours 条目数, theirs 条目数, 3)`；union 后超过 N 的最早条目截断，preview 摘要里告知 "X 条早期已完成项被截断，建议 /progress-archive 归档" |
| `🧱 Blockers & Issues` | **union + 去重** | — | 问题不该丢 |
| `🧠 Context Notes` | **union（追加）** | — | 笔记类宁多勿漏 |
| `⚡ Quick Recovery` | **union + 标注来源** | — | theirs 条目标注 `[from <branch>]`，避免遗漏关键恢复信息（如环境变量、依赖装载命令）；用户可在 preview 阶段决定是否保留 |
| `🏛️ Archive Links` | **union** | — | 纯链接，最安全 |

### 5.1 Section 命名兼容

复用 save / restore / summary 现有的 intelligent section detection 模式
识别两侧 section 类型。**本 skill 内置完整的 9 行 detection 表**
（save 现有表只列 6 行，缺 Recovery / Archive Links / Last updated 行；
建议在本次 PR 顺便补齐 save，让两者一致——见 §16）：

| Content Type | Possible Section Names |
|---|---|
| Last updated time | `> Last updated`, `> 最后更新` |
| Current focus | `🎯`, `Current Focus`, `当前`, `Current` |
| Next steps | `📥`, `Next Phases`, `Todo`, `下一步`, `Next` |
| Paused tasks | `⏸️`, `Paused Tasks`, `暂停`, `Hold` |
| Completed | `✅`, `Completed`, `已完成`, `Recently Completed` |
| Blockers | `🧱`, `Blockers`, `Issues`, `问题` |
| Notes | `🧠`, `Context Notes`, `Notes`, `备注`, `Context` |
| Recovery | `⚡`, `Quick Recovery`, `恢复`, `Recovery` |
| Archive Links | `🏛️`, `Archive Links`, `归档`, `Archive` |

如果两侧 section 命名风格不同（一边 `🎯 Current Focus`，一边
`## 当前焦点`），列入"必问"队列，让用户决定以哪边命名为准。

### 5.2 Format Preservation

沿用现有 skill 的硬约束："只更新内容，不重构格式"。merge 输出的
PROGRESS.md 必须保持其中一侧（默认 ours）的 section 命名风格、顺序、
emoji 用法。**不强制对齐到标准格式**。

### 5.3 大文件性能预算

PROGRESS.md 在 archive 前可能积累上千行。task 级解析 + 逐 section 策略
应用会产生大量 token，需要预算控制：

**判定阈值**：单侧 PROGRESS.md > **500 行**视为大文件。

**降级策略**（任一侧超过阈值即生效）：
- **核心 section**（`Current Focus` / `Next Phases` / `Paused Tasks` /
  `Recently Completed` / **`Blockers & Issues`**）→ 仍做完整 task 级
  解析与启发式仲裁。Blockers 即便降级也保留 task 级——一条 blocker
  遗漏可能导致生产事故，多解析一个 section 的 token 代价可控
- **次要 section**（`Context Notes` / `Quick Recovery` /
  `Archive Links`）→ 退化为整段 section-level union（两侧文本拼接 +
  全文级去重），不做 task 拆分
- **`Last updated`** → 仍取 max，无影响
- 在 preview 摘要顶部标注："文件较大（X 行），次要 section 已降级
  为 section-level union；如需完整 task 级合并，建议先 /progress-archive
  收敛后再 merge"

**建议用户先归档**：超过 1000 行时，preview 之前先弹一个轻量提示
"PROGRESS.md 已超过 1000 行，强烈建议先 /progress-archive，再来 merge"
——但不强制阻断。

## 6. Task 级合并规则

每个 union 操作落到 task 层。

### 6.0 Task 的解析启发式（前置）

PROGRESS.md 不是结构化数据，task 可能以多种格式存在。Parse 阶段
（§4 Step 2）按以下启发式识别 task 边界：

| 格式 | 识别规则 | 示例 |
|---|---|---|
| **Checkbox** | 行以 `- [ ]` / `- [x]` / `* [ ]` / `* [x]` 开头 | `- [x] 实现 OAuth` |
| **Bullet** | 行以 `-` / `*` / `+` 开头（无 checkbox）| `- 修 PROGRESS 触发词` |
| **子标题** | `###` / `####` 开头，行内含 `:` 或状态 marker | `### Phase 4.1: Tools Module ✅` |
| **编号** | 行以 `1.` / `2.` 开头 | `1. 重写 runtime` |

**Task 的"属性"提取**：
- **标题** = 去掉前缀 marker、checkbox、状态 emoji 后的剩余文本
- **状态** = 根据 marker 推断（`[x]`/`✅` = done、`[ ]` = pending、
  `⏸️` = paused、行内含 `(WIP)`/`(进行中)` = in_progress、其他为 pending）
- **子项** = 后续缩进更深的 bullet/checkbox 行
- **描述** = 标题行后非 task 结构的文本（如注释、说明）

**子标题格式的 task 体（特殊规则）**：

子标题（`### Phase X` 等）格式的 task，其**"体"的边界**是以下任一
最先出现的标记：

1. **下一个同级标题**（如本 task 是 `###`，遇到下一个 `###`）
2. **更高级标题**（如本 task 是 `###`，遇到 `##` 或 `#`）
3. **`---` 水平分隔线**（独立成行，PROGRESS.md 常用作 task 视觉分隔）
4. **文件结尾**

任一命中即停。整段内容——包括 free-form 段落、bullet、代码块、引用——
都视为该 task 的"体"，整体参与合并仲裁。

**不做嵌套拆解**——避免引入子项的子项这种递归解析复杂度。例如：

```markdown
### listSnapshots 过滤 pre-rewind-safety + rewindPrefill refactor ✅ (2026-06-08)

分支：`wip/gateway-tui`

闭环了 2 个 hygiene 修复：
- 修复 A
- 修复 B

根因：旧版本在 X 处遗漏 Y

---

### 下一个 task ...
```

整段（从 `###` 到 `---` 之前）作为 **一个 task 的"体"**：合并时
按"体的内容是否高度相似"判定是否同一个 task；若同一个 task 两边的
"体"差异大，整体进必问队列（不再细拆 bullet）。

**降级规则**：
- 无法识别 task 边界的纯文本块 → 整段作为 "section-level union"，
  不做 task 拆分（避免误识别）
- 同一 section 内格式混用（如 checkbox + 子标题）→ 按每种格式分别
  解析，结果合并

### 6.1 "是否同一个 task" 判断

由 LLM 语义判断，**不硬编码相似度阈值**——LLM 自身具备 task 名近义
识别能力，硬阈值反而限制其判断力。判断结果落到三档：

```
1. 明确同一个（normalize 后同名，或语义高度等价）        → 自动合并
2. 不确定（语义近似但有歧义，如 "Phase 4.1" vs "Phase 4")→ 进必问队列
3. 明确不同（语义无关）                                   → 当成两个 task
```

LLM 在"不确定"时**必须**把判断收敛到必问队列，不能擅自决定——这是
本 skill"启发式仲裁"风格的硬约束。

### 6.2 同一 task 的内容仲裁

| 字段 | 仲裁规则 |
|---|---|
| 状态（pending / in_progress / ✅ / ⏸️） | 取"更靠右"的（完成度高的赢） |
| 描述文字 | 若内容差异大（语义方向不同） → **一律进必问队列**，不自动择长；仅当描述高度相似（如一边是另一边的精简版）才取较长的 |
| 子项（phases / bullets） | union + 去重 |
| 时间戳 | max |

### 6.3 状态完成度顺序

**清晰段**（可直接比较）：

```
pending  <  in_progress  <  ✅ done
```

`✅ done` 永远赢；`pending` 永远输（除非另一边也是 pending）。

**灰色段**（paused 与 in_progress 不可线性排序）：

`⏸️ paused` 是一个**语义特殊**的状态——它可能代表：
- "推进到 80% 后暂停"（实际比 in_progress 更完成）
- "刚开始就暂停"（实际等同于 pending）

因此当一侧是 paused、另一侧是 in_progress 或 pending 时：
- 若 paused 条目带有进度描述（如"已完成 X，待审核 Y"），由 LLM 据此
  判断它是否比对方更"完成"
- 若无法据描述判断 → **进必问队列**，让用户决策
- 若另一侧是 `✅ done` → 取 done（done 始终赢）

**关于必问的硬约束**：本节灰色段的判断与 §6.1 同——LLM 不确定时
**必须**收敛到必问队列，不擅自决定。

## 7. 启发式仲裁与推荐机制

### 7.1 必问场景清单

跑完算法后，下列情况会被收集到"必问"队列：

1. **Current Focus 两边都非空且不同**
2. **同名 task 状态冲突无法自动决定**（如一边删了、一边改了；或
   paused vs in_progress 灰色段且无法据描述判断，见 §6.3）
3. **task 名 LLM 判断为"不确定是否同一个"**（见 §6.1）
4. **section 命名风格差异**

### 7.2 Batching 规则

`AskUserQuestion` 工具规约：一次调用支持 1–4 个 question，每个 question
2–4 个 option。本 skill 的 batching 严格对齐这个工具规约：

- 必问场景 ≤ 4 个 → 一次 AskUserQuestion 调用，每场景对应 1 个 question
- 必问场景 > 4 个 → 按重要性分批（Current Focus 类优先），每批 ≤ 4 个
  question。**进度标嵌入 question 的 `header` 字段**（AskUserQuestion
  的 `header` 限 12 字符）：
  - 批次总数 ≤ 9 时：格式 `X/Y·<类型>` 共 ≤ 12 字符
    - 例：`2/3·CF` / `1/2·Task` / `3/3·Name` / `2/2·Sect`
    - 类型缩写：`CF` = Current Focus / `Task` = 同名 task 状态 /
      `Name` = task 名 LLM 不确定 / `Sect` = section 命名
  - **批次总数 > 9 时**：缩写为 `X/Y`（省略类型），避免接近 12 字符上限
    - 例：`10/12` / `11/12`
  - 进度信息直接出现在用户的交互 UI（chip/tag 位置），无法被忽略
- 全部无歧义 → 跳过提问，直接进 preview

注：本节"4 个"指的是 **4 个 question**（即 4 个独立必问场景），不是
4 个 section——一个 section 可能产生多个必问场景（如同时有命名风格
冲突 + 多个 task 状态冲突）。

### 7.3 推荐选项设计

每个 AskUserQuestion 选项必须满足：

- **第一个选项** = 推荐选项，label 末尾加 `(Recommended)`
- **每个选项的 description 字段** = 一行推荐依据
- 推荐基于 §8 的 git 上下文 + §6 的状态优先级 + 当前 HEAD 信号

**示例**：

```
❓ 两个分支的 Current Focus 都非空且不同，怎么处理？

  ◉ 并存（标注来源）              (Recommended)
     依据：两边都有近 2 小时内的 commit，
          说明都在并行推进，没有明显"主"工作流

  ◯ 保留 ours (main)
     依据：你当前在 main，commit 也更早 (a3f2c1d 2026-06-10)

  ◯ 保留 theirs (feature/oauth)
     依据：theirs 更新鲜 (b8e4f29 2026-06-11)，多 5 个 commit

  ◯ 全删重写
```

```
❓ "Phase 4.1: Tools" 状态冲突，怎么处理？

  ◉ 取 theirs (✅)               (Recommended)
     依据：theirs 有 commit 印证完成
          • b8e4f29 fix(tools): finish phase 4.1 tests
          • 9c2d108 feat(tools): wrap up phase 4.1
          ours 该范围内无相关 commit

  ◯ 取 ours (in_progress)
     依据：你当前在 ours，可能有未 commit 的改动

  ◯ 两者并存
```

## 8. Git 上下文收集

### 8.1 默认收集（Step 3 中执行）

```bash
# 两边相对差异（核心）
git log --oneline -20 OURS..THEIRS
git log --oneline -20 THEIRS..OURS

# 分叉点 + 时间锚点
git merge-base OURS THEIRS
git log -1 --format='%H %ai %s' OURS
git log -1 --format='%H %ai %s' THEIRS
```

**关于 `OURS` / `THEIRS` 参数的解析**：

这里的 `OURS` / `THEIRS` 是**分支名 / commit ref**（git log 接受
的引用），不是 stage 编号。在不同模式下，要按 §4.2 的映射表把
"用户直觉 ours/theirs" 翻译到正确的分支引用。**所有路径必须用
`<GITDIR>` 替代硬编码 `.git/`**（worktree 兼容，见 §4.2 Step 0a）：

| 模式 | git log 用的 `OURS` | git log 用的 `THEIRS` |
|---|---|---|
| `merge` | 当前 HEAD 分支名（`git symbolic-ref --short HEAD`）| `cat <GITDIR>/MERGE_HEAD`（commit hash）|
| `rebase` | **`cat <GITDIR>/rebase-merge/head-name`**（内容如 `refs/heads/feature`，直接作 ref 用，git rev-parse 接受全路径）| **`cat <GITDIR>/rebase-merge/onto`**（commit hash）|
| `cherry-pick` | 当前 HEAD 分支名 | `cat <GITDIR>/CHERRY_PICK_HEAD`（commit hash）|

具体读取命令：

```
# merge 模式
ours_ref=$(git symbolic-ref --short HEAD)
theirs_ref=$(cat <GITDIR>/MERGE_HEAD)

# rebase 模式（注意 ours/theirs 与 stage 反转）
ours_ref=$(cat <GITDIR>/rebase-merge/head-name)
theirs_ref=$(cat <GITDIR>/rebase-merge/onto)

# cherry-pick 模式
ours_ref=$(git symbolic-ref --short HEAD)
theirs_ref=$(cat <GITDIR>/CHERRY_PICK_HEAD)
```

注意 rebase 模式下用户直觉的"ours"是 feature 分支（被 rebase 的那个），
不是 `git status` 里看到的"HEAD now at"。**spec 全文 ours/theirs 一律
按用户直觉理解**，实现层在此处做参数转换。

**展示层优化**：rebase 模式下 `head-name` 返回全路径 ref（如
`refs/heads/feature`），`git log` 命令用全路径 ref 更可靠，但 preview
摘要展示时用短名更可读。读取后加一步：

```
git symbolic-ref --short <full-ref> 2>/dev/null || echo <full-ref>
```

`git log` 命令仍用全路径 ref，preview 摘要用短名。

### 8.2 按需收集（仲裁时触发）

```bash
# task 名 grep commit message（仅在出现 task 冲突时）
git log --oneline --grep="<task-keyword>" -10 OURS
git log --oneline --grep="<task-keyword>" -10 THEIRS
```

**`<task-keyword>` 提取规则**（中英文场景区分）：

| 步骤 | 规则 |
|---|---|
| 1 | 去除状态 marker（✅ / ⏸️ / `(WIP)` 等） |
| 2 | 去除前置序号（`Phase 4.1:`, `1.`, `- [ ]` 等） |
| 3 | 去除 emoji |
| 4 | **优先提取英文/代码 token**：驼峰词（`listSnapshots`）、连字符词（`pre-rewind-safety`）、下划线词、纯英文单词 |
| 5 | 若英文/代码 token ≥ 1 个 → 取前 1–2 个作 keyword（高匹配率，commit message 通常是英文）|
| 6 | 若英文/代码 token 数 = 0（task 名纯中文）→ 取剩余文本前 1–2 个中文词作 keyword |

**示例**：

| Task 名 | 提取的 keyword |
|---|---|
| `Phase 4.1: Tools Module ✅` | `Tools Module` |
| `listSnapshots 过滤 pre-rewind-safety + rewindPrefill refactor ✅` | `listSnapshots pre-rewind-safety`（取英文 token，跳过"过滤"）|
| `Fix login redirect bug` | `login redirect` |
| `实现用户登录` | `实现用户`（纯中文，取前 2 词）|

**Fallback 规则（提升召回）**：
- 第一次用提取的 keyword 搜（高精度）
- 若结果为空 → **再用完整 task 名搜一次**（高召回）
- 两次结果合并去重后展示给用户作推荐依据
- 例：`Fix login redirect bug` → keyword `login redirect`，若搜空 →
  再用 `Fix login redirect bug` 整句搜

### 8.3 Dry-run 模式下的降级

| Dry-run 输入 | git 上下文 | 推荐质量 |
|---|---|---|
| 两个 git ref（分支/commit）| ✅ 完整收集 | 完整推荐 |
| 两个文件路径（非 git）| ❌ 跳过 | 仅基于文本启发式，推荐弱一些 |

降级时，推荐选项的 description 字段需明确标注"基于文本启发式"，
让用户知道判断依据较弱。

### 8.4 Cherry-pick 场景的特殊处理

cherry-pick 与 merge / rebase 不同——theirs 侧只反映**单个 commit**
的改动，不是整个分支的最新状态：

- `git log THEIRS..OURS` 范围可能很小甚至为空（因为 cherry-pick 的
  来源 commit 只是一个孤立点，不代表完整分支历史）
- `git log OURS..THEIRS` 通常只返回那一个 cherry-pick 的 commit

**调整策略**：
- 检测到 cherry-pick 模式（`.git/CHERRY_PICK_HEAD` 存在）→ 在推荐依据
  中明确标注 "theirs 来自单个 cherry-pick commit，不代表完整分支状态"
- 若 `git log THEIRS..OURS` 为空 → 跳过这条信号，仅使用 OURS..THEIRS
  的结果作推荐依据
- 时效性比较（"谁更新鲜"）在 cherry-pick 场景下意义降低 → 推荐倾向
  从"时间"转向"内容覆盖度"

## 9. 时序约束（三层说明）

### 9.1 Layer 1 — AGENTS.md 规则（约束 AI）

当 **AI 主动执行** `git merge` / `git rebase` 时，按 AGENTS.md
sequencing 规则走：先解决非 PROGRESS.md 冲突 → 验证 → 调用
`/progress-merge`。这层在调用 progress-merge **之前** 生效，
是核心约束。

### 9.2 Layer 2 — Pre-check Warn（向用户提问）

`progress-merge` Step 0 检测：

```bash
git diff --name-only --diff-filter=U
```

若除 PROGRESS.md 还有其他 unmerged 文件 → 弹 AskUserQuestion：

```
⚠️ 检测到还有 N 个文件处于冲突状态：
   - src/api/auth.ts
   - src/utils/db.ts
   - ...

PROGRESS.md 描述的是"代码推进到哪一步"，建议先解决以上文件的
冲突（让代码 merge 完成、test 跑过）再来 merge PROGRESS.md。

❓ 现在怎么办？
  ◉ 暂停，先去解决代码冲突            (Recommended)
  ◯ 继续，先 merge PROGRESS.md
  ◯ 取消
```

**接收者是用户**，AI 是问题呈现者。AI 基于 Layer 1 规则把"暂停"
设为推荐项，但用户做最终决策。

### 9.3 Layer 3 — Cautionary Note（向用户告知）

若用户在 Layer 2 选择"继续"，preview 摘要顶部加 cautionary note：

```
⚠️ Cautionary note: 你跳过了 sequencing check，当前还有 N 个文件
   未解决。PROGRESS.md 合并结果基于"假设上述冲突按当前 working
   tree 状态解决"。如果后续回滚代码 merge，请重跑 /progress-merge。
```

仅展示，无问题。**接收者是用户**，仅作知情通知。

## 10. 用户交互流程

### 10.1 Preview 格式

落盘前必须给用户看一个紧凑、可扫读的 preview，分两块：

**(a) 变更摘要**

```
🔀 PROGRESS.md 合并预览

[可能的 cautionary note]

📊 来源：
  ours    = main           (a3f2c1d 2026-06-10)
  theirs  = feature/oauth  (b8e4f29 2026-06-11)

⚙️ 自动决策：
  ✓ Last updated → 2026-06-11 (max)
  ✓ Recently Completed → union 2 项，按时间排序
  ✓ Blockers → union 1 项（ours "无" 被丢弃）
  ✓ Quick Recovery → union，theirs 条目标注 [from feature/oauth]

❓ 你的决策：
  • Current Focus → 选择"并存（标注分支来源）"
  • Phase 4.1 状态 → 选择"取 theirs (✅)"
```

**段落出现规则**：
- 若所有 section 都自动决策（无必问）→ 省略 "❓ 你的决策" 段
- 若所有 section 都靠用户决策（无自动）→ 省略 "⚙️ 自动决策" 段
- 若某段为空 → 保留段标题 + 显示 "（无）"，避免用户误以为是渲染 bug

**(b) 完整 PROGRESS.md 预览**

紧跟摘要后直接显示 fenced markdown 块，最终落盘内容。

**大文件截断**：合并后 PROGRESS.md > **300 行**时（终端阅读舒适度
阈值，低于 §5.3 的 500 行解析性能阈值），不显示完整 fenced markdown，
改为：

- 仅显示**各 section 的 diff 概要**（新增哪些条目 / 删除哪些条目 /
  修改哪些条目，按 section 列出）
- 在结尾输出 "完整合并结果将在落盘后写入 PROGRESS.md，可用编辑器
  打开查看"
- 落盘后**仍然**走 §10.2 的 confirm 流程，不绕过用户审核

### 10.2 落盘 + git 解除冲突（Mode A）

合并为**一个 AskUserQuestion**，减少交互轮次：

```
[preview 展示完]
→ Ask: "如何处理这个合并结果？"
   ◉ 写入 PROGRESS.md + git add（解除冲突）   (Recommended)
   ◯ 仅写入 PROGRESS.md（保留 unmerged，自己审）
   ◯ 再改改（回到 Preview 重做）
   ◯ 取消（PROGRESS.md 保持原冲突状态）
```

**"再改改" 循环退出机制**：
- 单次 merge 流程最多允许 **3 次** "再改改"
- 进入第 4 次时，"再改改"选项消失，只剩 "写入 / 仅写入 / 取消"
- 第 3 次循环时在 preview 顶部提示 "已是第 3 次修订，下一次将强制选择"

**异常路径**：

- 若用户选"写入 + git add"，**写入成功但 git add 失败**（权限/锁等）→
  告知用户文件已写入但 git add 失败，提供手动命令：`git add PROGRESS.md`
- 若用户选"写入 + git add"，**写入本身失败**（磁盘/权限）→ 报错并退出，
  git 状态不变
- **并发安全 — 写入前重读**：preview → confirm 之间存在时间窗口，
  用户可能另开终端修改了工作区的 PROGRESS.md。比对对象**必须是工作区
  文件**（不是 `:2:` / `:3:`，后者是 git index 里的两侧版本）：
  - **Step 1 时**：读取并记录工作区 `PROGRESS.md` 的内容（或 SHA-256
    hash 等指纹）。在 unmerged 状态下，这份内容包含 `<<<<<<<` markers
  - **Step 7 写入前**：重新读取工作区 `PROGRESS.md` 并与 Step 1 的
    指纹比对
  - 任一字节不一致 → 中止写入，提示 "检测到 PROGRESS.md
    在 preview 期间被外部修改（可能是你手动解了冲突），
    请重跑 /progress-merge 以基于最新内容重新合并"
  - 不要拿 `git show :2:PROGRESS.md` 做比对——`:2:` 是 git index 里
    的版本，用户编辑工作区文件不会改它

**绝不**主动执行 `git commit` 或 `git merge --continue`——把推进 git
状态的决定留给用户/后续 workflow。

### 10.3 Mode C 收尾

```
[preview 展示完]
→ 不问任何写盘问题，直接结束，但输出引导句：

  💡 后续操作提示：
  - 实际合并到当前分支：在 unmerged 状态下使用 `/progress-merge`（Mode A）
  - 保存对比报告：复制上方 preview 内容到 .md 文件
  - 切换分支后再 merge：先 `git checkout <target>` 再触发 Mode A
```

## 11. 错误处理矩阵

| 异常 | 处理 |
|---|---|
| PROGRESS.md 在某一侧不存在 | 告诉用户，问"是否直接取另一侧（degenerate merge）" |
| PROGRESS.md 两侧均不存在 | 退出，提示走 progress-save 初始化 |
| 当前不在 git 仓库（dry-run 用绝对路径除外） | 报错并退出 |
| 不在 unmerged 状态但用户主动调 `/progress-merge`（无 conflict 上下文） | 进入"主动比较"模式：问用户两边来源（HEAD vs 哪个分支/路径） |
| 在 unmerged 状态但 PROGRESS.md 不在冲突列表 | 跳过 PROGRESS.md，告诉用户没事可做 |
| 解析失败（非 markdown / 完全空白 / 二进制） | 报错并退出，提示手工修复 |
| 用户在批量提问中选了 "Other" 自由文本 | 当成 raw 内容直接插入对应 section |
| Preview 展示后用户选"取消" | 不写盘，git 状态保持原样 |
| `git add` 调用失败（权限/锁） | 文件已写入，告诉用户手动 `git add` |
| Mode A 中还有其他 unmerged 文件 | 走 §9.2 Layer 2 流程：soft warn + 用户决定 |
| PROGRESS.md 含 `<<<<<<<` 残留 marker 但 git 不在 unmerged 状态 | 提示用户残留冲突标记，让其选：①手工清理 ②按 Mode C 比较两个分支 ③取消（见 T14） |

## 12. 与其他 skill 的边界 & 哨兵集成

### 12.1 能力边界

| 方面 | save | restore | archive | summary | **merge** |
|---|---|---|---|---|---|
| 是否写盘 | ✓ | ✗ | ✓ | ✗ | ✓ |
| 读 git（深度） | `status --short` + `diff --stat` + `log -5 --oneline`（浅）| `log -3 --stat`（中，含变更文件）| ✗ | `status --short` + `diff --stat` + `log -5 --oneline`（浅）| **`log -20 OURS..THEIRS` + reverse + `merge-base` + `diff --name-only --diff-filter=U` + `show :2 :3`（最深）** |
| 执行 git 改动 | ✗ | ✗ | 可选 commit | ✗ | **可选 `git add` 解除冲突** |
| 触发 git 命令询问 | ✗ | ✗ | "commit?" | ✗ | **"write + git add?" 三选一** |
| 哨兵建议跳转 | → archive（任务完成）<br>**→ merge（检测 conflict）** | ✗ | ✗ | ✗ | ✗ |
| 解决 PROGRESS.md 冲突 | ✗ | ✗ | ✗ | ✗ | ✓（独占） |

### 12.2 progress-save 的最小修改

仅增 Step 0，不动其他逻辑：

```
### Step 0: Conflict State Pre-check (新增)
- Execute `git diff --name-only --diff-filter=U` to detect unmerged files
- If PROGRESS.md is in unmerged state OR contains `<<<<<<<` markers:
    Output: "PROGRESS.md is in merge conflict state. Use /progress-merge
             to resolve, then re-run /progress-save."
    Halt execution (do not proceed to Step 1-5)
- Otherwise: proceed to Step 1
```

与 save 现有的 Step 4 "completion detection → archive prompt" 完全
对称的模式。

### 12.3 不动其他 skill

restore、archive、summary 在本次设计中不修改。

## 13. SKILL.md 文件结构

**SKILL.md 与本设计文档的关系**：SKILL.md 是给 LLM 运行时读的**精简
实施版**，不是设计文档的完整搬运。两者职责不同：

| | 设计文档（本文件）| SKILL.md |
|---|---|---|
| 读者 | 维护者、reviewer | LLM 运行时 |
| 长度 | 600+ 行可接受 | 目标 **6–8KB**（≈ 200 行）|
| 包含 | 推导过程、trade-off、示例对话、版本演化 | 决策结果、表格、操作步骤 |
| 不包含 | — | 推导过程、示例对话、未决项、trade-off 论证 |

**裁剪原则**：

| 设计文档章节 | SKILL.md 处理 |
|---|---|
| §1 概述、§2 范围 | 仅一句话引语 |
| §3 三层防御 | 引用 README/AGENTS.md，不复述细节 |
| §4 数据流 | 保留 Step 0-7 骨架，删除流程图 |
| §5 策略表 | **完整保留**（运行时核心参考）|
| §6 task 解析 + 仲裁规则 | **完整保留** |
| §7 仲裁、§8 git 上下文 | 保留命令清单和必问场景列表，删除示例 |
| §9 sequencing | **三层说明完整保留** |
| §10 交互模式 | 保留落盘交互三选一，删除完整 preview 示例 |
| §11 错误处理矩阵 | **完整保留** |
| §12 边界表 | **完整保留** |
| §13 (本节) | 不进 SKILL.md |
| §14 description | 仅作为 frontmatter |
| §15 测试场景、§16 文档清单、§17 排除项、§18 trade-off | 不进 SKILL.md |

参照其他 4 个 skill 的范式：

```markdown
---
name: progress-merge
description: <见 §14.2 的完整 description 文本>
version: 1.0.0
---

# Progress Merge

Merge PROGRESS.md across two git branches semantically. Use during merge
conflicts on PROGRESS.md, or for comparing progress state between branches.

## Global Rules
1. Project Root Detection: Search upward from current directory until
   finding `.git` or `PROGRESS.md`.
2. File Path: All operations target `PROGRESS.md` in project root.
3. Language Following User: Analyze commit history and user input to
   auto-detect language.
   - **Detection Priority**: User input > Recent commit messages > System locale
   - **Supported Languages**: English (en) and Chinese (zh)
4. **If PROGRESS.md does not exist on either side**: Do NOT initialize
   (merge is not the right place for that). Instead:
   - Both sides missing → exit and direct user to `/progress-save`
   - One side missing → propose degenerate merge (take the other side
     as-is); user confirms before write
5. CRITICAL: Preserve Existing Format. Never restructure either side's
   section naming style; output follows ours (or user-chosen side).
6. Sequencing Awareness: see "Sequencing Awareness" section below.

## Execution Modes

### Mode A: Conflict Resolution
Triggered by:
- explicit `/progress-merge` while git index has PROGRESS.md as `UU`
- `/progress-save` sentinel redirect

Input: `git show :2:PROGRESS.md` + `git show :3:PROGRESS.md`.
**CRITICAL — stage mapping depends on git mode**: in `git rebase` the
stage 2/3 semantics are REVERSED vs `git merge`. Always run Step 0
(detect mode via `.git/MERGE_HEAD` / `rebase-merge/` / `CHERRY_PICK_HEAD`)
FIRST to determine which stage is "ours" and which is "theirs". See
spec §4.2 for the full mapping table.

### Mode C: Compare (Dry-run)
Triggered by:
- `/progress-merge --compare <ref1> <ref2>`
- `/progress-merge --compare <path1> <path2>` (non-git inputs)

Input: read from two refs or two file paths.

## Execution Flow
### Step 0: Mode Detection + Sequencing Check (Mode A only)
### Step 1: Read Both Sides
### Step 2: Parse (Section + Task granularity)
### Step 3: Collect Git Context
### Step 4: Apply Merge Strategy Table
### Step 5: Heuristic Arbitration (Batched AskUserQuestion)
### Step 6: Preview
### Step 7: Write + Optional `git add` (Mode A only)

## Merge Strategy Table
<§5 完整表>

## Task-Level Merge Rules
<§6 完整规则>

## Heuristic Arbitration & Recommendation
<§7 必问场景 + batching + 推荐示例>

## Git Context Collection
<§8 命令清单 + dry-run 降级>

## Sequencing Awareness
**Layer 1 (Constraint on AI)**: When AI is the actor executing
`git merge` / `git rebase`, follow AGENTS.md sequencing rule: resolve
non-PROGRESS.md conflicts FIRST, then call /progress-merge. This
prevents reaching the pre-check warning in the normal path.

**Layer 2 (Pre-check, addressed to user)**: If /progress-merge is
invoked while other unmerged files exist, surface a warning to the
user via AskUserQuestion. AI presents "pause and resolve code first"
as the recommended option, but the user makes the final decision.

**Layer 3 (Cautionary note, addressed to user)**: If user opts to
continue, embed a cautionary note in the preview so the user knows
the merged PROGRESS.md is based on a partially-resolved state.

## Interaction Patterns
<§10 preview 格式 + 落盘交互>

## Error Handling
<§11 错误处理矩阵>

## Difference from Other Skills
<§12.1 能力边界表>

## Standard PROGRESS.md Structure (Optional Reference)
<沿用 save/restore 的标准格式表>
```

预计 size：**6-8KB**。略大于其他 skill（4-6KB），符合该 skill 复杂度。

## 14. description 设计（CSO 原则）

### 14.1 反模式

不要在 description 里"总结工作流"——这会让 Claude 把 description
当 shortcut 跳过 SKILL.md body。

❌ 反模式示例：
```
description: Reads both sides of PROGRESS.md, merges section by section
  using heuristics, asks user for ambiguous cases, writes the result back.
```

### 14.2 正确写法

description 仅列**触发短语集合** + **触发情境**：

```yaml
description: Use when PROGRESS.md has merge conflicts (after `git merge`
  / `git rebase` / `git cherry-pick` involving PROGRESS.md), or when user
  wants to compare/merge progress state between two branches (合并 PROGRESS
  冲突, merge progress across branches, 两个分支的 PROGRESS 怎么合,
  PROGRESS conflict 怎么解决, 解决 progress 冲突, dry-run compare branches)
```

覆盖：
- 英文触发短语：`merge progress`, `PROGRESS conflict`, `compare branches progress`
- 中文触发短语：`合并 PROGRESS 冲突`, `两个分支的 PROGRESS 怎么合`,
  `解决 progress 冲突`
- 短形式：`PROGRESS 冲突怎么解决`
- 情境触发：`git merge` / `git rebase` / `git cherry-pick` 涉及
  PROGRESS.md

## 15. 测试场景清单

skill 上线前要跑通这些 case：

| Case | 输入 | 预期行为 |
|---|---|---|
| T1 基础 conflict | Mode A，两侧 PROGRESS.md 都有改动 | 触发完整流程，preview → 落盘 → ask git add |
| T2 一侧缺失 | Mode A，theirs 删了 PROGRESS.md | 提示 degenerate merge，问是否直接取 ours |
| T3 无歧义 union | 两侧改动都是 Completed 追加 | 无问题需问，直接 preview |
| T4 Current Focus 冲突 | 两侧 Current Focus 都非空且不同 | 弹问，附 git log 依据 |
| T5 task 状态冲突 | 同名 task ours=in_progress, theirs=✅ | 推荐"取 theirs"，依据含 commit message grep 结果 |
| T6 sequencing warn | Mode A，还有别的文件未解决 | Soft warn，默认推荐先去解决 |
| T7 dry-run 分支 | `--compare main feature/x` | 同主流程，但不写盘 |
| T8 dry-run 路径 | `--compare ./a.md ./b.md` | degraded mode，无 git log 信号 |
| T9 save 哨兵 | 在 unmerged 状态调 `/progress-save` | save 拒绝执行，提示走 merge |
| T10 取消落盘 | preview 完用户选取消 | 不写盘，git 状态不变 |
| T11 format preservation | 项目用自定义 section 名（非标准 emoji） | 沿用项目原有 section 名 |
| T12 EN/ZH 双语 PROGRESS | 一侧英文 section、一侧中文 section | 必问"以哪边命名风格为准" |
| T13 完全空 PROGRESS | 一侧空文件 | degenerate，直接取另一侧 |
| T14 残留 conflict marker | PROGRESS.md 含 `<<<<<<<` / `=======` / `>>>>>>>` 但 git index 无 unmerged 状态（如用户手工编辑残留） | 检测到 marker 但 git 状态正常 → 提示"检测到 PROGRESS.md 残留冲突标记，但 git 不在 unmerged 状态"，让用户选：①手工清理 ②按 Mode C 比较两个分支 ③取消 |

## 16. 文档更新清单

随 skill 一起改：

- `skills/progress-merge/SKILL.md` — 新增（按 §13 结构）
- `skills/progress-save/SKILL.md` — 加 Step 0 conflict pre-check（§12.2）；
  顺便补齐 detection 表的 Recovery / Archive Links / Last updated 三行
  （见 §5.1 注释）
- `skills/progress-restore/SKILL.md` 和 `skills/progress-archive/SKILL.md` —
  顺便把 Global Rules 第 3 条升级到带 priority 的两层结构（与 save/summary
  对齐）。这是可选的小幅一致性修复，不阻塞 merge 上线
- `README.md`
  - 添加 progress-merge 介绍（5 类 skill 表 + 4 字决策指南更新）
  - **强调 AGENTS.md 模板的安装是 §3.2 Layer 2 生效的前置条件**——
    没有它，AI 主导 merge 时的 sequencing 约束失效；README 应在
    "Progress Tracking" 那节明显位置标注 "Critical: install template
    in your project's AGENTS.md / CLAUDE.md / GEMINI.md"
  - 更新 AGENTS.md 模板（trigger 规则 + sequencing 规则，见 §3.2 Layer 2）
  - 添加安装命令 `npx skills add wuruofan/agent-skills --skill progress-merge -g -y`
- `README_CN.md`（与英文 README 保持同步，逐项中文化）
  - 在 "Included Skills" / "Progress Skills Suite" 节追加 progress-merge
    介绍：用途 / 触发时机 / 与其他 skill 关系
  - 安装命令 `npx skills add wuruofan/agent-skills --skill progress-merge -g -y`
  - 更新 AGENTS.md 模板：追加 `Before git merge/rebase involving PROGRESS.md`
    触发规则、sequencing 规则、`合并 PROGRESS 冲突` 等用户短语映射
  - "强烈建议先在项目的 AGENTS.md / CLAUDE.md / GEMINI.md 安装该模板"
    的中文版警示
  - 4 字决策指南（中文版）增加："合并分支进度? → /progress-merge"

其他 skill 主体逻辑（restore / archive / summary 的执行流程）本次不动。

### 16.1 合并后的完整 AGENTS.md 模板

避免维护者拼接出错，**直接给出合并后的完整模板**——README 应原样
搬入此片段：

```markdown
## Progress Tracking (Critical)

> **Critical**: This section must be installed in your project's
> AGENTS.md / CLAUDE.md / GEMINI.md to make AI sequencing constraints
> effective. Without it, /progress-merge falls back to weaker triggers.

### Situation-based triggers
- Before commit/stash/PR: Call `/progress-save` to update PROGRESS.md
- When resuming work after a break: Call `/progress-restore` to restore session context
- When a major task or all its phases are finished: Call `/progress-archive` to archive history
- When starting a new session to continue previous work: Call `/progress-summary` to get session context
- **Before `git merge` / `git rebase` / `git cherry-pick` involving
  PROGRESS.md** (or detected PROGRESS.md conflict markers): Call
  `/progress-merge`
- **After resolving any git operation that left PROGRESS.md in conflict
  state**: Call `/progress-merge` before continuing

### Sequencing during `git merge` operations
When AI executes `git merge` / `git rebase` / `git cherry-pick` and
conflicts arise:
1. FIRST resolve non-PROGRESS.md conflicts in the working tree
2. Verify tests pass or merge correctness is confirmed
3. THEN call /progress-merge to handle PROGRESS.md conflict
4. Finally `git add` + complete the merge

### User-phrase triggers (defense-in-depth)
- User says "update/save/record PROGRESS.md", "更新进度", "保存进度", "记录进度" → `/progress-save`
- User says "restore/resume/load context", "恢复进度", "接着干", "继续上次", "之前干到哪了" → `/progress-restore`
- User says "archive/clean up", "归档", "任务完成", "进度太长" → `/progress-archive`
- User says "summarize/recap/handoff", "总结一下", "会话总结", "交接", "写个摘要" → `/progress-summary`
- **User says "merge progress", "合并进度", "PROGRESS 冲突",
  "两个分支的 PROGRESS 怎么合" → `/progress-merge`**
```

加粗部分为本次新增内容。维护者应**直接替换**README 现有的 Progress
Tracking 节，不是手工拼接。

## 17. 显式排除的项

- ❌ `git commit` / `git merge --continue` — 不执行
- ❌ 自动跳到 `/progress-save` — merge 和 save 是两件事
- ❌ 静默写盘 — 任何写入前必有用户 confirm
- ❌ 修改 PROGRESS.md 外的任何文件
- ❌ Git hook 配置 — 项目反 hook
- ❌ N 路（>2 源）合并 — 不存在的需求
- ❌ 强制 PROGRESS.md 格式标准化 — 始终 preserve existing format

## 18. 已知 Trade-off 与未决项

本节列出设计过程中明确意识到、但**有意按当前方式定**的 trade-off。
实施后若发现假设错误，按此清单回头调整。

### 18.1 Quick Recovery 改为 union 而非择一

- **Trade-off**：union 更安全（不丢 theirs 关键恢复命令），但会让该
  section 在 merge 后变长，可能带入"在 ours 跑不通"的命令
- **当前选择**：union + 标注 `[from <branch>]`，让 preview 阶段交给
  用户筛
- **触发回调条件**：实际使用中若发现"标注 union 反而比择一更麻烦"
  （如用户每次都得手工删 theirs 条目），回退到 §5 原始的"择一"

### 18.2 Task 相似度交给 LLM 语义判断，不硬编码阈值

- **Trade-off**：放弃可复现的硬阈值（如 Jaccard 0.8），换取 LLM 语义
  判断的灵活性，但带来"同一输入两次判断不一定一致"的风险
- **当前选择**：LLM 自判 + 不确定收敛到必问队列
- **触发回调条件**：若实际跑下来"必问队列"在 task 多的项目中过载（如
  >10 个必问场景/次），考虑追加硬阈值过滤一道

### 18.3 落盘交互合并为一步（3 选 1）

- **Trade-off**：减少 UX 摩擦 vs 风格上跟 archive 的"两步制"不一致
- **当前选择**：合并为一步，因为 merge 流程前面已经问了多轮，再多
  一步交互边际成本高
- **触发回调条件**：若用户反馈"想要先看落盘结果再决定是否 git add"，
  退回两步制

### 18.4 不引入 Git Hook

- **Trade-off**：放弃"git merge 自动触发"的可能，换取与项目"反 hook"
  方向一致
- **当前选择**：靠 description + AGENTS.md + 哨兵三层覆盖
- **触发回调条件**：项目方向改变（如未来欢迎 hook）则可加 hook，但
  这是项目级别决策，不在本 skill 范围

### 18.5 Mode B（跨分支主动拉取）暂不实现

- **Trade-off**：少一个 entry point，可能让"看下另一个分支进度"场景
  绕路（用户得手动切分支或先 merge）
- **当前选择**：B 与 A 技术同源，先证明 A+C 的设计可行再追加
- **触发回调条件**：用户实际反馈"需要不切分支看 theirs 进度"足够频繁

### 18.6 Sequencing Layer 2（pre-check）保持 soft warn 而非硬阻止

- **Trade-off**：硬阻止能强保证顺序，但卡住用户的反向工作流
- **当前选择**：soft warn + 默认推荐先解决代码，把决策权留给用户
- **触发回调条件**：若实际发现 soft warn 经常被忽略导致 PROGRESS.md
  描述与代码状态不符，考虑升级为硬阻止
