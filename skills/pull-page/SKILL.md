---
name: pull-page
description: Pull a page from Notion into the current CLI context. Retrieves properties and content as clean markdown.
---

# Pull Page

Pulls a Notion page into your CLI context — properties and full content rendered as markdown. Works with any page: plans, tasks, research notes, documentation.

## Workflow

1. **Resolve the page.** Accept any of:
   - A page name or title (e.g., "Auth redesign plan")
   - A Notion page ID or URL
   - A task auto-increment ID (e.g., "TASK-42") if working with a tasks database
   - A search term

2. **Memory-first resolution:**
   - Check `store_memory` for a matching key: `notion_page:{name}` or `notion_task:{id}`
   - If found, use the stored page ID directly — skip search

3. **Search fallback (if not in memory):**
   - Call `mcp__notionApi` search with the page name/term
   - If exactly one match: use it
   - If multiple matches: pick the most likely one based on context (title similarity, recency). Only ask the user if genuinely ambiguous (e.g., two pages with near-identical titles)
   - If no matches: tell the user — don't prompt them to try something else unless they ask

4. **Fetch the page:**
   - Retrieve page properties via `mcp__notionApi` Retrieve page
   - Retrieve page content via `mcp__notionApi` Retrieve block children (recurse into nested blocks)

5. **Format as markdown:**
   - Start with page title as `# Title`
   - Render properties as a compact metadata block (skip empty/null properties):
     ```
     **Status**: In Progress | **Priority**: High | **Due**: 2026-04-25
     ```
   - Render page content as standard markdown:
     - Headings → `##`, `###`
     - Paragraphs → text
     - Bulleted lists → `- item`
     - Numbered lists → `1. item`
     - To-do items → `- [x]` or `- [ ]`
     - Code blocks → fenced code blocks
     - Callouts → blockquotes
   - Keep it clean and readable — this is meant for CLI consumption

6. **Store to memory (silent):**
   - `store_memory` the page ID with key pattern `notion_page:{normalized_title}`
   - If the page has an auto-increment ID property, also store as `notion_task:{ID}`
   - If you discovered the database this page belongs to, store that too: `notion_db:{database_name}`

## Important

- **Be autonomous.** Don't ask the user to clarify unless you truly cannot determine which page they mean.
- **Render content faithfully** but keep it concise. If a page has 100+ blocks, summarise very long sections and note "[truncated — X more blocks]".
- **Nested blocks:** Recurse into child blocks (toggles, columns, synced blocks). Indent nested content appropriately.
- **If the page doesn't exist**, say so clearly. Don't suggest alternatives unless asked.
- **Database pages vs standalone pages:** Handle both. Database pages have properties — render them. Standalone pages may not — that's fine.
