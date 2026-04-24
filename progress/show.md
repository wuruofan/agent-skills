# /progress show

## Command

Read the current project's `PROGRESS.md`, extract key information and format the output.

## Execution Flow

1. Locate and read `PROGRESS.md`.
2. Parse the file, extracting the following sections:
   - 🎯 Current Focus tasks
   - 📥 Todo Queue (first 5 items)
   - ✅ Recently Completed task count
   - 📅 Task History (last 3 days)
   - ⚡ Quick Recovery steps
   - 🏛️ Archive Links
3. If the file does not exist, prompt to run `/progress checkpoint` to initialize.

## Output Format

```
📊 **Project Progress Overview**
- **Current Focus**: [List tasks under 🎯]
- **Next Steps**: [List first items under 📥]
- **Recently Completed**: [count] items
- **Last 3 Days Tasks**: [List last 3 days under 📅]
- **Recovery Guide**: [List steps under ⚡]
- **Archive Links**: [List recent links under 🏛️]
```
