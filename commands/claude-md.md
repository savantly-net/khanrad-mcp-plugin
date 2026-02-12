---
description: Generate CLAUDE.md task management instructions for the current project — teaches Claude when and how to use Khanrad tools
allowed-tools: [Read, Write, Edit, Glob, Grep, AskUserQuestion, mcp__khanrad__list-projects, mcp__khanrad__list-boards, mcp__khanrad__list-issues]
---

# Generate Khanrad Task Management Instructions for CLAUDE.md

Analyze the current project and generate a `## Khanrad` section for CLAUDE.md that teaches Claude when and how to use Khanrad tools for task management. The user may provide a focus area in `$ARGUMENTS` (e.g., "minimal", "full", "agent-workflow").

## Phase 1: Analyze Project

Detect the project context:

0. **Workspace config** — check for `.khanrad.json` in the project root:
   - If it exists, read it and extract the `project` slug and optional `defaultBoard` slug
   - These slugs will be used in Phase 2 to resolve IDs instead of name matching

1. **Project name** — resolve in priority order:
   - `package.json` → `name`
   - `Cargo.toml` → `[package] name`
   - `pyproject.toml` → `[project] name`
   - `go.mod` → module path
   - Git remote → repo name
   - Fall back to current directory name

2. **Stack and complexity** — scan for signals:
   - Count top-level directories, source files, config files
   - Check for monorepo indicators (workspaces, lerna, nx, turborepo)
   - Detect frameworks and languages from manifests and file extensions
   - **Simple**: single-purpose project, one language
   - **Medium**: multiple modules, 1-2 languages
   - **Complex**: monorepo, microservices, 3+ languages

3. **Existing CLAUDE.md** — check if one exists at project root:
   - If it has a `## Khanrad` section already, note this for Phase 4 (update instead of append)
   - Read the full file to understand existing instructions and avoid conflicts

Report findings to the user: project name, detected stack, complexity tier.

## Phase 2: Check Existing Khanrad Board

Call these tools to see if a Khanrad board already exists for this project:

1. **If `.khanrad.json` was found** — resolve slugs to IDs:
   - `list-projects` — find the project whose slug matches the `project` field
   - If found, `list-boards` — find the board whose slug matches `defaultBoard` (if provided), otherwise note all boards
   - If the slug doesn't match any project or board, warn the user and fall back to name matching

2. **Otherwise** — fall back to name matching:
   - `list-projects` — check for an existing project matching the detected name
   - If found, `list-boards` — check for an active board

If a board exists, note the project ID and board ID for inclusion in the generated instructions. If no board exists, the generated instructions will include setup steps.

## Phase 3: Select Patterns

Based on complexity tier and any user-provided `$ARGUMENTS` focus:

**Minimal** (simple projects):
- Session Start only (check assigned tasks)

**Standard** (medium projects):
- Session Start
- Task Lifecycle (claim, update, move)
- Progress Reporting

**Full** (complex projects):
- Session Start
- Task Lifecycle
- Progress Reporting
- Board Awareness
- Multi-agent Coordination

If `$ARGUMENTS` specifies a focus area, adjust selection:
- "minimal" / "simple" → force Minimal tier
- "full" / "everything" → force Full tier
- "agent-workflow" → include Task Lifecycle and Multi-agent Coordination

Tell the user which patterns were selected and why.

## Phase 4: Propose Snippet

Read the skill template file at `skills/claude-md/operations/templates.md` relative to the plugin directory. Use it to compose the appropriate snippet for the selected patterns.

Compose the `## Khanrad` section using the selected pattern templates.

If `.khanrad.json` exists, use the Workspace Context template block instead of hardcoding project/board IDs. This makes the generated instructions portable across environments.

If no `.khanrad.json` exists but a Khanrad board was found in Phase 2, include the project and board IDs in the generated instructions so Claude can immediately start working with the board.

Present the exact content in a fenced code block and explain:
- Where it will be inserted (new CLAUDE.md, or appended/updated in existing one)
- Which patterns are included and what they do

Ask the user for confirmation before applying. Use AskUserQuestion with options:
- "Apply as shown"
- "Adjust patterns" (go back to Phase 3)
- "Edit manually" (write to CLAUDE.md and let user edit)

## Phase 5: Apply

Based on confirmation:

1. **No existing CLAUDE.md** — create one with a minimal header and the Khanrad section:
   ```
   # CLAUDE.md

   ## Khanrad
   {generated content}
   ```

2. **Existing CLAUDE.md without Khanrad section** — append the section at the end

3. **Existing CLAUDE.md with Khanrad section** — replace the existing `## Khanrad` section (from the heading to the next `##` heading or end of file)

After writing, report success and remind the user that Claude will now use Khanrad tools according to these instructions in future sessions.
