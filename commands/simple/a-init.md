---
name: init
description: Initialize a new greenfield project (alias for /arc:new-project)
argument-hint: "[--auto]"
allowed-tools:
  - Read
  - Bash
  - Write
  - Task
  - AskUserQuestion
---

<objective>
Initialize a new greenfield project through unified flow: questioning → research (optional) → requirements → roadmap.

**Creates:**
- `.planning/PROJECT.md` — project context
- `.planning/config.json` — workflow preferences
- `.planning/research/` — domain research (optional)
- `.planning/REQUIREMENTS.md` — scoped requirements
- `.planning/ROADMAP.md` — phase structure
- `.planning/STATE.md` — project memory

**After this command:** Run `/a:plan` to start iterative phase planning.

**Note:** For brownfield projects (existing codebase), use `/a:onward` instead.
</objective>

<context>
**Flags:**
- `--auto` — Automatic mode. After config questions, runs research → requirements → roadmap without further interaction. Expects idea document via @ reference.

**Load project state if exists:**
Check for .planning/STATE.md - if project already initialized, warn user and offer to continue or abort.

**This command:**
- For greenfield projects only (new projects)
- Direct wrapper for /arc:new-project
- Delegates to new-project workflow
</context>

<execution_context>
@~/.claude/get-shit-done/workflows/new-project.md
@~/.claude/get-shit-done/references/questioning.md
@~/.claude/get-shit-done/references/ui-brand.md
@~/.claude/get-shit-done/templates/project.md
@~/.claude/get-shit-done/templates/requirements.md
</execution_context>

<process>
Execute the new-project workflow from @~/.claude/get-shit-done/workflows/new-project.md end-to-end.
Preserve all workflow gates (validation, approvals, commits, routing).
</process>

<success_criteria>
- [ ] Project validated (not already initialized)
- [ ] new-project workflow executed
- [ ] .planning/ directory created
- [ ] All project artifacts created
- [ ] User knows next steps (/a:plan to start planning)
</success_criteria>
