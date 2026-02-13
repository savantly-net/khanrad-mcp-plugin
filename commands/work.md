---
description: Work through Khanrad issues autonomously — claim, implement, verify, commit, and mark done in a loop until the board is complete
allowed-tools: [AskUserQuestion, Read, Write, Edit, Glob, Grep, Bash, mcp__khanrad__list-projects, mcp__khanrad__list-boards, mcp__khanrad__list-issues, mcp__khanrad__get-issue, mcp__khanrad__claim-issue, mcp__khanrad__unclaim-issue, mcp__khanrad__move-issue, mcp__khanrad__update-issue, mcp__khanrad__add-comment]
---

# Khanrad Work

Work through issues on a Khanrad board one by one. Claim each issue, implement it, verify, commit, mark done, and move to the next.

---

## Phase 0: Configuration

Before any work begins, establish working parameters.

### 0.1 — Resolve Khanrad Context

Read `.khanrad.json` if it exists. Resolve the project slug to a project ID via `list-projects`, and the board slug to a board ID via `list-boards`. Note the board's state IDs — you will need these throughout the loop:

- **Ice Box** — issues not yet scheduled
- **Backlog** — issues ready to be worked on
- **Todo** — issues prioritized for immediate work
- **In Progress** — the issue currently being worked on
- **Blocked** — issues that could not be completed
- **In Review** — issues awaiting user review (used in supervised mode)
- **Done** — completed issues

If `.khanrad.json` does not exist and no board can be resolved, inform the user and suggest running `/khanrad:setup` first.

### 0.2 — Load Board State

Call `list-issues` on the board. Count issues by state and present a summary:

> **Board: {board_name}**
>
> | State | Count |
> |-------|-------|
> | Todo | {n} |
> | Backlog | {n} |
> | Ice Box | {n} |
> | In Progress | {n} |
> | Blocked | {n} |
> | Done | {n} |
> | **Total** | **{n}** |

Show the workable issues (Todo + Backlog, not blocked) sorted by priority.

### 0.3 — Ask Autonomy Level

Use AskUserQuestion:

> How autonomous should I be?

Options:
- **Dry run (Recommended for first pass)** — "I'll read each issue, plan what I'd do, and surface any questions or ambiguities. No code changes."
- **Supervised** — "I'll implement each issue, then show you the changes for approval before marking it done."
- **Autonomous** — "I'll work continuously, pausing only when blocked or when I have a question I can't resolve myself."

### 0.4 — Ask Scope Filter

Use AskUserQuestion:

> What should I work on?

Options:
- **All workable issues** — "Todo and Backlog issues, sorted by priority."
- **Filter by tag** — "Only issues matching a specific tag (e.g., `feature:dark-mode`, `domain:auth`, `phase:mvp`)."
- **Specific issues** — "I'll show you the list and you pick which ones."

If the user chooses tag filtering, ask for the tag. If they choose specific issues, present the workable issues as a numbered list and let them select.

### 0.5 — Ask Git Strategy

Use AskUserQuestion:

> How should I handle git commits?

Options:
- **Commit on current branch (Recommended)** — "One commit per issue on the current branch. Simple and linear."
- **Branch per issue** — "Create a branch for each issue, commit there, then merge back when done."

### 0.6 — Ask Verification Commands

Ask the user:

> What commands should I run to verify changes? I need at least a build/compile command. Test and lint commands are optional.

Use AskUserQuestion with free-text answers for:
- "Build/compile command" (e.g., `npm run build`, `cargo build`, `go build ./...`)
- "Test command" (e.g., `npm test`, `pytest`, `cargo test`) — optional
- "Lint/typecheck command" (e.g., `npm run lint`, `npm run typecheck`) — optional

If the user doesn't provide any, try to auto-detect from config files:
- `package.json` → look for `scripts.build`, `scripts.test`, `scripts.lint`
- `Cargo.toml` → `cargo build`, `cargo test`
- `pyproject.toml` → look for test/lint config
- `Makefile` → look for `build`, `test`, `lint` targets

Store these commands for use throughout the loop.

### 0.7 — Confirm and Start

Present a summary of the configuration:

> **Ready to work**
>
> - **Mode:** {autonomy_level}
> - **Scope:** {scope_description} ({n} issues)
> - **Git:** {strategy}
> - **Build:** `{build_command}`
> - **Test:** `{test_command}` (or "none")
> - **Lint:** `{lint_command}` (or "none")
>
> Start working?

Wait for confirmation before beginning the loop.

---

## The Work Loop

Repeat the following for each issue until a stopping condition is met.

### Step 1 — Select Next Issue

Refresh the board state by calling `list-issues`. Apply the scope filter. Select the next issue using this priority:

