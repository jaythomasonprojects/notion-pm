---
name: list-tasks
description: List and filter tasks from a Notion database. Schema-agnostic — discovers filterable properties dynamically.
---

# List Tasks

Lists tasks from a Notion database with optional filtering. Discovers available properties dynamically and maps user filter requests to them.

## Workflow

1. **Resolve the tasks database:**
   - Check memory for `notion_db:tasks`
   - If not in memory: search Notion for databases with task-like names
   - Once found, silently `store_memory` as `notion_db:tasks`

2. **Discover schema for filtering:**
   - Retrieve the database schema to identify filterable properties
   - Common filter targets: status, priority, assignee, tags, due date, project/category
   - Map user filter requests to actual property names and types

3. **Determine filters from user request:**
   - "show in-progress tasks" -> filter status property for "In Progress" or similar
   - "list high priority bugs" -> filter priority for "High" AND tags/type for "Bug"
   - "what's due this week" -> filter date property for this week's range
   - "my tasks" -> filter assignee for the current user (if identifiable)
   - No filters specified -> show all non-completed tasks (exclude "Done"/"Complete"/"Archived" status options)

4. **Query the database:**
   - Call `mcp__notionApi` Query database with constructed filters
   - Sort by: priority (if exists) descending, then due date ascending, then created time descending
   - Limit to 50 results max

5. **Display as a compact table:**
   - Include columns for properties that exist and have values
   - If the database has an auto-increment ID property, show it as the first column
   - Keep rows compact — truncate long titles
   - Show a summary line with count and active filters

6. **Store to memory (silent):**
   - `store_memory` the database ID: `notion_db:tasks`

## Important

- **Schema-agnostic filtering.** Don't assume property names. Discover them from the schema and match user intent.
- **Smart defaults.** If no filter specified, exclude completed/archived tasks. The user wants to see active work.
- **Be autonomous.** Don't ask "what would you like to filter by?" — if the user just says "list tasks", show them all active tasks.
- **Compact output.** This is a CLI — keep the table tight. No verbose descriptions in the list view.
- **If the database is empty or no results match**, say so clearly in one line.
- **Property value matching.** When the user says "in progress", find the status property and match against options case-insensitively. Handle variations: "in-progress", "In Progress", "in_progress", "WIP".
