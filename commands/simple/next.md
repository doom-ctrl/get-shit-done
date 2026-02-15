---
name: next
description: Execute only the current phase, then stop
argument-hint: ""
allowed-tools:
  - Read
  - Bash
  - Write
  - Task
  - AskUserQuestion
---

<objective>
Execute just one phase with full visibility, then stop.

**Flow:**
1. Read current phase from STATE.md
2. Run discuss → plan → execute → verify for current phase only
3. Stop and report completion

**Use when:** You want to see one phase at a time without auto-continuing.
</objective>

<process>
1. **READ STATE**
   - Read `.planning/STATE.md` to get current phase
   - Read `.planning/ROADMAP.md` to get total phases
   - Report: "Executing Phase [current] of [total]..."

2. **DISCUSS (if needed):**
   - Check if `{current}-CONTEXT.md` exists
   - If NOT exists:
     - Run: `/gsd:discuss-phase [current]`
     - Wait for full completion

3. **PLAN (if needed):**
   - Check if plan files exist
   - If NOT exists:
     - Run: `/gsd:plan-phase [current]`
     - Wait for full completion

4. **EXECUTE:**
   - Run: `/gsd:execute-phase [current]`
   - Wait for full completion

5. **VERIFY:**
   - Run: `/gsd:verify-work [current]`
   - Wait for full completion

6. **UPDATE STATE:**
   - Update `.planning/STATE.md` to increment phase
   - Create git commit

7. **STOP AND REPORT:**
   - Report: "Phase [current] ✓ complete."
   - If more phases remain:
     - Report: "Run `/next` to continue to Phase [next]."
     - Report: "Or run `/go` to auto-execute all remaining phases."
   - If all phases complete:
     - Run `/gsd:complete-milestone`
     - Report: "All phases complete! Milestone finished."
</process>
