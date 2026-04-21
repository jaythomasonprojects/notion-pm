---
name: update-task
description: Update a task's properties or content in Notion. Schema-agnostic, appends to content by default.
---

# Update Task

Updates an existing task's properties or page content. Discovers the schema dynamically and maps user intent to available properties. Content updates append by default — never overwrites unless explicitly asked.

## Workflow

1. **Resolve the task:**
   - Accept: task name, page ID, auto-increment ID (e.g., "TASK-42"), or search term
   - Check memory first: `notion_task:{id}` or `notion_page:{name}`
   - If not in memory: search the tasks database (from memory `notion_db:tasks` or discover it)
   - If multiple matches: pick the most likely one. Ask only if genuinely ambiguous.

2. **Fetch current state:**
   - Retrieve page properties via `mcp__notionApi`
   - Retrieve page content via block children (if content update is needed)
   - Never make blind writes — always read first

3. **Determine what to update:**
   - **Property changes:** status, priority, tags, assignee, due date, estimate, etc.
   - **Content changes:** append notes, update a section, check/uncheck items
   - **Mixed:** both properties and content in one operation

4. **Discover schema (if property update):**
   - Retrieve the database schema to understand available properties and their valid values
   - Map user intent to available options (e.g., "mark it high priority" -> find priority select -> find "High" option)
   - If the user requests a property that doesn't exist, skip silently (don't error)

5. **Apply property changes:**
   - Call `mcp__notionApi` Update page properties with the mapped values
   - For status changes: find the matching option in the status property (handle both select and status types)
   - For relation changes: resolve the related page ID first

6. **Apply content changes:**
   - First convert any user-provided notes, summaries, checklists, plans, or prose into the right Notion block sequence (paragraph, heading, bulleted list, numbered list, to_do, quote, code, divider, and so on).
   - **Append (default):** Add the new block sequence after existing content. If adding to a specific section (e.g., "add to implementation notes"), find that heading and insert the new blocks before the next heading. If the heading is missing, append to the end of the page.
   - **Replace section:** If the user says "replace the description" or "update the plan", find the section heading and replace the blocks between it and the next heading with the newly constructed block sequence.
   - **Check/uncheck to-do items:** Find the specific to_do block and update its checked state.
   - **Never replace the entire page content** unless the user explicitly says so.
   - Never append literal markdown markers, section boilerplate, or fenced code blocks as plain text unless the user explicitly wants raw text.

7. **Store to memory (silent):**
   - `store_memory` the page ID: `notion_page:{title}` and `notion_task:{id}` if applicable
   - `store_memory` the database ID if discovered: `notion_db:tasks`

8. **Report:**
   Show what changed in one or two lines. Only show what actually changed.

## Important

- **Read before write.** Always fetch current state before making changes.
- **Append by default.** Content updates add to existing content. Replacing requires explicit user instruction.
- **Schema-agnostic.** Discover properties dynamically. Don't assume property names or values.
- **Be confident.** If the user says "update task X to done", just do it. Don't ask for confirmation on simple property changes.
- **Batch updates.** If the user wants multiple property changes, do them all in one API call.
- **Block-first updates.** Any added or replaced content must be written as proper Notion blocks, not markdown strings.
- **Section detection.** When appending to a section, find heading blocks (heading_2, heading_3) by text content to locate the right position.
- **If the task doesn't exist**, say so. Don't create a new one — that's what `create-task` is for.
