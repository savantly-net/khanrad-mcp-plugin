# CLAUDE.md

## Project

Khanrad MCP Plugin — a Claude Code plugin that connects to a deployed Khanrad kanban instance via HTTP transport. This is a configuration-only plugin (no runtime code) containing MCP server config, commands, and skills.

## Best Practices

Follow these principles when modifying or extending this plugin:

### 12-Factor App Principles

- **Config in the environment** — all instance-specific values (`KHANRAD_URL`, `KHANRAD_API_KEY`) are environment variables, never hardcoded. The `.mcp.json` references `${VAR}` placeholders resolved at runtime.
- **Dev/prod parity** — the plugin connects to any Khanrad instance via URL. No separate "dev mode" or mock endpoints. Test against a real instance.
- **Backing services as attached resources** — the Khanrad API is treated as an attached resource identified only by URL and credentials. Swapping instances requires only changing env vars.
- **Explicitly declare dependencies** — all MCP server dependencies are declared in `.mcp.json`. No implicit assumptions about what's available.
- **Keep config strictly separated from code** — commands and skills contain logic and templates; connection details live entirely in env vars and settings files.

### SOLID Principles

- **Single Responsibility** — each file has one job: `.mcp.json` handles connection, `commands/setup.md` handles configuration, `commands/claude-md.md` handles generation, skills define trigger conditions, operations hold reusable templates.
- **Open/Closed** — add new commands by creating new files in `commands/`, new skills in `skills/`. Existing files don't need modification to extend functionality.
- **Interface Segregation** — skills are split by concern (setup vs claude-md). Each command declares only the tools it needs in its `allowed-tools` frontmatter.
- **Dependency Inversion** — the plugin depends on the MCP protocol abstraction, not on Khanrad internals. Any server implementing the same MCP tools would work.

### General

- Keep command instructions declarative — describe *what* to do, not *how* Claude should think about it.
- Templates in `skills/claude-md/operations/templates.md` use behavioral triggers ("When X, do Y"), not documentation. Don't add tool parameter docs there.
- Plugin files are plain markdown and JSON — no build step, no transpilation, no dependencies.
- Test changes with `claude --plugin-dir .` before committing.
- Never hardcode URLs, API keys, or instance-specific values in any file.