1. **State order**: Todo > Backlog (never pick from Ice Box unless explicitly in scope)
2. **Priority order**: URGENT > HIGH > MEDIUM > LOW
3. **Unblocking value**: prefer issues that block the most other issues (clears the path for subsequent work)
4. **Stable tiebreaker**: lowest issue ID first (respects creation order from planning commands)

If no workable issues remain, go to the Completion section.

### Step 2 — Claim and Start

- Call `claim-issue` to assign the issue to yourself
- Call `move-issue` to move the issue to **In Progress**
- Call `add-comment` with: "Starting work on this issue."
- Announce to the user: `[{current}/{total}] Starting: "{issue_title}" ({priority}, {risk_tag})`

### Step 3 — Understand the Issue

- Call `get-issue` to read the full issue details
- Parse the description for:
  - **Acceptance criteria** — the definition of done for this issue
  - **Referenced files** — specific files to create or modify
  - **Conventions to follow** — patterns or examples to match
  - **Dependencies** — other issues this depends on (verify they are Done)
  - **Parent epic** — context about the broader goal
- Parse the labels for context:
  - `type:*` — what kind of work (story, refactor, migration, test, spike, infra)
  - `risk:*` — blast radius (high, medium, low)
  - `feature:*` — which feature this belongs to
  - `impact:*` — which area of the codebase
  - `domain:*` — which domain (for greenfield issues)

### Step 4 — Read Referenced Code

Read every file referenced in the issue description. If the issue mentions patterns to follow (e.g., "Follow the pattern in `src/routes/users.ts`"), read those example files too.

If the issue references files that do not exist yet (for `type:story` with new code), read the parent directory to understand the file organization.

If the issue lacks specific file references, use Glob and Grep to find relevant files based on the issue's description and labels.

### Step 5 — Execute by Issue Type

Execute differently based on the issue's `type:*` label:

**`type:story`** — Implement new functionality
1. Read the acceptance criteria carefully
2. Write or modify the code, following the conventions noted in the issue
3. Only write tests if the story involves complex logic (conditional branching, data transformations, state machines, algorithms). Simple CRUD, config changes, and straightforward UI do not need new tests.
4. Run verification commands

**`type:refactor`** — Restructure existing code
1. Read the existing code thoroughly before changing anything
2. Make the structural changes described in the issue
3. Run the full test suite — refactoring must not change behavior
4. If tests fail, the refactoring introduced a bug. Fix it before proceeding.

**`type:migration`** — Database or schema changes
1. Identify the project's migration tooling from the codebase
2. Create the migration file following the project's naming and placement conventions
3. If possible, run the migration locally to verify it works
4. Include both up and down migrations where the tooling supports it

**`type:test`** — Write tests
1. Read the code being tested
2. Write tests following the project's test framework and patterns
3. Run the tests to verify they pass
4. Ensure tests are meaningful — they should catch real bugs, not just assert truisms

**`type:spike`** — Research
1. Investigate the question described in the issue
2. Add a detailed comment to the issue with findings, recommendations, and any code examples
3. Do NOT write production code for a spike
4. Move directly to Done after commenting

**`type:infra`** — Configuration and infrastructure
1. Make the config, environment, or deployment changes described
2. Verify the changes don't break the build
3. If adding environment variables, document them in the appropriate config files

If an issue has no `type:*` label, treat it as `type:story`.

### Step 6 — Verify

Run the verification commands configured in Phase 0:

1. **Build command** — must pass. If it fails, the implementation has a compile/syntax error. Fix it.
2. **Test command** (if configured) — must pass. If tests fail, read the failures and fix.
3. **Lint command** (if configured) — should pass. Fix lint errors.

If verification fails:
- Read the error output
- Attempt to fix the issue
- Re-run verification
- Repeat up to **3 attempts**

If still failing after 3 attempts, go to Step 8 (Block).

### Step 7 — Commit

Stage only the files changed for this issue. Do NOT use `git add -A` or `git add .` — be specific about which files to stage.

Commit message format:
```
{issue_title}

Khanrad: {issue_id}
```

If using branch-per-issue strategy:
- Before Step 5, create a branch: `git checkout -b issue/{issue-id}-{slugified-title}`
- After committing, merge back: `git checkout {base_branch} && git merge issue/{issue-id}-{slugified-title}`

### Step 8 — Handle Completion or Failure

**If verification passed (normal path):**

- Call `add-comment` with a summary of what was done:
  ```
  Completed. Changes:
  - {file_1}: {what changed}
  - {file_2}: {what changed}

  Commit: {commit_hash}
  ```

