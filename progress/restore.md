# /progress restore

## Command

Restore work session, sync remote progress, provide context recall, and start development environment.

## Execution Flow

### Step 1: Check Local Status

- Execute `git status`.
- **If local is dirty (has uncommitted changes)**:
  - Output warning: "⚠️ Local has uncommitted changes,建议 first `/progress checkpoint` to handle."
  - Abort process, let user decide.

### Step 2: Check Remote Progress Updates

- Execute `git fetch origin`.
- Compare local and remote `PROGRESS.md`.
- **If remote has updates**:
  - Display remote PROGRESS.md diff summary.
  - Ask: "Remote progress has updates, execute `git pull --rebase`?"
  - Execute pull after user confirmation.
- **If remote has no updates**: Continue to next step.

### Step 3: Read Multi-dimensional Context

Read the following content to build recovery context:

1. **`PROGRESS.md`**: Focus on 🎯, 📥, ⚡ sections, especially identify WIP status tasks.
2. **Recent commits**: `git log -3 --stat` (understand last exit position).
3. **WIP task detection**: Analyze WIP tags in recent commits, identify incomplete features.
4. **Latest design documents**:
   - **Whitelist directories**: Only search in `docs/`, `designs/`, `specs/`, `doc/`, `arch/` and project root directory.
   - **Blacklist exclusion**: Ignore `CHANGELOG.md`, `README.md`, `LICENSE.md`, etc.
   - **Time window**: `.md` files modified within 7 days, read their filenames and summaries.

### Step 4: Output Recovery Report and Start Environment

Structured output context, and interactively start environment:

```
🚀 **Work Session Recovery Report**

📍 **Last Exit Position**
- Recent commit: `<hash>` <message>
- Changed files: <list>

🎯 **Current Focus**
- [Extracted from 🎯, specially marked WIP tasks]

📥 **Next Steps**
- [Extracted from 📥]

💡 **Related Design Documents**
- <file_path> (modified on <date>)

🛠 **Prepare to Start Development Environment**
- Parse commands in `⚡ Quick Recovery`, infer if none (based on Makefile/package.json, etc.).
- If parsing fails or no command can be inferred, prompt user: "No startup command detected, please manually start the development environment or update the ⚡ Quick Recovery section in PROGRESS.md."
- If there is an inferred command, ask: "Execute `[inferred/extracted command]`?"
- **Auto-writeback**: If user corrects the startup command, automatically update it to the `⚡ Quick Recovery` section of `PROGRESS.md` for permanent memory.

🔄 **WIP Task Recovery**
- Identify and list all WIP status tasks
- Provide options for quick WIP environment recovery
```
