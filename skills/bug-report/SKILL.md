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

   Never write section outlines as raw markdown or plain text — always construct proper Notion block sequences. Omit any section where content is unavailable.

   **Always include:**
   - `Description` — `heading_2` block, then one or more `paragraph` blocks with the core problem statement; for exceptions, include `{ErrorType}: {message}` and the culprit line
   - `Fix Criteria` — `heading_2` block, then one `to_do` block per criterion; tailor items to context

   **Include when available:**
   - `Steps to Reproduce` — `heading_2` block, then `numbered_list_item` blocks; omit for exception-based bugs where a stack trace is more useful
   - `Stack Trace` — `heading_2` block, then a `code` block with the top 5 application-layer frames; omit for ad-hoc reports without trace data
   - `Expected Behaviour` — `heading_2` block, then `paragraph` blocks; omit when self-evident from the exception
   - `Actual Behaviour` — `heading_2` block, then `paragraph` blocks; omit when self-evident from the exception
   - `Affected Files / Components` — `heading_2` block, then `bulleted_list_item` blocks for each path; omit if unknown
   - `Triage` — `heading_2` block, then `paragraph` blocks for category and severity, each with a one-sentence rationale; include even for ad-hoc bugs when context allows a reasonable determination
   - `Source` — `heading_2` block, then `paragraph` or `bulleted_list_item` blocks for provenance metadata (Sentry: issue ID, URL, first/last seen, event count, users affected; ad-hoc: environment, reporter, version, or other relevant context)

   Section order: Description → Steps to Reproduce → Stack Trace → Expected Behaviour → Actual Behaviour → Affected Files / Components → Triage → Source → Fix Criteria

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
- **Structured format.** Use the unified section order: Description → Steps to Reproduce → Stack Trace → Expected Behaviour → Actual Behaviour → Affected Files / Components → Triage → Source → Fix Criteria. Only include sections where content is available; never leave a section empty.
- **Steps vs Stack Trace.** These are parallel, not interchangeable. Steps apply to ad-hoc reports; Stack Trace applies to exception-based bugs. Both can appear together if both are available.
- **Block-first writes.** Section headings, reproduction steps, and fix criteria must be written as proper Notion blocks, not literal markdown like `##` or `- [ ]`.
- **Don't over-prompt.** If the user gives you a vague bug report ("login is broken"), create the bug with what you have. They can update it later with `update-task`.
- **Priority defaults.** If the user doesn't mention priority, don't set one (or use the database's default). Don't assume Medium.
- **Fix Criteria vs Acceptance Criteria.** For bugs, always use "Fix Criteria" as the section name.
