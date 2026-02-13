---
description: Plan a feature for an existing application — explore the codebase, perform impact analysis, generate stories with parallel subagents, and populate a Khanrad board
allowed-tools: [AskUserQuestion, Task, Read, Glob, Grep, mcp__khanrad__list-projects, mcp__khanrad__list-boards, mcp__khanrad__create-board, mcp__khanrad__list-issues, mcp__khanrad__get-issue, mcp__khanrad__create-issue, mcp__khanrad__update-issue, mcp__khanrad__move-issue]
---

# Khanrad Plan Feature

Guide the user from a feature idea to a set of implementation stories on a Khanrad board, informed by the actual codebase.

---

## Phase 1: Intake & Context Resolution

### 1.1 — Resolve Khanrad Context

Read `.khanrad.json` if it exists. Resolve the project slug to a project ID via `list-projects`, and the board slug to a board ID via `list-boards`. Note the board's state IDs (Ice Box, Backlog, Todo, In Progress, etc.).

Call `list-issues` on the existing board to get a snapshot of current issues — their titles, labels, states, and priorities. This is used later for duplicate detection and understanding in-flight work.

If `.khanrad.json` does not exist and no board can be resolved, inform the user and suggest running `/khanrad:setup` first. Do not proceed without a target board — brownfield planning requires board context.

### 1.2 — Collect the Feature Description

If the user has not already provided a description, ask:

> Describe the feature you want to build. What should it do, and why does it matter?

Accept whatever level of detail the user provides.

### 1.3 — Ask Refining Questions

Ask questions **iteratively, not all at once**. Group 2-3 related questions per round using AskUserQuestion. Skip questions the user already addressed.

**Round 1 — Users & Behavior:**
- "Which users or roles does this feature affect?"
- "Walk me through the primary user flow — what does the user do step by step?"

**Round 2 — Integration & Constraints:**
- "Does this feature extend or modify any existing features? Which ones?"
- "Are there backward compatibility concerns — existing APIs, data formats, or UI contracts that must be preserved?"

**Round 3 — Rollout & Risk:**
- "Should this be behind a feature flag for gradual rollout?"
- "Are there any areas of the codebase you consider high-risk or fragile that this might touch?"

**Round 4 — Definition of Done:**
- "What does 'done' look like? How would you demo this feature?"
- "Any specific acceptance criteria or edge cases you already know about?"

After each round, synthesize what you've learned and show the summary to the user for correction.

When you have enough context to understand the feature's scope and boundaries, move to Phase 2.

---

## Phase 2: Codebase Exploration

Spawn **three Explore subagents in parallel** to analyze the codebase. Each subagent receives the feature description and refined context.

### Subagent 1: Architecture Scout

```
You are analyzing an existing codebase to understand its architecture for planning
a new feature.

## Feature Being Planned
{feature_description_and_refined_context}

## Task

Explore the project at {project_root} and produce an architecture brief:

1. **Project structure** — map the top-level directory layout and identify layers
   (API/routes, services/business logic, data/models, UI/frontend, config, tests)
2. **Tech stack** — identify the language, framework, database, test framework,
   and build tools from config files (package.json, Cargo.toml, pyproject.toml, etc.)
3. **Entry points** — find the main entry point(s) and how requests/events flow
   through the system
4. **Patterns** — identify architectural patterns in use (MVC, hexagonal, CQRS,
   microservices, monolith, etc.) and how they're implemented
5. **Test structure** — where tests live, what framework is used, how tests are
   organized (unit vs integration vs e2e)

Output a structured brief with these 5 sections. Be specific — reference actual
file paths and directory names.
```

### Subagent 2: Feature-Adjacent Scout

```
You are analyzing an existing codebase to find code related to a planned feature.

## Feature Being Planned
{feature_description_and_refined_context}

## Existing Khanrad Board Issues
{summary_of_existing_issues_titles_labels_states}

## Task

Explore the project at {project_root} and find:

1. **Similar features** — existing functionality that is similar to or will interact
   with the planned feature. Show the key files and explain how they work.
2. **Relevant models/schemas** — data models, database schemas, or type definitions
   that the feature will need to read, extend, or create. Show the actual field
   definitions.
3. **API endpoints** — existing routes or endpoints in the neighborhood of this
   feature. Show the route definitions and their handlers.
4. **Shared components** — reusable UI components, utility functions, middleware,
   or services that the feature could leverage or would need to modify.
5. **Related board issues** — from the existing Khanrad issues, identify any that
   overlap with or are prerequisites for this feature. List their titles and current
   states.

Output a structured brief with these 5 sections. Reference actual file paths,
function names, and line numbers.
```

