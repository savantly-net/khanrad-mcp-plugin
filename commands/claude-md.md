---
description: Generate CLAUDE.md task management instructions for the current project — teaches Claude when and how to use Khanrad tools
allowed-tools: [Read, Write, Edit, Glob, Grep, AskUserQuestion, mcp__khanrad__list-projects, mcp__khanrad__list-boards, mcp__khanrad__list-issues]
---

# Generate Khanrad Task Management Instructions for CLAUDE.md

Analyze the current project and generate a `## Khanrad` section for CLAUDE.md. The Khanrad MCP server already provides standard workflow instructions (claim, move, comment, unclaim, check assigned tasks). **CLAUDE.md only needs project-specific context** that the MCP server cannot provide. The user may provide a focus area in `$ARGUMENTS`.

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

2. **Existing CLAUDE.md** — check if one exists at project root:
   - If it has a `## Khanrad` section already, note this for Phase 3 (update instead of append)
   - Read the full file to understand existing instructions and avoid conflicts

Report findings to the user: project name, whether `.khanrad.json` exists.

## Phase 2: Check Existing Khanrad Board

Call these tools to see if a Khanrad board already exists for this project:

1. **If `.khanrad.json` was found** — resolve slugs to IDs:
   - `list-projects` — find the project whose slug matches the `project` field
   - If found, `list-boards` — find the board whose slug matches `defaultBoard` (if provided), otherwise note all boards
   - If the slug doesn't match any project or board, warn the user and fall back to name matching

2. **Otherwise** — try slug-based auto-discovery:
   - Normalize the detected project name to slug format: lowercase, replace spaces/underscores with hyphens, strip non-alphanumeric characters (e.g., `My App` → `my-app`)
   - `list-projects` — find a project whose slug exactly matches the normalized name
   - If matched, `list-boards` — use the first board as the default
   - If no slug match, proceed without board context

If slug discovery succeeded, **offer to create `.khanrad.json`** to persist the mapping.

## Phase 3: Compose Snippet

Read the skill template file at `skills/claude-md/operations/templates.md` relative to the plugin directory. Use it to compose the `## Khanrad` section.

**Always include** the appropriate project context block:
- If `.khanrad.json` exists → Workspace Context block
- If no `.khanrad.json` but slug discovery succeeded → Slug Discovery block
- If neither → include a comment noting that no board was found

**Optionally include** the Issue Management block if:
- The project actively creates issues from Claude sessions
- The user requests it via `$ARGUMENTS` (e.g., "with issue creation")
- The project has existing issues that suggest active tracking

**Do NOT include** instructions that duplicate the MCP server's standard workflow (claim, move, comment, unclaim, check assigned tasks, board summary).

Present the exact content in a fenced code block and explain:
- Where it will be inserted (new CLAUDE.md, or appended/updated in existing one)
- What's included and why

Ask the user for confirmation before applying. Use AskUserQuestion with options:
- "Apply as shown"
- "Add issue management" / "Remove issue management" (toggle the optional block)
- "Edit manually" (write to CLAUDE.md and let user edit)

## Phase 4: Apply

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
