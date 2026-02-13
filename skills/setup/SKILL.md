---
name: setup
description: This skill should be used when the user asks to "set up khanrad", "configure khanrad", "connect to khanrad", mentions "KHANRAD_API_KEY", "KHANRAD_URL", or discusses khanrad configuration, authentication setup, or API key configuration.
version: 1.0.0
---

# Khanrad Setup Skill

This skill helps users configure their connection to the Khanrad Kanban API.

## When This Skill Applies

- User wants to set up or configure the Khanrad plugin
- User needs help with API key or URL configuration
- User is troubleshooting connection issues
- User wants to configure different Khanrad instances per project
- User wants to create or update `.khanrad.json`

## Key Concepts

- **`KHANRAD_URL`** — the URL of the deployed Khanrad instance (e.g., `https://khanrad.dev`). Does not include `/api/mcp`.
- **`KHANRAD_API_KEY`** — an API key generated from the Khanrad Settings UI. Prefixed with `knrd_`.
- **`.khanrad.json`** — workspace config file mapping this project to a Khanrad project and board by slug. Safe to commit. Example:
  ```json
  {
    "project": "my-project",
    "defaultBoard": "development"
  }
  ```

The URL is typically set globally (one Khanrad instance), while the API key can be global or per-project (different orgs may use different keys).

## Configuration

### Recommended Setup

Set the URL **globally** in `~/.claude/settings.json`:

```json
{
  "env": {
    "KHANRAD_URL": "https://khanrad.dev"
  }
}
```

Set the API key **per-project** in `.claude/settings.json` in the project root:

```json
{
  "env": {
    "KHANRAD_API_KEY": "knrd_your_api_key_here"
  }
}
```

### Global API Key

If using the same key everywhere, set both globally:

```json
{
  "env": {
    "KHANRAD_URL": "https://khanrad.dev",
    "KHANRAD_API_KEY": "knrd_your_api_key_here"
  }
}
```

### Workspace Context (`.khanrad.json`)

A `.khanrad.json` file in the project root maps the workspace to a Khanrad project and board by slug:

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `project` | string | Yes | Slug of the Khanrad project |
| `defaultBoard` | string | No | Slug of the default board |

This file is safe to commit — it contains only slugs, no secrets. At session start, Claude reads this file and resolves slugs to IDs via `list-projects` and `list-boards`. When no `.khanrad.json` exists, the plugin falls back to slug-based auto-discovery by matching the project name against Khanrad project slugs. The `/khanrad:setup` command offers to create this file after environment variable configuration.

### Git Safety

Make sure `.claude/settings.json` is in `.gitignore` to avoid committing secrets. Project-level settings override global settings.

## Setup Command

Users can run `/khanrad:setup` for a guided walkthrough that handles URL input, API key configuration, scope selection, `.gitignore` checks, and `.khanrad.json` creation.

## Verification

After configuration, restart Claude Code and verify the Khanrad MCP tools are available. If the connection fails, check:

1. The API key is valid and not expired
2. Network connectivity to the Khanrad instance
3. The URL does not include `/api/mcp` (the plugin appends this)
4. Project-level settings aren't being overridden unexpectedly
