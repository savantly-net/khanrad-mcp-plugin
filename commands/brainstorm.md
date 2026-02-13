---
description: Brainstorm a greenfield project — decompose an application idea into domains, generate stories with parallel subagents, and populate a Khanrad board
allowed-tools: [AskUserQuestion, Task, Read, Glob, mcp__khanrad__list-projects, mcp__khanrad__create-project, mcp__khanrad__list-boards, mcp__khanrad__create-board, mcp__khanrad__create-issue, mcp__khanrad__update-issue, mcp__khanrad__move-issue]
---

# Khanrad Brainstorm

Guide the user from an application idea to a fully populated Khanrad board of tagged, prioritized issues.

---

## Phase 1: Intake & Refinement

### 1.1 — Resolve Khanrad Context

Check if `.khanrad.json` exists in the project root. If it does, read it to get the project slug. Resolve the slug to a project ID via `list-projects`. Note the project ID for later — the user may want to create the board under this project, or they may want a new project entirely.

If `.khanrad.json` does not exist, that is fine. You will ask about the project in Phase 5.

### 1.2 — Collect the Application Description

If the user has not already provided a description, ask:

> What application do you want to build? Describe it in as much detail as you can — what it does, who it's for, and why it matters.

Accept whatever level of detail the user provides. Even a single sentence is fine — the refining questions will fill in the gaps.

### 1.3 — Ask Refining Questions

Ask questions **iteratively, not all at once**. Group 2-3 related questions per round using AskUserQuestion. Adapt based on prior answers — skip questions the user already addressed in their description.

**Round 1 — Users & Scope:**
- "Who are the primary users? Are there distinct roles (e.g., admin, customer, vendor)?"
- "What are the 3-5 most important things a user should be able to do?"

**Round 2 — Technical & Integration:**
- "Any technology requirements or constraints (language, framework, platform, hosting)?"
- "Does this need to integrate with external systems (auth providers, payment processors, third-party APIs, existing databases)?"

**Round 3 — Scale & Constraints:**
- "What are the non-functional requirements? Think about: expected user scale, compliance/regulatory needs, offline support, real-time features, multi-tenancy."
- "Any specific security or data privacy requirements?"

**Round 4 — MVP & Vision:**
- "What would a minimum viable product look like — the smallest version that delivers value?"
- "What features are explicitly out of scope for now but planned for the future?"

After each round, synthesize what you've learned so far into a running summary. Show this summary to the user periodically so they can correct any misunderstandings.

When you have enough context to identify distinct feature domains, move to Phase 2. You will know you have enough when you can list at least 3-4 clear domains with distinct responsibilities.

---

## Phase 2: Domain Decomposition

### 2.1 — Identify Domains

Analyze the refined description and identify **bounded contexts** — cohesive feature areas with distinct responsibilities. Examples:

- `auth` — Authentication & Authorization
- `user-profiles` — User Management & Profiles
- `catalog` — Product/Content Catalog & Search
- `orders` — Order Management & Checkout
- `payments` — Payment Processing
- `notifications` — Email, SMS, Push Notifications
- `admin` — Admin Dashboard & Moderation
- `analytics` — Reporting & Analytics
- `infrastructure` — CI/CD, Monitoring, Deployment

Aim for 4-10 domains. Fewer than 4 likely means the domains are too broad. More than 10 likely means they are too granular — consider merging.

### 2.2 — Present Domains for Confirmation

Present the domains as a numbered list with a one-sentence description of each. Ask:

> Here are the domains I've identified. Should any be merged, split, renamed, added, or removed?

Iterate until the user confirms the domain list.

### 2.3 — Classify Domains by Phase

Ask the user to classify each domain:
- **MVP** — required for the first usable release
- **v2** — planned for the second release
- **Future** — aspirational, not yet committed

Use AskUserQuestion with multiSelect to let the user pick MVP domains efficiently:

> Which domains are essential for the MVP? (Select all that apply)

The remaining domains default to `v2`. Ask if any should be pushed to `future`.

### 2.4 — Record the Decomposition

At this point you should have:
- A refined application description
- A confirmed list of domains with phase classifications
- Enough context to brief subagents

Hold all of this in your working context for Phase 3.

---

## Phase 3: Parallel Story Generation

### 3.1 — Spawn Subagents

For each confirmed domain, spawn a `Task` subagent with `subagent_type: "general-purpose"`. Run all subagents **in parallel** by issuing all Task calls in a single message.

Each subagent receives this prompt (fill in the placeholders):

```
You are a senior product engineer decomposing a greenfield application into implementable issues.

## Application
{refined_application_description}

## All Domains
{numbered_list_of_all_domains_with_descriptions}

## Your Assigned Domain: {domain_name}
{domain_description}

## Phase: {mvp_or_v2_or_future}

## User Roles
{confirmed_user_roles}

## Technical Constraints
{confirmed_tech_constraints}

## Task

Generate issues for the **{domain_name}** domain. Produce:

1. **1-3 epics** — large feature areas within this domain
2. **3-8 stories per epic** — each story should be implementable in 1-3 days by a single developer

For EACH issue, output this exact format:

### [EPIC or STORY] Title here
- **Parent epic**: (for stories only — the epic title this belongs under; omit for epics)
- **Priority**: HIGH | MEDIUM | LOW
- **Description**: 2-4 sentences explaining what this issue accomplishes and why it matters.
- **Acceptance criteria**:
  - [ ] Criterion 1
  - [ ] Criterion 2
  - [ ] Criterion 3
- **Dependencies**: List any other domains this depends on (e.g., "Requires domain:auth for user sessions"). Write "None" if independent.

## Guidelines

- Stories should be concrete and actionable, not vague ("Implement password reset flow" not "Handle authentication stuff")
- Include infrastructure/setup stories if this domain needs them (database schema, API scaffolding, third-party SDK integration)
- Consider error handling, edge cases, and observability as separate stories where non-trivial
- Order stories roughly by implementation sequence within each epic
- If the domain's phase is MVP, focus on the minimum viable implementation. For v2/future, you can be more aspirational.
- Do NOT generate stories for other domains. Only generate for {domain_name}.
```

