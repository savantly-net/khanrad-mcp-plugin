# Project Analysis Reference

How to analyze a project for CLAUDE.md generation.

## Workspace Config Detection

Before resolving the project name, check for a `.khanrad.json` file in the project root. If it exists, read it and extract:

- `project` (required) — slug of the Khanrad project
- `defaultBoard` (optional) — slug of the default board

These slugs are resolved to IDs at runtime via `list-projects` and `list-boards`, so the generated CLAUDE.md never needs hardcoded IDs when this file is present.

## Slug-Based Discovery Fallback

When no `.khanrad.json` exists, the plugin can auto-discover a matching Khanrad project by comparing the resolved project name against project slugs:

1. Resolve the project name using the Project Name Resolution table below
2. Normalize the name to slug format: lowercase, replace spaces/underscores with hyphens, strip non-alphanumeric characters (e.g., `My App` → `my-app`, `@org/my_project` → `my-project`)
3. Call `list-projects` and look for an exact slug match
4. If matched, use that project. Call `list-boards` and use the first board as the default.
5. If no match, proceed without board context

When slug discovery succeeds during `/khanrad:claude-md`, offer to create `.khanrad.json` to make the mapping permanent.

## Project Name Resolution

Resolve in priority order — stop at the first match:

| Source | Field | Example |
|--------|-------|---------|
| `.khanrad.json` | `project` | `"my-app"` |
| `package.json` | `name` | `"my-app"` |
| `Cargo.toml` | `[package] name` | `name = "my-app"` |
| `pyproject.toml` | `[project] name` | `name = "my-app"` |
| `go.mod` | module path | `module github.com/user/my-app` → `my-app` |
| `composer.json` | `name` | `"vendor/my-app"` → `my-app` |
| `*.sln` or `*.csproj` | filename | `MyApp.sln` → `MyApp` |
| Git remote | repo name | `origin → github.com/user/my-app` → `my-app` |
| Directory name | basename | `/home/user/projects/my-app` → `my-app` |

For scoped names (e.g., `@org/my-app`), use the unscoped portion (`my-app`) in instructions.

## Complexity Signals

### Simple (→ Minimal tier)

- Single `package.json` / `Cargo.toml` / `pyproject.toml` at root
- Fewer than 5 top-level source directories
- One primary language
- No docker-compose, k8s, or CI/CD pipelines
- No workspace or monorepo configuration
- Examples: CLI tools, single-page apps, libraries, scripts

### Medium (→ Standard tier)

- Multiple modules or packages but single repo
- 1-2 primary languages
- Has build configuration (webpack, vite, rollup, etc.)
- May have docker-compose for local dev
- Has CI configuration (GitHub Actions, GitLab CI, etc.)
- 5-15 top-level source directories
- Examples: web apps with API, mobile apps with backend, libraries with examples

### Complex (→ Full tier)

- Monorepo indicators: `workspaces` in package.json, `lerna.json`, `nx.json`, `turbo.json`, `pnpm-workspace.yaml`
- Multiple services: multiple `Dockerfile`s, docker-compose with 3+ services, k8s manifests
- 3+ primary languages
- Extensive CI/CD (multiple workflows, deployment pipelines)
- Infrastructure as code (Terraform, Pulumi, CloudFormation)
- 15+ top-level source directories
- Examples: microservice platforms, monorepos, full-stack enterprise apps

## Tech Stack Detection

Scan for these files to detect the stack (report to user for context):

| File | Indicates |
|------|-----------|
| `package.json` | Node.js / JavaScript / TypeScript |
| `tsconfig.json` | TypeScript |
| `Cargo.toml` | Rust |
| `pyproject.toml`, `setup.py`, `requirements.txt` | Python |
| `go.mod` | Go |
| `Gemfile` | Ruby |
| `composer.json` | PHP |
| `*.csproj`, `*.sln` | .NET / C# |
| `build.gradle`, `pom.xml` | Java / Kotlin |
| `Dockerfile` | Containerized |
| `docker-compose.yml` | Multi-service |
| `terraform/`, `*.tf` | Infrastructure as code |

## Handling Existing CLAUDE.md

### No CLAUDE.md exists

Create a new file with a header and the Khanrad section:

```markdown
# CLAUDE.md

## Khanrad
{generated content}
```

### CLAUDE.md exists, no Khanrad section

Append the Khanrad section at the end of the file, separated by a blank line.

### CLAUDE.md exists, has `## Khanrad` section

Replace the existing section content. The section spans from `## Khanrad` to the next `##` heading or end of file. Preserve everything else in the file.

### CLAUDE.md exists, has Khanrad content under a different heading

Ask the user whether to:
1. Replace the existing section with a standardized `## Khanrad` heading
2. Keep the existing heading and update content underneath
3. Add a separate `## Khanrad` section (not recommended — may cause conflicts)
