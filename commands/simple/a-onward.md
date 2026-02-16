---
name: onward
description: Map existing codebase and initialize new project
argument-hint: ""
allowed-tools:
  - Read
  - Bash
  - Write
  - Task
  - AskUserQuestion
---

<objective>
Start working with an existing codebase.

**Flow:**
1. Run `/gsd:map-codebase` to analyze existing code (you see analysis)
2. Run `/a:init` to create project and plan all phases

**Use when:** You have existing code and want to add new features using GSD.
</objective>

<process>
1. **MAP CODEBASE**
   - Run: `/gsd:map-codebase`
   - Wait for full completion
   - You will see the parallel analysis of your stack, architecture, conventions
   - Confirm analysis files created in `.planning/`

2. **INITIALIZE PROJECT**
   - Run: `/a:init`
   - Wait for full completion
   - You will see project creation and all phases being planned

3. **COMPLETION**
   - Report: "Codebase mapped and project initialized."
   - Report: "Run `/a:go` to execute all phases automatically."
</process>
