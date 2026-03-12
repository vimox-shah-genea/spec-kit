---
description: Execute the implementation planning workflow using the plan template to generate design artifacts.
handoffs:
  - label: Create Tasks
    agent: speckit:tasks
    prompt: Break the plan into tasks
    send: true
  - label: Create Checklist
    agent: speckit:checklist
    prompt: Create a checklist for the following domain...
scripts:
  sh: .specify/scripts/bash/setup-plan.sh --json
  ps: .specify/scripts/powershell/setup-plan.ps1 -Json
agent_scripts:
  sh: .specify/scripts/bash/update-agent-context.sh __AGENT__
  ps: .specify/scripts/powershell/update-agent-context.ps1 -AgentType __AGENT__
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

1. **Setup**: Run `{SCRIPT}` from repo root and parse JSON for FEATURE_SPEC, IMPL_PLAN, SPECS_DIR, BRANCH. For single quotes in args like "I'm Groot", use escape syntax: e.g 'I'\''m Groot' (or double-quote if possible: "I'm Groot").

2. **Load context**: Read FEATURE_SPEC and `.specify/memory/constitution.md`. Load IMPL_PLAN template (already copied).

3. **Execute plan workflow**: Follow the structure in IMPL_PLAN template to:
   - Fill Technical Context (mark unknowns as "NEEDS CLARIFICATION")
   - Fill Constitution Check section from constitution
   - Evaluate gates (ERROR if violations unjustified)
   - Phase 0: Generate research.md (resolve all NEEDS CLARIFICATION)
   - Phase 1: Generate data-model.md, contracts/, quickstart.md
   - Phase 1: Update agent context by running the agent script
   - Re-evaluate Constitution Check post-design

4. **Stop and report**: Command ends after Phase 2 planning. Report branch, IMPL_PLAN path, and generated artifacts.

## Phases

### Phase 0: Outline & Research

1. **Extract unknowns from Technical Context** above:
   - For each NEEDS CLARIFICATION -> research task
   - For each dependency -> best practices task
   - For each integration -> patterns task

2. **Generate and dispatch research agents**:

   ```text
   For each unknown in Technical Context:
     Task: "Research {unknown} for {feature context}"
   For each technology choice:
     Task: "Find best practices for {tech} in {domain}"
   ```

3. **Consolidate findings** in `research.md` using format:
   - Decision: [what was chosen]
   - Rationale: [why chosen]
   - Alternatives considered: [what else evaluated]

**Output**: research.md with all NEEDS CLARIFICATION resolved

### Phase 1: Design & Contracts

**Prerequisites:** `research.md` complete

1. **Extract entities from feature spec** -> `data-model.md`:
   - Entity name, fields, relationships
   - Validation rules from requirements
   - State transitions if applicable

2. **Generate API contracts** from functional requirements:
   - For each user action -> endpoint
   - Use standard REST/GraphQL patterns
   - Output OpenAPI/GraphQL schema to `/contracts/`

3. **Agent context update**:
   - Run `{AGENT_SCRIPT}`
   - These scripts detect which AI agent is in use
   - Update the appropriate agent-specific context file
   - Add only new technology from current plan
   - Preserve manual additions between markers

**Output**: data-model.md, /contracts/\*, quickstart.md, agent-specific file

## Key rules

- Use absolute paths
- ERROR on gate failures or unresolved clarifications
