---
name: claude-md
description: This skill should be used when the user mentions "CLAUDE.md" together with khanrad or task management, asks to "add task management instructions", "make Claude use khanrad", "configure khanrad in CLAUDE.md", or wants to generate instructions that teach Claude when to use Khanrad tools.
version: 1.0.0
---

# Khanrad CLAUDE.md Skill

This skill helps users generate and manage CLAUDE.md instructions that teach Claude when and how to use Khanrad kanban tools for task management.

## When This Skill Applies

- User wants to add Khanrad task management instructions to CLAUDE.md
- User asks how to make Claude use Khanrad for task tracking
- User mentions configuring kanban-driven development for a project
- User wants to customize which Khanrad tools Claude uses automatically

## Quick Tool Reference

| Tool | When to Use | Key Params |
|------|-------------|------------|
| `list-projects` | Discover available projects | _(none)_ |
| `create-project` | Set up a new project | `name`, `slug`, `description` |
| `list-boards` | Find boards in a project | `projectId` |
| `create-board` | Create a board with default states | `projectId`, `name`, `description` |
| `list-issues` | Browse issues, filter by state/assignee/priority | `boardId`, `stateId`, `assigneeId`, `priority` |
| `get-issue` | Read issue details and comments | `issueId` |
| `create-issue` | File a new issue | `boardId`, `title`, `description`, `priority`, `labels` |
| `update-issue` | Edit issue fields | `issueId`, `title`, `description`, `priority`, `labels` |
| `move-issue` | Change issue state (e.g., Todo → In Progress) | `issueId`, `stateId` |
| `claim-issue` | Assign issue to yourself | `issueId` |
| `unclaim-issue` | Remove yourself from an issue | `issueId` |
| `add-comment` | Log progress or notes on an issue | `issueId`, `content` |

## Resource Reference

| Resource | URI | When to Use |
|----------|-----|-------------|
| Board Summary | `kahnrad://board/{boardId}/summary` | Get overview of issue counts per state |
| Agent Tasks | `kahnrad://agent/tasks` | Check all issues assigned to you |

## Task Management Patterns

Five composable patterns, selected based on project complexity:

1. **Session Start** — always included. Check assigned tasks at session start to understand current work.
2. **Task Lifecycle** — for active development. Claim issues, update progress, move through states.
3. **Progress Reporting** — for tracked projects. Add comments to issues as work progresses.
4. **Board Awareness** — for complex projects. Check board summary before starting work.
5. **Multi-agent Coordination** — for team workflows. Claim before working, unclaim if blocked, check for conflicts.

## Setup Command

Users can run `/khanrad:claude-md` for a guided workflow that analyzes the project, detects existing boards, selects appropriate patterns, and generates the CLAUDE.md section.

## Templates

See `operations/templates.md` for the full composable template blocks, composition rules, and complete tier examples.
