---
name: plan
description: Plan all remaining phases automatically with full visibility
argument-hint: ""
allowed-tools:
  - Read
  - Bash
  - Write
  - Task
  - AskUserQuestion
---

<objective>
Plan all remaining phases autonomously while showing every step.

**Flow:**
1. Read current phase from STATE.md
2. For each unplanned phase:
   - Run `/gsd:discuss-phase` (you see context capture)
   - Run `/gsd:plan-phase` (you see research & plans)
   - Auto-advance to next phase
3. Report completion

**Use when:** You want to plan everything first, then execute later with `/a:go`.
</objective>

<process>
1. **READ STATE**
   - Read `.planning/STATE.md` to get current phase
   - Read `.planning/ROADMAP.md` to get total phases
   - Report: "Planning all phases from Phase [current] to [total]..."

2. **PLANNING LOOP**
   For phase from current_phase to total_phases:

   a) **CHECK IF ALREADY PLANNED:**
      - If `{phase}-RESEARCH.md` AND `{phase}-*-PLAN.md` exist:
        - Report: "Phase [phase] already planned. Skipping."
        - Continue to next phase

   b) **DISCUSS PHASE:**
      - Run: `/gsd:discuss-phase [phase]`
      - Wait for full completion (see all questions & answers)
      - Confirm `{phase}-CONTEXT.md` created

   c) **PLAN PHASE:**
      - Run: `/gsd:plan-phase [phase]`
      - Wait for full completion (see research agents & plan creation)
      - Confirm research and plan files created

   d) **REPORT PROGRESS:**
      - Report: "Phase [phase] planned. ([phase]/[total])"
      - Continue automatically to next phase

3. **COMPLETION**
   - Report: "All phases planned!"
   - Report: "Run `/a:go` to execute all phases automatically."
   - Report: "Or run `/a:next` to execute one phase at a time."
</process>
