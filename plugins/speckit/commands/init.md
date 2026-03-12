---
description: Initialize the .specify/ directory structure in this project by copying scripts and templates from the installed speckit plugin
---

Set up the speckit scaffold for this project.

## Steps

1. Locate the installed speckit plugin directory by running:
   ```bash
   find ~/.claude/plugins/cache -name "plugin.json" -path "*speckit*" 2>/dev/null | head -1 | xargs dirname 2>/dev/null
   ```
   Use the returned path as `<plugin-dir>`.

2. If nothing is returned, tell the user: "Could not find the installed speckit plugin. Please ensure it is installed with `claude plugin install speckit`." Then stop.

3. Create the following directories if they do not already exist:
   - `.specify/scripts/bash/`
   - `.specify/templates/`
   - `.specify/memory/`

4. Copy all files from `<plugin-dir>/scripts/bash/` into `.specify/scripts/bash/` (skip files that already exist, unless `--force` was passed as an argument).

5. Copy all files from `<plugin-dir>/templates/` into `.specify/templates/` (skip files that already exist, unless `--force` was passed).

6. Make all `.sh` files in `.specify/scripts/bash/` executable:
   ```bash
   chmod +x .specify/scripts/bash/*.sh
   ```

7. If `.specify/memory/constitution.md` does not exist, copy `<plugin-dir>/templates/constitution-template.md` to `.specify/memory/constitution.md`.

8. Report what was created/skipped with a short summary, e.g.:
   ```
   ✓ .specify/ initialized
     scripts/bash/    — 5 files copied
     templates/       — 6 files copied
     memory/          — constitution.md created
   ```

> If `--force` is passed, overwrite existing files instead of skipping them.
