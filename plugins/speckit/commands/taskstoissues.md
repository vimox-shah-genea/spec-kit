---
description: Convert existing tasks into actionable, dependency-ordered GitHub issues for the feature based on available design artifacts.
tools: ['github/github-mcp-server/issue_write']
scripts:
  sh: .specify/scripts/bash/check-prerequisites.sh --json --require-tasks --include-tasks
  ps: .specify/scripts/powershell/check-prerequisites.ps1 -Json -RequireTasks -IncludeTasks
---

## Pre-flight: Auto-init

Before doing anything else, check if `.specify/scripts/bash/` exists:

```bash
test -d .specify/scripts/bash && echo "EXISTS" || echo "MISSING"
```

If the result is `MISSING`, silently run the full init procedure:

**Step 1 — Locate the installed plugin directory:**
```bash
find ~/.claude/plugins/cache -name "plugin.json" -path "*speckit*" 2>/dev/null | head -1 | xargs dirname 2>/dev/null
```
Use the returned path as `<plugin-dir>`. If nothing is returned, tell the user: "speckit plugin not found in `~/.claude/plugins/cache`. Please reinstall with `claude plugin install speckit`." and stop.

**Step 2 — Create directories:**
```bash
mkdir -p .specify/scripts/bash .specify/templates .specify/memory
```

**Step 3 — Copy scripts and templates:**
```bash
cp <plugin-dir>/scripts/bash/* .specify/scripts/bash/
cp <plugin-dir>/templates/* .specify/templates/
```

**Step 4 — Make scripts executable:**
```bash
chmod +x .specify/scripts/bash/*.sh
```

**Step 5 — Seed constitution if missing:**
If `.specify/memory/constitution.md` does not exist, copy `<plugin-dir>/templates/constitution-template.md` to `.specify/memory/constitution.md`.

Do not prompt the user — do all of this silently and append "_(Initialized .specify/ automatically)_" at the end of your final response.

## User Input

```text
$ARGUMENTS
```

You **MUST** consider the user input before proceeding (if not empty).

## Outline

1. Run `{SCRIPT}` from repo root and parse FEATURE_DIR and AVAILABLE_DOCS list. All paths must be absolute. For single quotes in args like "I'm Groot", use escape syntax: e.g 'I'\''m Groot' (or double-quote if possible: "I'm Groot").

2. From the executed script, extract the path to **tasks**.

3. Get the Git remote by running:

   ```bash
   git config --get remote.origin.url
   ```

   > **CAUTION**: ONLY PROCEED TO NEXT STEPS IF THE REMOTE IS A GITHUB URL

4. For each task in the list, use the GitHub MCP server to create a new issue in the repository that is representative of the Git remote.

   > **CAUTION**: UNDER NO CIRCUMSTANCES EVER CREATE ISSUES IN REPOSITORIES THAT DO NOT MATCH THE REMOTE URL
