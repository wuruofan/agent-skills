# /progress show

## Command

Read the current project's `PROGRESS.md`, extract key information and format the output.

## Execution Flow

1. Locate `PROGRESS.md`.
2. Check if the file exists:
   - If it doesn't exist: Prompt to run `/progress checkpoint` to initialize.
   - If it exists and old format detected:
     - Create a backup: `PROGRESS.md.bak.<timestamp>`
     - Convert old content to new structure
     - Inform user: "PROGRESS.md has been upgraded to the new format. A backup has been created at PROGRESS.md.bak.<timestamp>."
   - If already using new format: Proceed normally
3. Read and parse the file, extracting the following sections:
   - 🎯 Current Focus tasks
   - 📥 Todo Queue (first 5 items)
   - ✅ Recently Completed task count
   - 📅 Task History (last 3 days)
   - ⚡ Quick Recovery steps
   - 🏛️ Archive Links

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
