# Notion Skills

Reusable Notion skills for GitHub Copilot CLI. Pull/push pages, manage tasks, file bugs — from any agent.

No dedicated agent required. Install the plugin and any agent can invoke these skills to interact with your Notion workspace.

## Setup

### 1. Notion Integration

Create a [Notion integration](https://www.notion.so/my-integrations) and give it access to the databases you want to work with.

### 2. Environment Variable

Set the `NOTION_TOKEN` environment variable to your integration's API key:

```sh
export NOTION_TOKEN="ntn_..."
```

### 3. Install the Plugin

```sh
copilot plugin install jaythomasonprojects/notion-skills
```

The plugin includes the Notion MCP server — no additional MCP configuration needed.

### 4. Usage

No setup wizard. Skills discover databases and pages autonomously on first use, and memorise IDs for future sessions. Just ask any agent to do Notion things:

- *"Pull down the auth redesign plan from Notion"*
- *"Push this plan to Notion"*
- *"Create a task for the login refactor"*
- *"List my in-progress tasks"*
- *"File a bug: the API returns 500 on empty payload"*

## Skills

| Skill | What it does |
| --- | --- |
| `pull-page` | Pull any Notion page into CLI context as clean markdown |
| `push-page` | Push/upsert content to a Notion page (plans, research, notes) |
| `create-task` | Create a new task — schema-agnostic property mapping |
| `update-task` | Update task properties or content (append by default) |
| `complete-task` | Mark a task as done — discovers completion status dynamically |
| `list-tasks` | List/filter tasks with compact table output |
| `bug-report` | File a structured bug report with reproduction steps |

## Design Principles

- **Any agent, any time.** No dedicated agent needed — works with your main agent, anvil, or any custom agent.
- **Schema-agnostic.** Skills discover database properties dynamically. Works with any Notion database structure.
- **Autonomous.** Silently memorizes database/page IDs. Searches Notion when needed. Only asks you when genuinely stuck.
- **Block-first writes.** When skills push or add content, they should convert it into real Notion blocks rather than leaving raw markdown markers in the page.
- **Push/pull model.** Pull pages into CLI context. Push CLI content back to Notion. Upserts by title — updates existing pages, creates new ones.
- **Minimal interruption.** Confident on routine operations. No confirmation dialogs for simple changes.
