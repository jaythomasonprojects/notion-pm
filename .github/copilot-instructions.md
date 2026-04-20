# Copilot Instructions

## Language

Use Australian English in any new or edited prose.

## Commands

No build, test, or lint commands are defined in this repository. Do not invent `npm`, `pytest`, or similar workflows.

## Architecture

This is a content-driven GitHub Copilot plugin for Notion workflows, not an application codebase.

- `plugin.json` is the entrypoint and points Copilot at `skills/`, `hooks.json`, and `.mcp.json`.
- `.mcp.json` wires in the Notion MCP server via `npx @notionhq/notion-mcp-server` and expects `NOTION_TOKEN` in the environment.
- Each skill lives in `skills/<skill-name>/SKILL.md`; those markdown files are the product.
- Keep `README.md` aligned with the installed plugin story, available skills, and design principles.

## Key Conventions

- Keep skills **schema-agnostic**. The existing skills never hardcode Notion property names or status values; they instruct Copilot to discover database schema and map user intent dynamically.
- Prefer **memory-first resolution**. Skills check stored IDs like `notion_db:*`, `notion_page:*`, and `notion_task:*` before searching Notion again.
- Be **autonomous by default**. The skills are written to avoid unnecessary prompting: ask the user only when the target page/database is genuinely ambiguous or a required concept cannot be inferred.
- Preserve each skill's write semantics:
  - `update-task` reads before writing and appends content by default
  - `push-page` upserts by title and replaces page content on update unless the user asked to append
  - `complete-task` discovers the done state dynamically instead of assuming `"Done"`
- When editing a skill, keep the current structure intact: YAML frontmatter (`name`, `description`), a workflow, and an `Important` section with behavioural guardrails.
- Keep outputs optimised for CLI use: compact summaries, compact tables for list views, and no verbose confirmation chatter unless the skill explicitly needs it.
