---
name: bug-report
description: Create a structured bug report in Notion with reproduction steps, expected/actual behaviour, and fix criteria.
---

# Bug Report

Creates a structured bug report page in a Notion tasks database. Formats the report with reproduction steps, expected/actual behaviour, and fix criteria. Maps properties schema-agnostically.

## Workflow

1. **Gather bug information:**
   - Title (required) — a clear summary of the bug
   - What's broken (required) — description of the problem
   - Steps to reproduce (if known)
   - Expected behaviour
   - Actual behaviour
   - Severity/priority (if mentioned)
   - Any relevant context (error messages, screenshots description, affected area)

2. **Resolve the target database:**
   - Check memory for `notion_db:tasks` or `notion_db:bugs`
   - If not in memory: search for databases that could hold bugs (tasks, bugs, issues)
   - Once found, silently `store_memory`

3. **Discover schema and map properties:**
   - Retrieve database schema
   - Map available properties:
     - Title -> title property
     - Status -> first "to do" or "backlog" option (initial state for new bugs)
     - Priority/Severity -> priority-like select property, match user's stated priority
     - Tags/Type/Category -> look for multi_select or select, set to "Bug" if that option exists
     - Assignee -> leave empty unless user specifies
   - Skip properties that don't exist — never error on missing properties

4. **Build the bug report page body as Notion blocks:**
   - Always create a proper Notion block sequence. Never write the section outline as raw markdown or plain text.
   - `Description` → `heading_2` block, then one or more `paragraph` blocks with the core problem statement
   - `Steps to Reproduce` → `heading_2` block, then `numbered_list_item` blocks for each step
   - `Expected Behaviour` → `heading_2` block, then `paragraph` blocks
   - `Actual Behaviour` → `heading_2` block, then `paragraph` blocks
   - `Fix Criteria` → `heading_2` block, then one `to_do` block per criterion
   - `Additional Context` → `heading_2` block, then paragraph, quote, or code blocks for logs, errors, or other supporting details
   - Omit sections that have no content

5. **Create the page:**
   - Call `mcp__notionApi` Create page in the target database
   - Set properties from step 3
   - Set `children` from the block array in step 4

6. **Store to memory (silent):**
   - `store_memory` the page ID: `notion_page:{title}`
   - `store_memory` the database: `notion_db:tasks`

7. **Report:** Filed: {title} | Database: {db_name} | Status: {status} | Priority: {priority}

## Important

- **Schema-agnostic.** Discover properties from the schema. If there's no "Bug" tag option, just skip the tag — don't fail.
- **Be autonomous.** If the user provides enough info to file the bug, just file it. Don't ask for every field — a title and description are sufficient.
- **Structured format.** Always use the structured sections (Description, Steps, Expected, Actual, Fix Criteria) when information is available. This makes bugs actionable.
- **Block-first writes.** Section headings, reproduction steps, and fix criteria must be written as proper Notion blocks, not literal markdown like `##` or `- [ ]`.
- **Don't over-prompt.** If the user gives you a vague bug report ("login is broken"), create the bug with what you have. They can update it later with `update-task`.
- **Priority defaults.** If the user doesn't mention priority, don't set one (or use the database's default). Don't assume Medium.
- **Fix Criteria vs Acceptance Criteria.** For bugs, use "Fix Criteria" as the section name with checkbox items describing what "fixed" means.
