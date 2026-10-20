# agent-skills

通用 AI Agent skills 集合，支持 Claude Code、OpenCode 等主流 Agent。

[English Version](README.md)

## 安装

### 安装所有技能

```bash
npx skills add wuruofan/agent-skills -g -y
```

### 安装特定技能

```bash
# 安装 progressing 技能
npx skills add wuruofan/agent-skills --skill progressing -g -y

# 安装 web-fetch-as-markdown 技能
npx skills add wuruofan/agent-skills --skill web-fetch-as-markdown -g -y
```

## 收录的 Skills

### web-fetch-as-markdown

将网页 URL 转换为干净的 Markdown，剥离广告、导航栏和干扰内容。

**使用场景：**
- 绕过 Agent 的域名安全检查
- 将网页转换为易解析的干净 markdown
- 遇到 "Unable to verify if domain" 错误时

**转换服务：**
1. `https://markdown.new/` - 首选（Cloudflare，优化 token 消耗）
2. `https://r.jina.ai/<url>` - 备用 1（Jina AI Reader API）
3. `https://markdownforagents.com/r?url=<url>` - 备用 2（需用户授权）

**触发词：** "上网搜索"、"帮我查一下"、"fetch 网页"、"帮我解析"

---

### Progressing

一个 todo，但带普通 todo 没有的三件法宝 —— git 自动对账、显式关闭条件、读取时强制复审过期条目。一个 `PROGRESS.md` 在项目根目录就够了；git 已经替你跨设备同步，不必另维护外部归档。

**它做什么：**

| 场景 | 动作 |
|---|---|
| 新会话 / 换机器 / "接着干 / 继续 / 之前干到哪了" | Load —— 读 PROGRESS.md，与 git 对账，列出过期条目，输出恢复报告 |
| 本会话内某条 `Done when:` 已满足 | Close —— 当场删条目，刷新时间戳 |
| 会话中途中断 | Save —— 写未完成状态。强指定可用 `/progressing save` |

**什么时候不要用：** 常规 commit、没有任何未完成工作时 —— 如果没有"进行中"的事，就没什么可保留。

**PROGRESS.md 格式：**

```markdown
# Progress · managed by `progressing` skill

> Last updated: 2026-10-20

> 一条目一行；详情在各自 spec/plan/debug 文件（见 `Detail:`）。手动编辑会破坏格式契约 —— 改完请跑格式闸门。

## Open
- [2026-10-20] eager terminal session — 落地完成，待真机验证. Done when: `bun run test:gateway` 全绿. Detail: docs/specs/2026-10-13-eager-terminal-session.md

## Verify
- [2026-10-19] ink CR 列漂移修复 — 提交 `04139ea0`. Awaiting: user TTY check

## Paused
- [2026-09-01] Tool/Thinking display — 阻塞于 v0.7.1. Restart: docs/specs/tool-display.md
```

一个条目 = 一行。表头是纯日期。固定三个 section。每条目以 `Done when:` / `Awaiting:` / `Restart:` 结尾；`Detail: <path>` 指向存放详情的文件。

H1 承载 managed-by 标记，其下一行短 `>` 承载格式说明——两处都会渲染，所以打开文件的人能看见谁在管它。闸门只允许 `>` 行是日期头或 ≤ 200 字符的 managed-by / 格式说明，表头因此回退成变更日志。

**安装命令要写进项目的 AGENTS.md，不要写进台账。** 没装这个 skill 的 agent 根本不会打开 `PROGRESS.md`，那里的命令谁也看不到；而 `AGENTS.md` 是每个 agent 每次会话都会读的。

约束：条目 ≤ 256 字符；`## Open` ≤ 15 条、`## Verify` ≤ 10 条、`## Paused` ≤ 10 条；文件 ≤ 60 行且 ≤ 12,000 字符。默认 `STALE_DAYS=7`。

光有行数约束并不够——一行也能塞进 2 万字符，而且臃肿的台账每次会话都要整读一遍。**每次**写入 PROGRESS.md 后（load / close / save / 会话中临时改动）都要机械校验格式；命令内嵌在 skill 里，写入生效前必须 exit 0：

```bash
# 运行 skills/progressing/SKILL.md 内嵌的 gate 代码块
```

gate 不过意味着：把详情搬进它描述的那个文件，用 `Detail:` 链过去；把撑爆配额的条目关闭、确认或归档。

**推荐 AGENTS.md 配置：**

> **重要**：以下内容必须安装到项目的 AGENTS.md / CLAUDE.md / GEMINI.md 中，AI 时序约束才能生效。

```markdown
## 进度追踪（重要）

项目根目录一个 PROGRESS.md，不是每次提交的仪式。按上下文自动选动作：
- 新会话 / 回到项目 / "接着干 / 继续 / 之前干到哪了" / 换机器 → `/progressing`（load）
- 本会话内某条 `Done when:` 已满足 → `/progressing`（close）
- 会话中途中断（断网、切换机器、显式交接、压缩前需保留）→ `/progressing`（save）或 `/progressing save`

> **换机器 / 新环境**：本仓库的 `PROGRESS.md` 由 `progressing` skill 管理。若该 skill 未安装，先装：
> ```bash
> npx skills add wuruofan/agent-skills --skill progressing -g -y
> ```
> 未安装时，`PROGRESS.md` 里的条目仍可读（纯 markdown），只是失去自动对账与闭合检查。

### PROGRESS.md 保持小体量（这是它能整读的前提）
PROGRESS.md 每次 load 都会完整重读，体积是每个会话都在付的成本。
一个条目 = 一行；表头是纯日期；详情放在 spec/plan/debug 文件里，用 `Detail: <path>` 链过去。
**每次写入 PROGRESS.md 后都要跑格式闸门** —— 不只是 load 和 save 时。「基本都跑」的闸门等于没跑。

### 读取项目状态文件（防 thrashing）
读取可能较大的项目文档时（CLAUDE.md、AGENTS.md、spec、plan、handoff 等）：
- 文件 > 300 行时，绝不带 `offset`/`limit` 调 Read —— 先读前 50 行（frontmatter + section 标题），再按行段定向读取需要的具体 section
- 多文件场景（多个 AGENTS.md / 子目录 CLAUDE.md）：按每次 Read 摄入总量预算，而不是按单文件大小 —— 一批中等文件比一个大文件更容易触发 autocompact thrashing
```

让 LLM 在合适时机自主触发技能。PROGRESS.md 设计上保持小体量（≤ 60 行），可整读。

---

更多 skill 持续添加中。
