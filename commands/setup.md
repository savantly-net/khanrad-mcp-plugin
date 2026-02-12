---
description: Configure Khanrad connection (set instance URL and API key for this project or globally)
allowed-tools: [Bash, Read, Write, Edit, AskUserQuestion]
---

# Khanrad Setup

Help the user configure their connection to the Khanrad Kanban API.

## Steps

1. Check if `KHANRAD_URL` and `KHANRAD_API_KEY` are already set:
   - Run `echo $KHANRAD_URL` and `echo $KHANRAD_API_KEY` to check
   - If set, show the URL and key prefix (first 10 characters only) and confirm the configuration
   - If both are already configured, ask if they want to reconfigure or exit

2. Ask for the Khanrad instance URL:
   - e.g., `https://khanrad.example.com`
   - Strip any trailing slash
   - Validate it looks like a URL (starts with `http://` or `https://`)

3. Ask for their API key (prefix: `knrd_`):
   - Generated from Khanrad Settings > API Keys
   - Validate it starts with `knrd_`

4. Ask the user what scope they want for each setting:

   **For the URL:**
   - Recommend setting **globally** in `~/.claude/settings.json` (since users typically have one Khanrad instance)
   - Create or update the settings file to include:
     ```json
     {
       "env": {
         "KHANRAD_URL": "https://your-khanrad-instance.example.com"
       }
     }
     ```
   - Preserve any existing settings in the file

   **For the API key:**
   - **Per-project** (recommended) — set in `.claude/settings.json` in the project root. Different projects may use different API keys for different orgs.
   - **Global** — set in `~/.claude/settings.json` if the same key is used everywhere
   - Create or update the chosen settings file to include:
     ```json
     {
       "env": {
         "KHANRAD_API_KEY": "knrd_your_api_key_here"
       }
     }
     ```
   - Preserve any existing settings in the file

5. If any setting was saved per-project:
   - Check if `.gitignore` exists and whether it already includes `.claude/settings.json`
   - If not, offer to add it to prevent committing secrets

6. Verify the configuration works by restarting Claude Code or checking that the Khanrad MCP tools are available.

7. Report success and explain that Khanrad tools are now available in this session.

## Important

- Never log or display the full API key — only show the prefix (first 10 characters)
- Warn the user not to commit API keys to version control
- If the user already has a global key and wants to override it for a specific project, project-level settings take precedence
- The URL should not include `/api/mcp` — that path is appended automatically by the plugin