### Subagent 3: Convention Scout

```
You are analyzing an existing codebase to understand its conventions for planning
a new feature.

## Feature Being Planned
{feature_description_and_refined_context}

## Task

Explore the project at {project_root} and document:

1. **Naming conventions** — how files, functions, classes, variables, routes, and
   database columns are named. Show examples from the code.
2. **File organization** — where new files of each type should go. Trace an existing
   feature end-to-end (route → handler → service → model → test) and show the
   file path pattern.
3. **Error handling** — how errors are handled, propagated, and reported to users.
   Show the pattern with actual code examples.
4. **Logging & observability** — what logging library is used, what gets logged,
   structured vs unstructured.
5. **Code style** — formatting, linting config, import ordering, documentation
   style. Check for .eslintrc, .prettierrc, rustfmt.toml, ruff.toml, etc.

Output a structured brief with these 5 sections. Include actual code snippets as
examples of each convention. The goal is that someone reading this brief could
write new code that looks like it belongs in this codebase.
```

### 2.4 — Compile Codebase Context Brief

When all three subagents return, compile their outputs into a single **codebase context brief**. This brief will be included in every story-generation subagent's prompt in Phase 4.

The brief should contain:
- Architecture summary (layers, patterns, tech stack)
- Relevant existing code (files, models, endpoints, components)
- Conventions (naming, organization, error handling, style)
- Existing board issues that relate to or overlap with this feature

If any subagent returned incomplete or unclear results, note the gaps — you may need to do a quick targeted search yourself with Glob or Grep to fill them in.

---

## Phase 3: Impact Analysis

### 3.1 — Identify Areas of Impact

Using the feature description and codebase context brief, identify every area of the codebase that needs to change. Classify each as:

- **NEW** — code that does not exist yet (new endpoints, new components, new services, new models)
- **MODIFY** — existing code that needs changes (extending a model, adding routes, updating a shared component)
- **DATA** — database migrations, schema changes, seed data, data backfills
- **REFACTOR** — existing code that needs cleanup before the feature can be built cleanly (prerequisite refactoring)
- **TEST** — new tests or updates to existing tests
- **INFRA** — configuration changes, environment variables, feature flags, deployment changes

### 3.2 — Present Impact Map

Present the impact analysis as a structured map:

> **Feature: {feature_name}**
>
> | # | Type | Area | Description | Files |
> |---|------|------|-------------|-------|
> | 1 | REFACTOR | {area} | {what needs refactoring and why} | {file paths} |
> | 2 | DATA | {area} | {what migration is needed} | {file paths} |
> | 3 | NEW | {area} | {what new code is needed} | {where it should go} |
> | 4 | MODIFY | {area} | {what existing code changes} | {file paths} |
> | 5 | TEST | {area} | {what tests are needed} | {where tests should go} |
> | 6 | INFRA | {area} | {what config/deploy changes} | {file paths} |

### 3.3 — Confirm with the User

Ask:
> Here's my impact analysis. Should any areas be added, removed, or adjusted? Are there areas you want to handle differently (e.g., skip for now, handle separately)?

### 3.4 — Identify Sequencing Constraints

Ask:
> Based on the impact map, here's the implementation sequence I'd suggest:
> 1. {prerequisite_refactoring} (must land first)
> 2. {data_migrations} (schema must exist before code)
> 3. {core_logic} (services and models)
> 4. {api_layer} (endpoints that expose the logic)
> 5. {ui_layer} (frontend that calls the API)
> 6. {tests} (can partially parallel with implementation)
> 7. {infra} (feature flag setup, config changes)
>
> Does this order work? Any adjustments?

### 3.5 — Feature Flag Decision

