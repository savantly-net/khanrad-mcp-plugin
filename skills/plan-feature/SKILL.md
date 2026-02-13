---
name: plan-feature
description: This skill should be used when the user asks to "plan a feature", "decompose a feature", "break down a feature into stories", "add a feature to the board", or wants to go from a feature description to a set of Khanrad issues for an existing application. Use this for brownfield development — when there is already a codebase and the user wants to add new functionality.
version: 1.0.0
---

# Khanrad Plan Feature Skill

This skill takes a feature description for an existing application, explores the codebase to understand architecture and conventions, performs impact analysis, generates stories with parallel subagents, and populates a Khanrad board with tagged, prioritized, dependency-ordered issues.

## When This Skill Applies

- User wants to plan a new feature for an existing application
- User asks to break down or decompose a feature into stories
- User wants to go from a feature idea to Khanrad issues
- User mentions "plan feature", "feature decomposition", "brownfield", or "add a feature"
- User has a codebase and wants implementation stories, not a greenfield brainstorm

## Key Concepts

- **Impact area** — a concrete slice of the existing codebase that needs changes for the feature. Unlike greenfield domains (which are invented), impact areas are discovered by reading the code. Examples: `impact:api`, `impact:ui`, `impact:data`, `impact:auth`.
- **Codebase context brief** — a summary produced by Explore subagents that describes the project's architecture, conventions, relevant existing code, and current board state. This brief is passed to story-generation subagents so they produce code-aware stories.
- **Tag taxonomy** — issues are tagged with structured labels for filtering:
  - `feature:<name>` — the feature being developed (e.g., `feature:dark-mode`)
  - `impact:<area>` — area of the codebase affected (e.g., `impact:api`, `impact:ui`)
  - `type:story` / `type:refactor` / `type:migration` / `type:test` / `type:spike` — work type
  - `risk:high` / `risk:medium` / `risk:low` — based on what code is being touched
  - `cross-cutting` — items spanning multiple impact areas
- **Parallel subagents** — Explore subagents analyze the codebase and existing board in parallel, then general-purpose subagents generate stories per impact area concurrently.
- **Risk tagging** — brownfield's biggest danger is unexpected blast radius. Each story is tagged with risk level based on whether it touches isolated new code (low), shared modules (medium), or core abstractions and data schemas (high).

## Workflow Overview

1. **Intake** — collect the feature description, resolve the existing Khanrad project and board, load existing board issues, and ask refining questions.
2. **Explore** — spawn parallel Explore subagents to analyze architecture, find feature-adjacent code, identify conventions, and cross-reference existing Khanrad issues.
3. **Impact analysis** — identify areas of impact (new modules, modified modules, data changes, test surface, infrastructure, refactoring prerequisites) and confirm with the user.
4. **Generate** — spawn parallel subagents (one per impact area) to produce stories with file-level references, risk assessments, and dependency notes.
5. **Review** — present stories grouped by impact area with dependency graph, risk callouts, and duplicate detection against existing board issues. Confirm implementation sequence.
6. **Populate** — create issues on the existing board (or a feature-specific board) with tags, priorities, risk levels, and state organization.

## Output

Issues on a Khanrad board tagged by feature, impact area, work type, and risk level. Stories reference actual files and conventions from the codebase. Dependencies between stories reflect a concrete implementation sequence.