- **If supervised mode:**
  - Move the issue to **In Review**
  - Show the user a summary:
    ```
    [{current}/{total}] Completed: "{issue_title}"

    Files changed:
    - {file}: {summary}

    Commit: {hash}
    ```
  - Run `git diff HEAD~1` via Bash and present a summary of the diff
  - Ask: "Approve this issue as done, or request changes?"
  - If approved → move to **Done**
  - If changes requested → read the user's feedback, revise, re-verify, and ask again

- **If autonomous mode:**
  - Move directly to **Done**
  - Report: `[{current}/{total}] Completed: "{issue_title}" — {files_changed} files, +{insertions}/-{deletions}`

- **If dry-run mode:**
  - Do NOT write any code. Instead, after reading the issue and referenced files in Steps 3-4, present:
    ```
    [{current}/{total}] Dry run: "{issue_title}"

    Plan:
    - {what_would_be_created_or_modified}
    - {what_tests_would_be_written_if_any}

    Questions/concerns:
    - {any_ambiguities_in_the_issue}
    - {any_missing_information}
    - {any_risks_identified}
    ```
  - If there are questions, ask the user and record their answers as comments on the issue so they are available during the real execution run.
  - Move to the next issue without changing state.

**If verification failed after 3 attempts (blocked path):**

- Call `add-comment` explaining what was attempted and what's failing:
  ```
  Blocked after 3 implementation attempts.

  What was attempted:
  - {attempt_1_description}
  - {attempt_2_description}
  - {attempt_3_description}

  Current failure:
  {error_output}

  Likely cause: {analysis}
  ```
- Revert any uncommitted changes for this issue: `git checkout -- .`
- Call `move-issue` to move to **Blocked**
- Call `unclaim-issue` so a human or another agent can pick it up
- Report: `[{current}/{total}] Blocked: "{issue_title}" — {reason}. Moving to next issue.`
- Increment the consecutive failure counter

### Step 9 — Check Stopping Conditions

Before looping back to Step 1, check:

1. **Consecutive failure threshold** — if 3 issues in a row were blocked, pause and ask the user:
   > Three consecutive issues have been blocked. This might indicate a systemic problem (broken build, missing dependency, bad test environment). Want to investigate before continuing?
   - Reset the counter if the user says to continue.

2. **No more workable issues** — if the next `list-issues` call shows no unblocked issues in scope, go to Completion.

3. **All issues Done** — go to Completion.

Otherwise, loop back to Step 1.

---

## Completion

When the loop ends, present a final summary:

> **Work session complete**
>
> | Status | Count |
> |--------|-------|
> | Done | {n} |
> | Blocked | {n} |
> | Remaining | {n} |
> | **Total in scope** | **{n}** |
>
> **Commits:** {n} on branch `{branch_name}`
>
> {if_any_blocked}
> **Blocked issues (need attention):**
> - {issue_title}: {reason}
> - {issue_title}: {reason}
> {end_if}

If in dry-run mode:

> **Dry run complete**
>
> Reviewed {n} issues. Questions and plans have been added as comments on each issue.
>
> {if_any_questions}
> **Issues with open questions:**
> - {issue_title}: {question_summary}
> - {issue_title}: {question_summary}
> {end_if}
>
> Run `/khanrad:work` again and choose **Supervised** or **Autonomous** to start implementing.

---

## Important

- **Never force push, delete branches, or run destructive git commands.** Only constructive operations.
- **Never modify files outside the project root.** No system files, no global configs.
- **Commit after every completed issue.** If interrupted, work is preserved.
- **Comment on every issue** — start, completion, and blocking. Progress must be visible in Khanrad even if the agent session ends unexpectedly.
- **Unclaim on block.** When the agent cannot complete an issue, unclaim it so others can pick it up.
- **Revert on failure.** If an issue cannot be completed, revert uncommitted changes so the codebase stays clean for the next issue.
- **Never skip failing tests.** If tests fail, either fix them or block the issue. Do not mark an issue Done with broken tests.
- **Never write tests just for coverage.** Only write tests for complex logic — conditional branching, data transformations, state machines, algorithms. Simple CRUD, config, and straightforward UI do not need dedicated test stories.
- **For `risk:high` issues, be extra careful.** Read more context, make smaller changes, verify more thoroughly. Add more detail to the completion comment.
- **Stage files explicitly.** Never use `git add -A` or `git add .`. Only stage files that were intentionally changed for the current issue.
- **Respect the user's autonomy level.** In supervised mode, always wait for approval. In autonomous mode, only pause on blockers. In dry-run mode, never write code.
- **Ask when genuinely uncertain.** If an issue's description is ambiguous and you cannot make a reasonable default choice, ask the user rather than guessing. But do not ask about every small decision — use your judgment for reasonable defaults and note them in the completion comment.