### 3.2 — Collect Results

Wait for all subagents to return. Parse each subagent's output into a structured list of issues per domain. If any subagent fails or returns malformed output, note it — you will handle it in Phase 4.

---

## Phase 4: Review & Refinement

### 4.1 — Present Stories by Domain

For each domain, present a summary:

> **Domain: {domain_name}** ({phase}) — {count} issues ({epic_count} epics, {story_count} stories)
>
> Epic 1: {epic_title}
>   - {story_1_title} (HIGH)
>   - {story_2_title} (MEDIUM)
>   - ...
>
> Epic 2: {epic_title}
>   - ...

After presenting each domain, ask:

> Any changes to the **{domain_name}** stories? You can add, remove, modify, or reprioritize.

### 4.2 — Identify Cross-Cutting Concerns

Look across all domains for duplicate or overlapping stories (e.g., "Set up CI/CD" appearing in multiple domains, or "Configure logging" repeated). Consolidate these into cross-cutting stories and show the user:

> I found these cross-cutting concerns that span multiple domains. I'll tag them `cross-cutting` instead of a single domain:
> - {cross_cutting_story_1}
> - {cross_cutting_story_2}

### 4.3 — Final MVP Review

Present a count summary:
> **Total: {n} issues** — {mvp_count} MVP, {v2_count} v2, {future_count} future
> **MVP issues by domain:** {domain}: {count}, {domain}: {count}, ...

Ask:
> Does the MVP scope look right? Any stories to promote or demote?

Iterate until the user confirms.

---

## Phase 5: Khanrad Population

### 5.1 — Resolve or Create the Project

If you resolved a project from `.khanrad.json` in Phase 1, ask the user if they want to use that project or create a new one.

If no project was resolved, call `list-projects` and present the options:
- Pick an existing project
- Create a new one (ask for name and slug)

If creating, call `create-project` with the user's chosen name and slug.

### 5.2 — Create the Board

Always create a **new board** for a brainstorm session. Ask the user for a board name, suggesting a default like `"{project_name} Backlog"` or `"{project_name} v1"`.

Call `create-board` with the project ID and chosen name. Note the board ID and the state IDs returned (the board comes with default states: Ice Box, Backlog, Todo, In Progress, Blocked, In Review, Done).

Identify these key state IDs from the response:
- **Ice Box** — for `phase:future` issues
- **Backlog** — for `phase:mvp` and `phase:v2` issues (the default state for new issues)

### 5.3 — Create Issues

For each approved issue, call `create-issue` with:
- **boardId**: the new board's ID
- **title**: the issue title
- **description**: the full description including acceptance criteria and dependencies, formatted in markdown
- **priority**: HIGH, MEDIUM, or LOW
- **labels**: array of tags — for example: `["domain:auth", "phase:mvp", "type:story"]`

Create epics first, then stories. For stories, include the parent epic title in the description under a `**Parent Epic:**` heading so the relationship is documented.

**Create issues in batches** — call `create-issue` for multiple issues in parallel where possible. Do not create them one at a time.

### 5.4 — Organize by State

After all issues are created:
- Move `phase:future` issues to the **Ice Box** state via `move-issue`
- Leave `phase:mvp` issues in **Backlog** (the default)
- Leave `phase:v2` issues in **Backlog**

### 5.5 — Summary

Present the final summary to the user:

> **Brainstorm complete!**
>
> **Project:** {project_name}
> **Board:** {board_name}
>
> | Phase | Issues |
> |-------|--------|
> | MVP   | {count} |
> | v2    | {count} |
> | Future | {count} |
> | **Total** | **{count}** |
>
> **Domains:** {domain_1} ({count}), {domain_2} ({count}), ...
>
> All issues are tagged and organized on your board. Filter by `domain:*` to focus on a specific area, or by `phase:mvp` to see your MVP backlog.

---

## Important

- **Never skip the refinement questions.** A vague description produces vague stories. Always go through at least 2 rounds of questions, even if the user provides a detailed initial description.
- **Always confirm domains with the user before generating stories.** The domain list is the foundation — getting it wrong wastes the entire generation phase.
- **Subagent prompts must include the full application context.** Each subagent runs independently with no access to your conversation history. Include everything they need in the prompt.
- **Tag format is `namespace:value`** — always lowercase, always hyphenated (e.g., `domain:user-profiles`, not `domain:User Profiles` or `domain:userProfiles`).
- **Stories should be actionable and specific.** If a subagent returns vague stories like "Handle authentication", refine them yourself before presenting to the user.
- **Respect the user's time.** If the user wants to skip a refinement round or accept stories without changes, let them. The thorough process is there to help, not to force.