If the user indicated interest in feature flags (or if the feature touches user-facing behavior), ask:

> Should this feature be behind a feature flag? If yes, I'll include stories for flag setup and teardown.

Use AskUserQuestion with options:
- "Yes — gradual rollout with a feature flag"
- "No — ship directly when complete"

If yes, add a `INFRA` impact area for feature flag setup and a cleanup story for flag removal after full rollout.

---

## Phase 4: Story Generation

### 4.1 — Spawn Subagents

For each confirmed impact area, spawn a `Task` subagent with `subagent_type: "general-purpose"`. Run all subagents **in parallel**.

Each subagent receives this prompt (fill in the placeholders):

```
You are a senior engineer writing implementation stories for a feature in an
existing codebase. Your stories must be specific to this codebase — reference
actual files, follow established conventions, and account for existing code.

## Feature
{feature_description_and_refined_context}

## Codebase Context
{compiled_codebase_context_brief_from_phase_2}

## All Impact Areas
{numbered_list_of_all_impact_areas_with_types_and_descriptions}

## Your Assigned Impact Area
Name: {area_name}
Type: {NEW / MODIFY / DATA / REFACTOR / TEST / INFRA}
Description: {area_description}
Relevant files: {file_paths_from_impact_analysis}
Sequencing position: {where_this_falls_in_the_implementation_order}

## Existing Board Issues
{titles_and_labels_of_existing_issues_that_relate_to_this_feature}

## Task

Generate stories for this impact area. Produce 2-8 stories depending on scope.

For EACH story, output this exact format:

### [STORY or REFACTOR or MIGRATION or TEST or SPIKE] Title here
- **Risk**: HIGH | MEDIUM | LOW
  - HIGH = changes core abstractions, shared data schemas, or widely-used interfaces
  - MEDIUM = modifies shared modules or integrates with existing features
  - LOW = new isolated code or changes to leaf-node files
- **Files**: List the specific files to create or modify (use actual paths from the
  codebase context)
- **Description**: 2-4 sentences explaining what this story accomplishes. Reference
  the relevant existing code and explain how the new code integrates with it.
- **Acceptance criteria**:
  - [ ] Criterion 1 (be specific — reference actual behaviors, endpoints, or components)
  - [ ] Criterion 2
  - [ ] Criterion 3
- **Dependencies**: List other impact areas or stories this depends on. Write "None"
  if independent.
- **Conventions to follow**: Note any specific patterns from the codebase that apply
  to this story (e.g., "Follow the pattern in src/routes/users.ts for the new endpoint",
  "Use the existing ErrorBoundary component for error states").

## Guidelines

- Stories should reference real file paths and follow the project's conventions
- For MODIFY stories, describe what changes in the existing code (not just "update X")
- For DATA stories, describe the schema change precisely (columns, types, constraints)
- For TEST stories, specify what test framework and patterns to use
- For REFACTOR stories, explain why the refactoring is needed for the feature
- Order stories by implementation sequence within this impact area
- Do NOT generate stories for other impact areas
```

### 4.2 — Collect Results

Wait for all subagents to return. Parse each subagent's output into a structured list of stories per impact area. Note any gaps or quality issues to address in Phase 5.

---

## Phase 5: Review & Refinement

### 5.1 — Present Stories by Impact Area

For each impact area, present a summary following the implementation sequence:

> **{area_type}: {area_name}** — {count} stories
>
> | # | Type | Title | Risk | Dependencies |
> |---|------|-------|------|-------------|
> | 1 | REFACTOR | {title} | HIGH | None |
> | 2 | STORY | {title} | MEDIUM | Depends on #1 |
> | 3 | STORY | {title} | LOW | None |

After presenting each area, ask:

> Any changes to the **{area_name}** stories? You can add, remove, modify, or reprioritize.

### 5.2 — Cross-Cutting and Duplicate Detection

Look across all impact areas for:
- **Duplicates with existing board issues** — flag any new stories that overlap with issues already on the board. Ask the user if the new story should replace, extend, or be skipped.
- **Cross-cutting stories** — stories that appeared in multiple impact areas. Consolidate and tag `cross-cutting`.
- **Missing test coverage** — if an impact area has implementation stories but no corresponding test stories, flag it.

