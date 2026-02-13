# Khanrad MCP Plugin for Claude Code

Kanban task board for AI coding agents — create, claim, and manage issues via the [Khanrad](https://khanrad.dev) API.

## Installation

First, add the Khanrad marketplace:

```bash
claude plugin marketplace add savantly-net/khanrad-mcp-plugin
```

Then install the plugin:

```bash
claude plugin install khanrad@khanrad
```

Restart Claude Code to activate the plugin.

## Features

- **Project management** — create and list projects and boards
- **Issue tracking** — create, update, move, and get issues with full state management
- **Agent workflows** — claim and unclaim issues for agent-driven task execution
- **Comments** — add comments to issues for progress tracking
- **Board resources** — get board summaries and agent task lists via MCP resources
- **Workspace context** — commit a `.khanrad.json` file to map your repo to a Khanrad project and board by slug
- **Zero-config discovery** — automatically matches your project name against Khanrad project slugs when no config file exists
- **Greenfield brainstorm** — `/khanrad:brainstorm` decomposes an application idea into domains, generates stories with parallel subagents, and populates a Khanrad board with tagged, prioritized issues

## Configuration

Run `/khanrad:setup` inside Claude Code for guided configuration, or configure manually:

### Manual Setup

Set your Khanrad instance URL **globally** in `~/.claude/settings.json`:

```json
{
  "env": {
    "KHANRAD_URL": "https://khanrad.dev"
  }
}
```

Set your API key **per-project** in `.claude/settings.json` in your project root:

```json
{
  "env": {
    "KHANRAD_API_KEY": "knrd_your_api_key_here"
  }
}
```

Generate an API key from your Khanrad instance at **Settings > API Keys**.

### Global API Key

If you use the same API key across all projects, set it globally in `~/.claude/settings.json`:

```json
{
  "env": {
    "KHANRAD_URL": "https://khanrad.dev",
    "KHANRAD_API_KEY": "knrd_your_api_key_here"
  }
}
```

### Git Safety

Add `.claude/settings.json` to your `.gitignore` to avoid committing secrets. Project-level settings override global settings.

## Workspace Context (`.khanrad.json`)

Add a `.khanrad.json` file to your project root to map it to a Khanrad project and board by slug. This file is safe to commit — it contains no secrets.

```json
{
  "project": "my-project",
  "defaultBoard": "development"
}
```

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `project` | string | Yes | Slug of the Khanrad project |
| `defaultBoard` | string | No | Slug of the default board |

When this file is present, Claude resolves slugs to IDs at session start via `list-projects` and `list-boards` — no hardcoded IDs needed in CLAUDE.md. Run `/khanrad:setup` to create this file interactively.

### Zero-Config Discovery

When no `.khanrad.json` exists, the plugin can auto-discover a matching project by normalizing your project name (from `package.json`, directory name, etc.) to a slug and matching it against Khanrad project slugs. If a match is found, it uses that project's first board automatically.

## Tools

| Tool | Description |
|------|-------------|
| `list-projects` | List all projects for the organization |
| `create-project` | Create a new project |
| `list-boards` | List all boards for a project |
| `create-board` | Create a new board with default states |
| `list-issues` | List issues, optionally filtered by state, assignee, or priority |
| `get-issue` | Get a single issue with comments and state |
| `create-issue` | Create a new issue on a board |
| `update-issue` | Update an issue's title, description, priority, or labels |
| `move-issue` | Move an issue to a different state |
| `claim-issue` | Claim an issue (assign to current agent) |
| `unclaim-issue` | Unclaim an issue (remove assignee) |
| `add-comment` | Add a comment to an issue |

## Resources

| Resource | URI | Description |
|----------|-----|-------------|
| Board Summary | `khanrad://board/{boardId}/summary` | Issue counts per state for a board |
| Agent Tasks | `khanrad://agent/tasks` | All issues assigned to the current agent |

## Development

Test the plugin locally:

```bash
claude --plugin-dir .
```

## License

MIT
