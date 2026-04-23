---
name: push-page
description: Push or update content to a Notion page. Upserts by title — updates existing pages, creates new ones if not found.
---

# Push Page

Pushes content from the CLI to Notion. If a page with the same title already exists in the target database, it updates that page. Otherwise, it creates a new one. Use this for syncing plans, research notes, implementation details, or any content back to Notion.

## Workflow

1. **Determine what to push:**
   - Content from the current CLI plan (plan.md or conversation context)
   - Content the user provides or describes
   - A specific section or summary of work done

2. **Determine the target:**
   - If the user specifies a page name: use that for the upsert lookup
   - If the user specifies a database: use that as the target database
   - If neither: check memory for `notion_db:default` or recently used databases
   - If still ambiguous: ask the user which database to target (this is the ONE case where asking is appropriate)

3. **Resolve the target database:**
   - Check memory for `notion_db:{name}`
   - If not in memory, search Notion for databases matching the user's description
   - Once found, silently `store_memory` as `notion_db:{name}`

4. **Convert CLI markdown or prose to Notion blocks:**
   - Before any create, replace, or append write, construct a `children` array of Notion block objects. Never send raw markdown, HTML, or plain-text boilerplate directly as page content.
   - `# Heading` → heading_1 block
   - `## Heading` → heading_2 block
   - `### Heading` → heading_3 block
   - Paragraphs and plain prose → paragraph blocks
   - Blank lines separate blocks; they are not written literally
   - `- item` → bulleted_list_item blocks
   - `1. item` → numbered_list_item blocks
   - `- [ ] item` / `- [x] item` → to_do blocks (checked/unchecked)
   - ``` code ``` → code blocks (carry language when it is known)
   - `> quote` → quote blocks
   - Horizontal rules → divider blocks
   - If the source contains section labels or boilerplate, translate them into the right heading/paragraph/list blocks instead of leaving markdown markers visible in Notion
   - If the user explicitly wants raw text preserved, place it intentionally inside paragraph or code blocks

5. **Upsert logic:**
   - Search the target database for a page with a matching title (case-insensitive)
   - **If found:** update the existing page content
     - Read the existing block children before changing content
     - If the user asked to append to a named section: find that heading and insert the prepared blocks before the next heading. If the heading is missing, append the prepared blocks to the end of the page instead
     - If the user asked to append but did not name a section: append the prepared blocks to the end of the page
     - Otherwise replace the page body with the prepared block array (clear existing blocks, then write the replacement blocks)
     - Preserve page properties unless the user explicitly wants to change them
   - **If not found:** create a new page in the target database
     - Set title from the content heading or user-specified name
     - Write the prepared block array as the page body

6. **Store to memory (silent):**
   - `store_memory` the page ID: `notion_page:{title}`
   - `store_memory` the database ID: `notion_db:{database_name}`

7. **Report (one line):**
   ```
   ✅ Pushed "{title}" to {database_name} (updated existing page)
   ```
   or:
   ```
   ✅ Pushed "{title}" to {database_name} (created new page)
   ```

## Important

- **Upsert is the default.** Never create a duplicate when a page with the same title exists in the target database.
- **Be autonomous about database selection.** Only ask the user if you genuinely cannot determine the target from context or memory.
- **Content replacement vs append:** Default is full content replacement on update. If the user says "append" or "add to", append to existing content instead.
- **Format for Notion, not CLI.** The content may come from CLI plan mode — convert it cleanly to Notion's block structure. Don't leave raw markdown artefacts.
- **Block-first writes.** Every create, replace, or append operation must write proper Notion blocks. Never pass markdown-shaped text straight into `children`.
- **Large content:** Notion has a limit of ~100 blocks per API call. If content is very large, batch the block creation across multiple calls.
- **Properties:** When creating a new page, set whatever properties make sense from context (e.g., if pushing to a tasks database and content mentions a status). Don't force properties that don't exist in the schema.
