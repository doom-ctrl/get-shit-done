---
name: continue
description: Automatically continue where you left off (no prompts)
argument-hint: ""
allowed-tools:
  - Read
  - Bash
  - Write
  - Edit
  - Task
  - AskUserQuestion
---

<objective>
Automatically resume work from previous session without prompts.

**Flow:**
1. Load project state (interrupted agent info + phase progress)
2. Detect incomplete work (interrupted agents, mid-plan checkpoints, incomplete plans)
3. If nothing incomplete, find next incomplete phase
4. Execute the next logical action automatically
5. Continue until hitting a natural stopping point

**Use when:** You want to seamlessly continue without answering questions.
</objective>

<process>
1. **LOAD STATE**
   - Run: `node ~/.claude/get-shit-done/bin/arc-tools.js init resume`
   - Parse JSON for: `has_interrupted_agent`, `interrupted_agent_id`, `planning_exists`, `roadmap_exists`

   - Then run: `node ~/.claude/get-shit-done/bin/arc-tools.js init progress`
   - Parse JSON for: `current_phase`, `next_phase`, `has_work_in_progress`

2. **VALIDATE PROJECT**
   - If `planning_exists` is false:
     - Error: "No project found. Run `/a:init` or `/a:onward` first."
     - Exit
   - If `roadmap_exists` is false:
     - Error: "No roadmap found. Run `/a:init` or `/a:onward` first."
     - Exit

3. **DETECT INCOMPLETE WORK**

   **Priority 1: Interrupted Agent**
   - If `has_interrupted_agent` is true:
     - Display: "Resuming interrupted agent: {interrupted_agent_id}"
     - Use Task tool with `resume` parameter and the agent_id
     - Exit

   **Priority 2: Mid-plan Checkpoint**
   - Run: `ls .planning/phases/*/.continue-here*.md 2>/dev/null | head -1`
   - If a file is found:
     - Read the file to extract phase number from frontmatter
     - Display: "Resuming from mid-plan checkpoint in Phase {phase}"
     - Execute: `/arc:execute-phase {phase}`
     - Exit

   **Priority 3: Incomplete Plan**
   - Run: `node ~/.claude/get-shit-done/bin/arc-tools.js phase incomplete --format json`
   - If `has_incomplete` is true:
     - Get the first incomplete plan's phase
     - Display: "Completing incomplete plan in Phase {phase}"
     - Execute: `/arc:execute-phase {phase}`
     - Exit

4. **FIND NEXT INCOMPLETE PHASE**
   - If `current_phase` exists:
     - Set `target_phase` to `current_phase.number`
   - Else if `next_phase` exists:
     - Set `target_phase` to `next_phase.number`
   - Else:
     - Display: "All phases complete!"
     - Run: `/arc:complete-milestone`
     - Exit

5. **DETERMINE NEXT ACTION FOR PHASE**
   - Check for plans: `ls .planning/phases/{target_phase}-*/*-PLAN.md 2>/dev/null`
   - If no PLAN files exist:
     - Check for CONTEXT: `ls .planning/phases/{target_phase}-*/*-CONTEXT.md 2>/dev/null`

     **If CONTEXT missing:**
       - Display: "Gathering context for Phase {target_phase}..."
       - Execute: `/arc:discuss-phase {target_phase}`
       - Exit

     **If CONTEXT exists:**
       - Display: "Planning Phase {target_phase}..."
       - Execute: `/arc:plan-phase {target_phase}`
       - Exit

   - If PLAN files exist:
     - Display: "Executing Phase {target_phase}..."
     - Execute: `/arc:execute-phase {target_phase}`
     - Exit
</process>

<success_criteria>
- [ ] Project state loaded successfully
- [ ] Incomplete work detected and handled (if any)
- [ ] Next logical action executed automatically
- [ ] User knows what's happening without being prompted
- [ ] Continues until hitting a natural stopping point
</success_criteria>

<examples>
```bash
# Automatically continue where you left off
/a:continue

# Will automatically:
# - Resume interrupted agent (if any)
# - Complete mid-plan checkpoint (if any)
# - Finish incomplete plan (if any)
# - Gather context for next phase (if needed)
# - Plan next incomplete phase (if context exists)
# - Execute next incomplete phase (if planned)
# - Complete milestone (if all phases done)
```
</examples>
