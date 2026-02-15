---
name: go
description: Execute all remaining phases automatically with full step visibility
argument-hint: ""
allowed-tools:
  - Read
  - Bash
  - Write
  - Task
  - AskUserQuestion
---

<objective>
Execute all remaining phases autonomously while showing every step.

**Flow:**
1. Read current phase from STATE.md
2. For each remaining phase:
   - Run `/gsd:discuss-phase` if needed (you see context capture)
   - Run `/gsd:plan-phase` if needed (you see research & plans)
   - Run `/gsd:execute-phase` (you see implementation)
   - Run `/gsd:verify-work` (you see verification)
   - Auto-advance to next phase
3. Run `/gsd:complete-milestone` when all phases done

**You see everything** — no commands hidden, just automated sequencing.
</objective>

<process>
1. **READ STATE**
   - Read `.planning/STATE.md` to get current phase
   - Read `.planning/ROADMAP.md` to get total phases
   - Report: "Starting autonomous execution from Phase [current] of [total]..."

2. **MAIN EXECUTION LOOP**
   While current_phase <= total_phases:

   **PHASE [current_phase]:**
   
   a) **DISCUSS (if needed):**
      - Check if `{current_phase}-CONTEXT.md` exists
      - If NOT exists:
        - Run: `/gsd:discuss-phase [current_phase]`
        - Wait for full completion (see all questions & answers)
        - Confirm context file created
   
   b) **PLAN (if needed):**
      - Check if `{current_phase}-RESEARCH.md` and `{current_phase}-*-PLAN.md` exist
      - If NOT exists:
        - Run: `/gsd:plan-phase [current_phase]`
        - Wait for full completion (see research agents & plans)
        - Confirm plan files created
   
   c) **EXECUTE:**
      - Run: `/gsd:execute-phase [current_phase]`
      - Wait for full completion (see all code changes & commits)
      - Confirm execution summary created
   
   d) **VERIFY:**
      - Run: `/gsd:verify-work [current_phase]`
      - Wait for full completion (see verification results)
      - Confirm verification passed
   
   e) **ADVANCE:**
      - Update `.planning/STATE.md` to increment phase
      - Create checkpoint git commit: "checkpoint: Phase [current_phase] complete"
      - If more phases remain:
        - Report: "Phase [current_phase] ✓ complete. Starting Phase [next]..."
        - Automatically continue (loop back)
      - If last phase:
        - Report: "Phase [current_phase] ✓ complete. All phases done!"

3. **COMPLETE MILESTONE**
   - Run: `/gsd:complete-milestone`
   - Wait for completion
   - Report: "Milestone complete! All [total] phases built and verified."

4. **ERROR HANDLING**
   - If any phase fails verification:
     - Run `/gsd:debug` to diagnose
     - Attempt auto-fix and re-execute
     - If unresolvable, stop and report: "Stopped at Phase [N]. Run `/go` to retry."
</process>
