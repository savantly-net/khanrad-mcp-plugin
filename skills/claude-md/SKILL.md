---
name: claude-md
description: This skill should be used when the user mentions "CLAUDE.md" together with khanrad or task management, asks to "add task management instructions", "make Claude use khanrad", "configure khanrad in CLAUDE.md", or wants to generate instructions that teach Claude when to use Khanrad tools.
version: 1.1.0
---

# Khanrad CLAUDE.md Skill

This skill helps users generate and manage CLAUDE.md instructions that teach Claude how to connect to the right Khanrad project and board for this workspace.

## When This Skill Applies

- User wants to add Khanrad task management instructions to CLAUDE.md
- User asks how to make Claude use Khanrad for task tracking
- User mentions configuring kanban-driven development for a project
- User has a `.khanrad.json` file and wants CLAUDE.md to reference it

## What the MCP Server Already Provides

The Khanrad MCP server ships its own instructions that cover the standard workflow. Claude already knows to:

- Check `khanrad://agent/tasks` at session start
- Claim issues before starting work
- Move issues through states as work progresses
- Add comments for progress, blockers, and decisions
- Unclaim issues when blocked so others can pick them up
- Use `khanrad://board/{boardId}/summary` for board overview

**CLAUDE.md should NOT repeat any of this.** It only needs to provide what the MCP server cannot: project-specific context.

## What CLAUDE.md Adds

1. **Workspace context** — how to resolve `.khanrad.json` slugs to IDs so Claude knows which board to use
2. **Slug discovery** — how to auto-discover the project when no `.khanrad.json` exists
3. **Issue creation guidance** — when to create or update issues (not part of the MCP server's standard workflow)

## Project Discovery

The plugin resolves project/board context in priority order:

1. **`.khanrad.json`** — read committed config, resolve slugs to IDs via `list-projects` and `list-boards`
2. **Slug discovery** — normalize the project name to a slug, match against Khanrad project slugs via `list-projects`, use the first board. When this succeeds, offer to create `.khanrad.json` to persist the mapping.
3. **No match** — proceed without board context; user can create a project manually

This makes the CLAUDE.md portable — it works for any contributor without knowing specific IDs.

## Setup Command

Users can run `/khanrad:claude-md` for a guided workflow that analyzes the project, detects existing boards, and generates the CLAUDE.md section.

## Templates

See `operations/templates.md` for template blocks, composition rules, and examples.
