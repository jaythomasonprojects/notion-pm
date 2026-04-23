---
name: sentry-to-notion
description: "Gathers context from a Sentry issue and creates a Bug task in Notion with initial triage. Use when: triaging a Sentry error, logging a crash to Notion, creating a bug task from a Sentry issue URL or ID, converting a production error into a development backlog item, creating a development task from a crash report or unhandled exception."
argument-hint: "Sentry issue ID (e.g. PROJECT-123) or full Sentry issue URL"
---

# Sentry to Notion Task

Fetches a Sentry issue, does a brief triage pass to identify error category, severity, and affected files, then delegates to the `bug-report` skill.

## Workflow

### 1. Fetch the Sentry issue

Accept input as a full Sentry URL or a short issue ID plus organisation slug. Call `mcp_io_github_get_get_issue_details` and extract:

- **Error type / title**: exception class and short message
- **Culprit**: file/function Sentry identifies as root cause
- **Stack trace**: top 5–10 frames from the most recent event
- **Frequency**: event count and unique users affected
- **First/last seen**: timestamps
- **Tags**: environment, release, or custom tags

### 2. Triage

A quick pass — don't over-analyse.

**Affected files**: scan stack frames, note 3–5 application-layer paths (skip `site-packages`, `node_modules`, stdlib). If a workspace is open, do a quick search to confirm paths and identify component ownership.

**Error category**:

| Category    | When                                       |
| ----------- | ------------------------------------------ |
| `crash`     | Unhandled exception, hard failure          |
| `assertion` | Failed assertion or invariant              |
| `network`   | HTTP, connection, or timeout failure       |
| `data`      | Bad/unexpected data shape or parse failure |
| `auth`      | Authentication or permission failure       |
| `ui`        | Rendering or display error                 |
| `logic`     | Incorrect behaviour without an exception   |

**Severity**:

| Severity   | Criteria                                           |
| ---------- | -------------------------------------------------- |
| `critical` | High event count OR >10 users affected, production |
| `high`     | Moderate frequency OR 2–10 users affected          |
| `low`      | Rare, single user, or non-production environment   |

### 3. Prepare the bug report fields

Map extracted data to the `bug-report` format:

- **Title**: `[Bug] {ErrorType}: {short message}` (max ~80 chars)
- **Description**: `{error_type}: {error_message}` followed by `**Culprit**: {culprit}`
- **Stack Trace**: top 5 application-layer frames as a code block
- **Affected Files / Components**: bulleted list of paths from triage
- **Triage**: category and severity with one-sentence rationale for each
- **Source**:
  - **Issue ID**: {issue_id}
  - **URL**: {issue_url}
  - **First seen**: {first_seen} | **Last seen**: {last_seen}
  - **Events**: {event_count} | **Users affected**: {user_count}
- **Fix Criteria**:
  - [ ] Root cause identified
  - [ ] Fix implemented and tested
  - [ ] Verified issue no longer appears in Sentry

Omit any field where data is unavailable.

### 4. Create the bug report

Invoke the `bug-report` skill with the title and pre-populated fields above. Let `bug-report` handle database resolution, schema mapping, and page creation.

### 5. Report

```
✅ Created: [Bug] {ErrorType}: {short_message}
   Sentry:   {issue_url}
   Severity: {severity} | Category: {category}
   Files:    {top affected files, comma-separated, or "n/a"}
   Notion:   {page_url}
```