### 5.3 — Dependency Graph

Present the full implementation sequence across all impact areas:

> **Implementation Sequence:**
> 1. {refactor_story_1} (prerequisite — no dependencies)
> 2. {migration_story_1} (prerequisite — no dependencies)
> 3. {core_story_1} (depends on #1, #2)
> 4. {core_story_2} (depends on #3)
> 5. {api_story_1} (depends on #3)
> 6. {ui_story_1} (depends on #5)
> 7. {test_story_1} (can parallel with #3-#6)
> ...

### 5.4 — Risk Summary

> **Risk distribution:**
> - HIGH: {count} stories (touching core code)
> - MEDIUM: {count} stories (modifying shared modules)
> - LOW: {count} stories (new isolated code)
>
> **High-risk stories to watch:**
> - {story_title} — {why it's high risk}
> - {story_title} — {why it's high risk}

### 5.5 — Final Confirmation

> **Total: {n} stories** — {by_type_breakdown}
>
> Ready to create these issues on the board?

Iterate until the user confirms.

---

## Phase 6: Khanrad Population

### 6.1 — Choose the Board

Ask the user using AskUserQuestion:

- "Add to the existing board `{board_name}` (Recommended)" — issues mix with other work, filtered by `feature:` tag
- "Create a new feature board" — isolated view for this feature

If creating a new board, ask for a name (suggest `"{feature_name}"`) and call `create-board`.

### 6.2 — Create Issues

For each approved story, call `create-issue` with:
- **boardId**: the target board's ID
- **title**: the story title
- **description**: the full description including acceptance criteria, relevant files, dependencies, and conventions to follow — all formatted in markdown. Include a `**Feature:**` heading with the feature name and an `**Impact Area:**` heading with the area name.
- **priority**: derived from the story's risk and importance:
  - HIGH risk prerequisites → URGENT or HIGH priority
  - MEDIUM risk core stories → HIGH or MEDIUM priority
  - LOW risk stories → MEDIUM or LOW priority
- **labels**: array of tags — for example: `["feature:dark-mode", "impact:api", "type:story", "risk:medium"]`

**Create issues in batches** — call `create-issue` for multiple independent issues in parallel.

### 6.3 — Organize by State

After all issues are created:
- Move **prerequisite** stories (refactoring, migrations) to **Todo** state — these should be started first
- Leave **core implementation** stories in **Backlog**
- Move **deferred** or low-priority stories to **Ice Box**

### 6.4 — Summary

Present the final summary:

> **Feature planning complete!**
>
> **Feature:** {feature_name}
> **Board:** {board_name}
>
> | Type | Count |
> |------|-------|
> | Story | {n} |
> | Refactor | {n} |
> | Migration | {n} |
> | Test | {n} |
> | Spike | {n} |
> | **Total** | **{n}** |
>
> | Risk | Count |
> |------|-------|
> | High | {n} |
> | Medium | {n} |
> | Low | {n} |
>
> **Implementation sequence:** {count} prerequisite stories in Todo, {count} core stories in Backlog, {count} deferred in Ice Box.
>
> Filter by `feature:{name}` to see all stories for this feature.

---

## Important

- **Always explore the codebase.** The entire value of brownfield planning over greenfield brainstorming is that stories reference real code. Never skip Phase 2.
- **Always check existing board issues.** Duplicate stories waste effort and create confusion. Flag overlaps explicitly.
- **Subagent prompts must include the full codebase context brief.** Each subagent runs independently with no access to your conversation history or the codebase. The brief is their only source of codebase knowledge.
- **Tag format is `namespace:value`** — always lowercase, always hyphenated (e.g., `feature:dark-mode`, `impact:user-profiles`).
- **Risk is about blast radius, not difficulty.** A hard but isolated story is LOW risk. An easy change to a shared model is HIGH risk.
- **Respect the user's time.** If the user wants to skip refinement rounds or accept stories without changes, let them. The thorough process helps, it does not obstruct.
- **If `.khanrad.json` is missing or the board cannot be resolved, stop early.** Suggest `/khanrad:setup` instead. Brownfield planning without board context is incomplete.
