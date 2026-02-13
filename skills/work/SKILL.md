---
name: work
description: This skill should be used when the user asks to "work through issues", "start working", "execute the board", "implement the stories", "grind through the backlog", or wants the agent to autonomously pick up and complete Khanrad issues one by one until the board is done.
version: 1.0.0
---

# Khanrad Work Skill

This skill turns the agent into an autonomous executor that works through Khanrad issues sequentially — claiming each issue, implementing the changes, verifying the code, committing, and marking the issue done before moving to the next.

## When This Skill Applies

- User wants the agent to work through issues on a Khanrad board
- User asks to "start working", "execute the backlog", or "implement the stories"
- User wants autonomous or supervised issue execution
- User has a board full of issues (from `/khanrad:brainstorm` or `/khanrad:plan-feature`) and wants them implemented

## Key Concepts

- **Work loop** — the agent continuously selects the next workable issue, implements it, verifies it, commits, and moves on. The loop runs until all issues are done, all remaining issues are blocked, or the user stops it.
- **Autonomy level** — the user chooses how much oversight they want:
  - **Dry run** — read each issue, plan what would be done, surface questions. No code changes.
  - **Supervised** — implement each issue, then show changes and ask for approval before marking done.
  - **Autonomous** — implement continuously, pause only on blockers or failures.
- **Issue selection** — issues are picked in priority order, respecting dependencies. Todo-state issues are picked before Backlog. Issues that unblock others are preferred.
- **Blocking** — when the agent cannot complete an issue after multiple attempts, it moves the issue to Blocked, adds a comment explaining why, unclaims it, and continues to the next workable issue.

## Workflow Overview

1. **Configure** — resolve the board, ask about autonomy level, scope filter, git strategy, and verification commands.
2. **Loop** — select next issue → claim → implement → verify → commit → mark done → report progress → repeat.
3. **Complete** — report final summary when all issues are done or no more are workable.

## Relationship to Other Commands

```
/khanrad:brainstorm    → Board of issues (greenfield)
/khanrad:plan-feature  → Board of issues (brownfield)
                             ↓
/khanrad:work          → Issues implemented and committed
```
