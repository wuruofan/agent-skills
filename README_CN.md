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
# 安装 progress 技能
npx skills add wuruofan/agent-skills --skill progress-save -g -y
npx skills add wuruofan/agent-skills --skill progress-restore -g -y
npx skills add wuruofan/agent-skills --skill progress-archive -g -y
npx skills add wuruofan/agent-skills --skill progress-summary -g -y
npx skills add wuruofan/agent-skills --skill progress-merge -g -y

# 安装 web-fetch-as-markdown 技能
npx skills add wuruofan/agent-skills --skill web-fetch-as-markdown -g -y

# 安装多个技能
npx skills add wuruofan/agent-skills --skill progress-save --skill progress-restore -g -y
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

### Progress 技能套件

跟踪项目开发进度，跨设备保存和恢复工作会话，归档开发历史。

5 个核心技能：

| 技能 | 用途 | 使用时机 |
|------|------|----------|
| `progress-save` | 更新 PROGRESS.md | 提交前、stash 前、切换分支前 |
| `progress-restore` | 恢复会话上下文 | 休息后、新设备、切换分支后 |
| `progress-archive` | 归档历史记录 | 大任务完成时 |
| `progress-summary` | 生成会话摘要 | 新会话继续工作时 |
| `progress-merge` | 跨分支合并 PROGRESS.md | `git merge`/`git rebase`/`git cherry-pick` 后 PROGRESS.md 冲突时 |

**快速选择指南：**
- 要提交代码？ → `/progress-save`
- 回来继续工作？ → `/progress-restore`
- 大任务完成了？ → `/progress-archive`
- 新会话继续工作？ → `/progress-summary`
- 合并分支进度冲突？ → `/progress-merge`

**推荐 AGENTS.md 配置：**

> **重要**：以下内容必须安装到项目的 AGENTS.md / CLAUDE.md / GEMINI.md 中，AI 时序约束才能生效。否则 `/progress-merge` 将降级为较弱的触发方式。

```markdown
## 进度追踪 (重要)

### 场景触发
- 提交/stash/PR 前：调用 `/progress-save` 更新 PROGRESS.md
- 休息后恢复工作：调用 `/progress-restore` 恢复会话上下文
- 大任务或所有阶段完成时：调用 `/progress-archive` 归档历史记录
- 新会话继续之前工作：调用 `/progress-summary` 获取会话上下文
- `git merge` / `git rebase` / `git cherry-pick` 涉及 PROGRESS.md 前（或检测到 PROGRESS.md 冲突标记）：调用 `/progress-merge`
- 任何 git 操作导致 PROGRESS.md 处于冲突状态后：在继续之前调用 `/progress-merge`

### git merge 操作的执行顺序
AI 执行 `git merge` / `git rebase` / `git cherry-pick` 出现冲突时：
1. 先解决非 PROGRESS.md 的代码冲突
2. 验证测试通过或合并正确性确认
3. 然后调用 /progress-merge 处理 PROGRESS.md 冲突
4. 最后 `git add` + 完成合并

### 用户说法触发（防御性补充）
- 用户说"update/save/record PROGRESS.md"、"更新进度"、"保存进度"、"记录进度" → `/progress-save`
- 用户说"restore/resume/load context"、"恢复进度"、"接着干"、"继续上次"、"之前干到哪了" → `/progress-restore`
- 用户说"archive/clean up"、"归档"、"任务完成"、"进度太长" → `/progress-archive`
- 用户说"summarize/recap/handoff"、"总结一下"、"会话总结"、"交接"、"写个摘要" → `/progress-summary`
- 用户说"merge progress"、"合并进度"、"PROGRESS 冲突"、"两个分支的 PROGRESS 怎么合" → `/progress-merge`
```

让 LLM 在合适时机自主触发技能。

**说明：** `/progress-save` 会自动检测已完成的大任务并提示归档建议，也会检测 PROGRESS.md 合并冲突并跳转到 `/progress-merge`。

---

更多 skill 持续添加中。