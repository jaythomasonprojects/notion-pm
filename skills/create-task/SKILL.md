---
name: create-task
description: Create a new task in a Notion database. Schema-agnostic — discovers and maps properties dynamically.
---

# Create Task

Creates a new task page in a Notion tasks database. Discovers the database schema dynamically and maps user-provided information to available properties — never assumes a fixed schema.

## Workflow

1. **Gather task information from the user or context:**
   - Title (required)
   - Description / body content (required — may come from a CLI plan, conversation, or user input)
   - Any properties the user mentions: status, priority, tags, assignee, due date, estimate, project, etc.

2. **Resolve the target database:**
   - Check memory for `notion_db:tasks` or similar task-database keys
   - If not in memory: search Notion for databases with task-like names (tasks, backlog, work, todos, issues)
   - If multiple candidates: pick the most likely one (prefer databases with status/priority-like properties)
   - Only ask the user if you genuinely cannot determine which database to use
   - Once resolved, silently `store_memory` as `notion_db:tasks` and `notion_db:{actual_name}`

3. **Discover the database schema:**
   - Retrieve the database via `mcp__notionApi` to get its property definitions
   - Identify available properties and their types:
     - title property (for the task name)
     - status property (look for select/status type with values like todo/backlog/in-progress/done)
     - priority property (select with low/medium/high or similar)
     - tags/labels (multi_select)
     - assignee/person (people type)
     - due/date (date type)
     - estimate/size (select with XS/S/M/L/XL or similar)
     - project/category (relation or select)
   - Map user-provided values to the discovered properties

4. **Set initial property values:**
   - Title: from user
   - Status: first option in the "to do" or "not started" group (discover from schema). If no groups, use the first option.
   - Other properties: set if the user provided values AND the property exists in the schema. Skip silently if a property doesn't exist.

5. **Format the page body:**
   - If the user provides a CLI plan or implementation: use it as the page body, formatted for Notion blocks
   - If the user provides a description: structure it as:
     ```
     ## Description
     {description}

     ## Acceptance Criteria
     - [ ] {criteria if provided, or leave section empty}
     ```
   - If pushing a full CLI plan as a task: keep the plan structure intact — headings, lists, code blocks all convert to Notion blocks
   - Don't force sections that don't have content

6. **Create the page:**
   - Call `mcp__notionApi` Create page with:
     - parent: the database ID
     - properties: mapped from step 4
     - children: the formatted content blocks

7. **Store to memory (silent):**
   - `store_memory` the new page ID: `notion_page:{title}`
   - If the page gets an auto-increment ID, store: `notion_task:{ID}`
   - Ensure database ID is stored: `notion_db:tasks`

8. **Report:**
   ```
   ✅ Created: {title}
      Database: {database_name} | Status: {status} | Priority: {priority}
   ```
   Only show properties that were actually set.

## Important

- **Schema-agnostic.** Never hardcode property names or assume specific values exist. Discover everything from the database schema.
- **Be autonomous.** Don't ask the user for properties they didn't mention — just create the task with what you have. Minimal viable task = title + body.
- **CLI plan as task body.** When the user says "upload this plan as a task" or "create a task from the plan", take the current plan content and use it directly as the page body. Format it for Notion but keep the structure.
- **Duplicate check.** Before creating, search the database for tasks with similar titles. If a very close match exists, mention it but still create unless the user said "update" (in which case, use `update-task` or `push-page` instead).
- **Don't force a project relation.** If the database has a project property and the user mentioned a project, set it. Otherwise skip it.
- **Property value mapping.** When the user says "high priority", find the priority property and look for a value containing "high" (case-insensitive). Same for status, tags, etc.
