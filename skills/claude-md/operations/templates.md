# CLAUDE.md Template Blocks

Composable blocks for generating the `## Khanrad` section in CLAUDE.md. Each block is a behavioral trigger: "When X, do Y." Claude already knows tool schemas from MCP — these instructions tell it *when* to act, not *how*.

## Template Blocks

### Workspace Context (prepend when `.khanrad.json` exists)

```
At session start, if `.khanrad.json` exists in the project root, read it to get the project
slug and default board slug. Resolve slugs to IDs via `list-projects` and `list-boards`.
Use these IDs for all subsequent Khanrad operations in this session.
```

### Slug Discovery (prepend when no `.khanrad.json` but auto-discovery is desired)

```
At session start, resolve the project name to a slug (lowercase, hyphenated). Call `list-projects`
and match against project slugs. If a match is found, call `list-boards` and use the first board
for all Khanrad operations in this session.
```

### Session Start (always included)

```
At session start, read the `khanrad://agent/tasks` resource to check for assigned issues before doing any work.
When starting a coding session, check if there are open issues on the board that match the current task context.
```

### Task Lifecycle (active development)

```
When beginning work on an issue, call `claim-issue` to assign it to yourself.
When finishing work on an issue, call `move-issue` to advance it to the next state (e.g., "In Progress" → "In Review" or "Done").
When creating a task that needs tracking, call `create-issue` with a clear title, description, and appropriate priority.
When an issue's scope or requirements change, call `update-issue` to keep the title, description, and labels current.
```

### Progress Reporting (tracked projects)

```
When completing a significant step on an issue, call `add-comment` with a brief summary of what was done.
When encountering a blocker or making a decision about an issue, call `add-comment` to document it.
```

### Board Awareness (complex projects)

```
Before starting work, read the `khanrad://board/{boardId}/summary` resource to understand the current state of the board.
When unsure what to work on next, call `list-issues` filtered by priority to find the highest-priority unassigned work.
```

### Multi-agent Coordination (team workflows)

```
Always `claim-issue` before starting work to prevent other agents from duplicating effort.
If blocked on an issue, call `unclaim-issue` so another agent can pick it up, and call `add-comment` explaining the blocker.
When creating issues for other agents, set appropriate priority and labels so they can be filtered and discovered.
```

## Composition Rules

1. **Session Start is always first.** Checking assigned tasks at session start is the single highest-value instruction.
2. **Keep total lines between 2-12.** More than that and instructions compete with each other for attention.
3. **Each line must be a behavioral trigger.** Format: "When {condition}, call `{tool}` with {key context}."
4. **Don't duplicate what MCP provides.** Tool schemas, parameter types, and defaults come from MCP — only specify *when* to use tools and which params matter.
5. **Prefer `.khanrad.json` over hardcoded IDs.** If `.khanrad.json` exists, use the Workspace Context block instead of embedding IDs. If no config file exists but slug discovery found a match, offer to create `.khanrad.json` or use hardcoded IDs as a fallback.
6. **Use Slug Discovery sparingly.** The Slug Discovery block adds a runtime `list-projects` call each session. Prefer `.khanrad.json` for committed projects; reserve Slug Discovery for projects that want zero-config onboarding.

## Anti-Patterns

- **Don't add tool documentation.** CLAUDE.md is not a reference manual — it's behavioral configuration.
- **Don't instruct on every tool.** Only include tools relevant to the selected patterns.
- **Don't add conditional logic.** Keep instructions simple and unconditional within their pattern.
- **Don't exceed 15 lines.** If the section is getting long, remove lower-priority patterns instead of cramming.
- **Don't repeat MCP defaults.** If a tool parameter has a sensible default, don't mention it.

## Complete Tier Examples

> **Note:** When `.khanrad.json` exists, prepend the Workspace Context block to any tier. When no config file exists but auto-discovery is desired, prepend the Slug Discovery block instead. Both replace hardcoded project/board IDs with runtime resolution.

### Minimal (simple projects)

```markdown
## Khanrad

At session start, read the `khanrad://agent/tasks` resource to check for assigned issues before doing any work.
When starting a coding session, check if there are open issues on the board that match the current task context.
```

### Standard (medium projects)

```markdown
## Khanrad

At session start, read the `khanrad://agent/tasks` resource to check for assigned issues before doing any work.
When starting a coding session, check if there are open issues on the board that match the current task context.
When beginning work on an issue, call `claim-issue` to assign it to yourself.
When finishing work on an issue, call `move-issue` to advance it to the next state (e.g., "In Progress" → "In Review" or "Done").
When creating a task that needs tracking, call `create-issue` with a clear title, description, and appropriate priority.
When completing a significant step on an issue, call `add-comment` with a brief summary of what was done.
When encountering a blocker or making a decision about an issue, call `add-comment` to document it.
```

### Standard with `.khanrad.json` (medium projects, workspace config)

```markdown
## Khanrad

At session start, if `.khanrad.json` exists in the project root, read it to get the project
slug and default board slug. Resolve slugs to IDs via `list-projects` and `list-boards`.
Use these IDs for all subsequent Khanrad operations in this session.
At session start, read the `khanrad://agent/tasks` resource to check for assigned issues before doing any work.
When starting a coding session, check if there are open issues on the board that match the current task context.
When beginning work on an issue, call `claim-issue` to assign it to yourself.
When finishing work on an issue, call `move-issue` to advance it to the next state (e.g., "In Progress" → "In Review" or "Done").
When creating a task that needs tracking, call `create-issue` with a clear title, description, and appropriate priority.
When completing a significant step on an issue, call `add-comment` with a brief summary of what was done.
When encountering a blocker or making a decision about an issue, call `add-comment` to document it.
```

### Full (complex projects)

```markdown
## Khanrad

At session start, read the `khanrad://agent/tasks` resource to check for assigned issues before doing any work.
Before starting work, read the `khanrad://board/{boardId}/summary` resource to understand the current state of the board.
When unsure what to work on next, call `list-issues` filtered by priority to find the highest-priority unassigned work.
Always `claim-issue` before starting work to prevent other agents from duplicating effort.
When finishing work on an issue, call `move-issue` to advance it to the next state (e.g., "In Progress" → "In Review" or "Done").
When creating a task that needs tracking, call `create-issue` with a clear title, description, and appropriate priority.
When an issue's scope or requirements change, call `update-issue` to keep the title, description, and labels current.
When completing a significant step on an issue, call `add-comment` with a brief summary of what was done.
When encountering a blocker or making a decision about an issue, call `add-comment` to document it.
If blocked on an issue, call `unclaim-issue` so another agent can pick it up, and call `add-comment` explaining the blocker.
When creating issues for other agents, set appropriate priority and labels so they can be filtered and discovered.
```
