---
name: complete-task
description: Mark a task as done/complete. Discovers the completion status dynamically from the database schema.
---

# Complete Task

Marks a task as done/complete in Notion. Discovers the "done" or "complete" status option from the database schema rather than assuming a fixed value.

## Workflow

1. **Resolve the task:**
   - Accept: task name, page ID, auto-increment ID (e.g., "TASK-42"), or search term
   - Check memory first: `notion_task:{id}` or `notion_page:{name}`
   - If not in memory: search the tasks database
   - If multiple matches: pick the most likely one. Ask only if genuinely ambiguous.

2. **Fetch the task:**
   - Retrieve page properties
   - Retrieve page content (to report completion state)

3. **Discover the "done" status:**
   - Retrieve the database schema
   - Find the status property (type: status or select)
   - For status type: look for an option in the "complete" group, or one named "Done", "Complete", "Completed", "Finished" (case-insensitive)
   - For select type: look for an option containing "done", "complete", "finished" (case-insensitive)
   - If no obvious completion status exists: ask the user which option represents "done"

4. **Set completion:**
   - Update the status property to the discovered "done" option
   - If a "completed on" or "completed date" or "done date" property exists (date type): set it to today's date
   - If the user provides a summary: append it to the page as a "## Final Summary" section

5. **Report completion state:**
   - If the page has to-do/checkbox items: report their state
   - Example: Completed: {title} | Status: {old} -> Done | Checklist: 4/5 items checked
   - If no checklists: Completed: {title} | Status: {old} -> Done | Completed on: {date}

6. **Store to memory (silent):**
   - Update stored page info: `notion_page:{title}`, `notion_task:{id}`
   - Ensure database stored: `notion_db:tasks`

## Important

- **Don't block on unchecked items.** Report the checklist state but still complete the task. The user explicitly asked to complete it — respect that.
- **Schema-agnostic.** Discover the "done" option dynamically. Never hardcode "Done" as the value.
- **Be confident.** If the user says "complete task X", do it. Don't ask "are you sure?" or validate acceptance criteria.
- **Final summary is optional.** Only append one if the user provides summary content. Don't prompt for it.
- **Completion date.** Only set if a date property clearly represents completion exists. Don't create properties.
- **If already done:** Tell the user it's already marked complete. Don't error.
