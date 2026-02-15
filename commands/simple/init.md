---
name: init
description: Initialize a new project and auto-plan all phases with full visibility
argument-hint: "[--auto]"
allowed-tools:
  - Read
  - Bash
  - Write
  - Task
  - AskUserQuestion
---

<context>
**Flags:**
- `--auto` — Automatic mode. After project creation, automatically plans all phases without further prompting.
</context>

<objective>
Start a new project and automatically plan all phases sequentially.

**Flow:**
1. Run `/gsd:new-project` to create PROJECT.md, REQUIREMENTS.md, ROADMAP.md, STATE.md
2. Read ROADMAP.md to determine total phases
3. For each phase 1 to N:
   - Run `/gsd:discuss-phase [N]` to capture context (you see all output)
   - Run `/gsd:plan-phase [N]` to create plans (you see research & plans)
4. Report completion when all phases are planned

**After this command:** Run `/go` to execute all phases automatically.
</objective>

<process>
1. **INITIALIZE PROJECT**
   - Run `/gsd:new-project` with provided arguments
   - Wait for completion (you'll see all questions, answers, and files created)

2. **READ PROJECT STATE**
   - Read `.planning/ROADMAP.md` to get total phase count
   - Read `.planning/STATE.md` to confirm current position

3. **PLAN ALL PHASES AUTONOMOUSLY**
   - For phase = 1 to total_phases:
     
     a) **DISCUSS PHASE [phase]:**
        - Run: `/gsd:discuss-phase [phase]`
        - Wait for full completion (you see all context questions & answers)
        - Confirm `{phase}-CONTEXT.md` was created
     
     b) **PLAN PHASE [phase]:**
        - Run: `/gsd:plan-phase [phase]`
        - Wait for full completion (you see research agents & plan creation)
        - Confirm `{phase}-RESEARCH.md` and `{phase}-*-PLAN.md` files exist
     
     c) **REPORT PROGRESS:**
        - Report: "Phase [phase] planned. ([phase]/[total] complete)"
   
   - Continue automatically to next phase without asking

4. **COMPLETION**
   - Report: "All [total] phases planned. Project ready for execution."
   - Report: "Run `/go` to execute all phases automatically with full visibility."
</process>
