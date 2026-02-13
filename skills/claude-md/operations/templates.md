# CLAUDE.md Template Blocks

Composable blocks for generating the `## Khanrad` section in CLAUDE.md.

The Khanrad MCP server already provides instructions that cover the standard workflow: checking assigned tasks at session start, claiming issues before work, moving issues through states, adding comments for progress and blockers, and unclaiming when blocked. **CLAUDE.md should only contain project-specific instructions that the MCP server cannot provide** — primarily workspace configuration and issue creation guidance.

## Template Blocks

### Workspace Context (when `.khanrad.json` exists)

```
At session start, if `.khanrad.json` exists in the project root, read it to get the project
slug and default board slug. Resolve slugs to IDs via `list-projects` and `list-boards`.
Use these IDs for all subsequent Khanrad operations in this session.
```

### Slug Discovery (when no `.khanrad.json` but auto-discovery is desired)

```
At session start, resolve the project name to a slug (lowercase, hyphenated). Call `list-projects`
and match against project slugs. If a match is found, call `list-boards` and use the first board
for all Khanrad operations in this session.
```

### Issue Management (when the project creates/updates issues from CLAUDE.md)

```
When creating a task that needs tracking, call `create-issue` with a clear title, description, and appropriate priority.
When an issue's scope or requirements change, call `update-issue` to keep the title, description, and labels current.
```

## Composition Rules

1. **Start with project context.** Every generated section needs either Workspace Context or Slug Discovery so Claude knows which board to operate on.
2. **Keep total lines between 2-6.** The MCP server handles the standard workflow — CLAUDE.md only needs to fill gaps.
3. **Each line must be a behavioral trigger.** Format: "When {condition}, call `{tool}` with {key context}."
4. **Don't duplicate MCP server instructions.** The server already instructs Claude to: check `khanrad://agent/tasks` at session start, claim before working, move issues through states, add comments for progress/blockers, unclaim when blocked, and use board summaries. Do NOT repeat any of these.
5. **Prefer `.khanrad.json` over hardcoded IDs.** If `.khanrad.json` exists, use the Workspace Context block. If slug discovery found a match, offer to create `.khanrad.json` to persist the mapping.

## Anti-Patterns

- **Don't repeat the MCP server workflow.** Claim, move, comment, unclaim — these are already covered.
- **Don't add tool documentation.** CLAUDE.md is behavioral configuration, not a reference manual.
- **Don't exceed 6 lines.** If the section is getting long, you're probably duplicating MCP instructions.

## Examples

### With `.khanrad.json` (most projects)

```markdown
## Khanrad

At session start, if `.khanrad.json` exists in the project root, read it to get the project
slug and default board slug. Resolve slugs to IDs via `list-projects` and `list-boards`.
Use these IDs for all subsequent Khanrad operations in this session.
```

### With `.khanrad.json` + issue creation

```markdown
## Khanrad

At session start, if `.khanrad.json` exists in the project root, read it to get the project
slug and default board slug. Resolve slugs to IDs via `list-projects` and `list-boards`.
Use these IDs for all subsequent Khanrad operations in this session.
When creating a task that needs tracking, call `create-issue` with a clear title, description, and appropriate priority.
When an issue's scope or requirements change, call `update-issue` to keep the title, description, and labels current.
```

### Zero-config (slug discovery, no `.khanrad.json`)

```markdown
## Khanrad

At session start, resolve the project name to a slug (lowercase, hyphenated). Call `list-projects`
and match against project slugs. If a match is found, call `list-boards` and use the first board
for all Khanrad operations in this session.
```
