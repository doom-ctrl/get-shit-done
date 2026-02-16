---
name: go
description: Execute and verify a phase (one phase at a time)
argument-hint: "<phase-number> [--gaps-only]"
allowed-tools:
  - Read
  - Write
  - Edit
  - Glob
  - Grep
  - Bash
  - Task
  - AskUserQuestion
---

<objective>
Execute all plans in a phase, then automatically verify the work.

**One phase at a time only** — requires explicit phase number.

**Workflow:**
1. Validate phase number exists in roadmap
2. Run execute-phase workflow (execute all plans)
3. After successful execution, automatically run verify-work workflow
4. Display routing based on verification results

**If verification finds issues:**
- Show fix plan options and gap closure workflow

**If verification passes:**
- Show next phase message
</objective>

<context>
Phase number: $ARGUMENTS (required)

**Flags:**
- `--gaps-only` — Execute only gap closure plans (after verify-work creates fix plans)

**Load project state:**
Check for .planning/STATE.md and .planning/ROADMAP.md

**This command:**
- Requires phase number argument
- Validates phase exists before execution
- Runs execute-phase workflow
- Automatically runs verify-work workflow after execution
- Routes based on verification results
- Preserves all workflow gates
</context>

<execution_context>
@.planning/STATE.md
@.planning/ROADMAP.md
@~/.claude/get-shit-done/workflows/execute-phase.md
@~/.claude/get-shit-done/workflows/verify-work.md
@~/.claude/get-shit-done/references/ui-brand.md
</execution_context>

<process>

## 1. Initialize and Validate Phase

Load planning context:

```bash
INIT=$(node ~/.claude/get-shit-done/bin/arc-tools.js init progress)
```

Parse JSON for: `planning_exists`, `roadmap_exists`.

**If `planning_exists` is false:**
```
No project initialized. Run `/a:init` or `/a:onward` first.
```
Exit workflow.

**If `roadmap_exists` is false:**
```
No roadmap found. Run `/a:init` or `/a:onward` first.
```
Exit workflow.

## 2. Parse and Validate Arguments

Extract from $ARGUMENTS: phase number (required), flags (`--gaps-only`).

**If no phase number provided:**
```
Phase number required.

Usage: /a:go <phase-number> [--gaps-only]

Example: /a:go 1
```
Exit workflow.

## 3. Validate Phase Exists

Validate phase against roadmap:

```bash
PHASE_INFO=$(node ~/.claude/get-shit-done/bin/arc-tools.js roadmap get-phase "${PHASE_NUMBER}")
```

Parse JSON for: `found`, `phase_number`, `phase_name`, `goal`.

**If `found` is false:**
```
Phase ${PHASE_NUMBER} not found in roadmap.

Use `/arc:progress` to see available phases.
```
Exit workflow.

**If `found` is true:** Continue to step 4.

## 4. Execute Phase

Display banner:
```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 Arc knows ► EXECUTING PHASE {X}
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

◆ Phase {X}: {Name}
◆ {goal}
```

Execute execute-phase workflow:
```
Task(
  prompt="""Follow the execute-phase workflow for Phase ${PHASE_NUMBER} with flags: ${FLAGS}.
           @~/.claude/get-shit-done/workflows/execute-phase.md""",
  subagent_type="general-purpose",
  description="Execute Phase ${PHASE_NUMBER}"
)
```

**On return:**
- **Execution completed successfully** — Continue to step 5
- **Execution failed** — Display error and exit
- **Execution aborted** — Exit and display summary

## 5. Verify Work

After successful execution, automatically run verification.

Display banner:
```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 Arc knows ► VERIFYING WORK
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

Execute verify-work workflow:
```
Task(
  prompt="""Follow the verify-work workflow for Phase ${PHASE_NUMBER}.
           @~/.claude/get-shit-done/workflows/verify-work.md""",
  subagent_type="general-purpose",
  description="Verify Phase ${PHASE_NUMBER}"
)
```

**On return:**
- **All tests passed** — Continue to step 6.1
- **Issues found** — Continue to step 6.2

## 6. Display Results and Next Steps

### 6.1 All Tests Passed

```bash
ROADMAP_ANALYSIS=$(node ~/.claude/get-shit-done/bin/arc-tools.js roadmap analyze)
```

Parse JSON for: `next_incomplete_phase`, `total_phases`, `completed_phases`.

**Output format:**

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 Arc knows ► PHASE {X} COMPLETE ✓
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

**Phase {X}: {Name}** — All tests passed

Progress: {completed_phases}/{total_phases} phases complete

───────────────────────────────────────────────────────────────

## ▶ Next Up

[If next_incomplete_phase exists:]
**Execute Phase {next_incomplete_phase}** — continue execution

`/a:go {next_incomplete_phase}`

<sub>`/clear` first → fresh context window</sub>

---

[If all phases complete:]
**All phases complete!** 🎉

Project is ready for deployment.

---

**Also available:**
- `/arc:progress` — View detailed phase status
- `/a:plan` — Plan remaining phases
- `/a:check-todos` — Check pending todos

---
```

End workflow.

### 6.2 Issues Found

Display:
```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 Arc knows ► ISSUES FOUND
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Verification found {N} issue(s).

[Summary of issues from verify-work workflow]

Fix plans have been created for Phase {PHASE_NUMBER}.

───────────────────────────────────────────────────────────────

## ▶ Next Up

**Execute gap fixes** — run fix plans to resolve issues

`/a:go {PHASE_NUMBER} --gaps-only`

<sub>`/clear` first → fresh context window</sub>

---

**Also available:**
- Review fix plans: `cat .planning/phases/{phase-dir}/*-PLAN.md`
- Re-verify after manual fixes: `/a:go {PHASE_NUMBER} --gaps-only`

---
```

End workflow.

</process>

<success_criteria>
- [ ] Project initialization validated
- [ ] Phase number argument provided
- [ ] Phase validated against roadmap
- [ ] execute-phase workflow executed
- [ ] verify-work workflow executed automatically
- [ ] Results displayed with appropriate routing
- [ ] User knows next steps (execute next phase or fix gaps)
</success_criteria>
