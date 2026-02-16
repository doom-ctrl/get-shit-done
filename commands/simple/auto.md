---
name: auto
description: Automatically execute full pipeline (plan → execute → verify) for all remaining phases
argument-hint: "[phase-number] [--from-scratch]"
allowed-tools:
  - Read
  - Write
  - Edit
  - Glob
  - Grep
  - Bash
  - Task
---

<objective>
Run complete GSD pipeline hands-off. Auto-detects next incomplete phase, runs discuss → plan → execute → verify, continues to next phase.

**Fully automatic operation:**
- No confirmation prompts during execution
- Auto-approves all gates (phase transitions, plan acceptance, etc.)
- Only stops on unrecoverable errors (execution failures, max retries exceeded)
- Continues until all phases complete or error occurs
</objective>

<context>
Phase number: $ARGUMENTS (optional - if not provided, auto-detects next incomplete phase)

**Flags:**
- `--from-scratch` — Start from phase 1, ignore all progress

**Load project state:**
Check for .planning/STATE.md and .planning/ROADMAP.md

**Checkpoint system:**
- Saves position after each major step
- Enables resumption across sessions
- Auto-uses checkpoint if < 1 hour old

**This command:**
- Orchestrates discuss-phase, plan-phase, execute-phase, verify-work workflows
- Continues to next phase automatically after successful completion
- Handles verification gaps automatically (max 3 iterations per phase)
- Requires no user intervention (yolo mode)
</context>

<execution_context>
@.planning/STATE.md
@.planning/ROADMAP.md
@~/.claude/get-shit-done/workflows/auto.md
@~/.claude/get-shit-done/references/ui-brand.md
</execution_context>

<process>

## 1. Initialize and Validate

Load planning context:

```bash
INIT=$(node ~/.claude/get-shit-done/bin/arc-tools.js init progress)
```

Parse JSON for: `planning_exists`, `roadmap_exists`.

**If `planning_exists` is false:**
```
No project initialized. Run `/init` or `/onward` first.
```
Exit workflow.

**If `roadmap_exists` is false:**
```
No roadmap found. Run `/init` or `/onward` first.
```
Exit workflow.

## 2. Parse and Determine Starting Phase

Extract from $ARGUMENTS: phase number (optional), flags (`--from-scratch`).

**If `--from-scratch` flag present:**
- Start from phase 1 (first phase in roadmap)
- Display: "Starting from scratch (ignoring all progress)"

**If phase number provided:**
- Validate phase exists in roadmap
- Start from specified phase

**If no arguments provided:**
- Check for checkpoint file (`.planning/.auto-checkpoint.json`)
- If checkpoint exists and is recent (< 1 hour): resume from checkpoint
- Otherwise: auto-detect next incomplete phase via `auto-resume`

## 3. Execute Auto Workflow

Display banner:

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 Arc knows ► FULL AUTO MODE
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Starting Phase: {X}

This will run the complete pipeline for all remaining phases:
• discuss → plan → execute → verify → next phase

No user input required. Stops only on unrecoverable errors.

Press Ctrl+C to pause (checkpoint saved).
───────────────────────────────────────────────────────────────
```

Execute auto workflow:
```
Task(
  prompt="""Follow the auto workflow with starting phase: ${PHASE}.
           Flags: ${FLAGS}
           @~/.claude/get-shit-done/workflows/auto.md""",
  subagent_type="general-purpose",
  description="Auto-execute all remaining phases"
)
```

**On return:**
- **All phases complete** — Display success message (see step 4.1)
- **Stopped on error** — Display error and resumption info (see step 4.2)

## 4. Display Results

### 4.1 All Phases Complete

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 Arc knows ► AUTO ► COMPLETE ✓
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

All {N} phases completed successfully!

Progress: {N}/{N} phases complete

═════════════════════════════════════════════════════════

## Summary

Total phases: {N}
Phases completed: {N}
Execution time: {duration}

═════════════════════════════════════════════════════════

## Next Steps

Project is ready for deployment!

- `/arc:progress` — View detailed phase status
- `/check-todos` — Check for any remaining todos

═════════════════════════════════════════════════════════
```

### 4.2 Stopped on Error

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 Arc knows ► AUTO ► STOPPED
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

**Phase {X}: {Name}** — {error reason}

───────────────────────────────────────────────────────────────

## Resume

Checkpoint saved at: {step}

To continue, run:

\`\`\`
/auto
\`\`\`

This will resume from the checkpoint.

Or start from a specific phase:

\`\`\`
/auto {phase_number}
\`\`\`

───────────────────────────────────────────────────────────────

## Error Details

{detailed_error_message_from_workflow}

───────────────────────────────────────────────────────────────
```

## 5. Display Resumption Info (if stopped)

If workflow stopped before completion, also display:

```
**Also available:**
- `/auto {next_phase}` — Start from specific phase
- `/auto --from-scratch` — Start from phase 1
- `/arc:progress` — View detailed phase status
```

</process>

<success_criteria>
- [ ] Project initialization validated
- [ ] Starting phase determined (from arg, checkpoint, or auto-detect)
- [ ] Auto workflow executed
- [ ] Results displayed with appropriate routing
- [ ] User knows how to resume (if stopped) or next steps (if complete)
</success_criteria>

<examples>
```bash
# Auto-detect and continue from where project is
/auto

# Start from phase 3 specifically
/auto 3

# Start from phase 1, ignore all progress
/auto --from-scratch
```
</examples>
