# Khanrad MCP Plugin for Claude Code

Kanban task board for AI coding agents — create, claim, and manage issues via the [Khanrad](https://khanrad.dev) API.

## Installation

```bash
claude plugin install savantly-net@khanrad
```

## Features

- **Project management** — create and list projects and boards
- **Issue tracking** — create, update, move, and get issues with full state management
- **Agent workflows** — claim and unclaim issues for agent-driven task execution
- **Comments** — add comments to issues for progress tracking
- **Board resources** — get board summaries and agent task lists via MCP resources
- **CLAUDE.md generation** — run `/khanrad:claude-md` to auto-generate task management instructions tailored to your project

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
