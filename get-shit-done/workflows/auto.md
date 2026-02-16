<purpose>
Orchestrate full automation pipeline: plan → execute → verify → next phase.
Continue until all phases complete or unrecoverable error.

This workflow runs the complete GSD pipeline hands-off. Auto-detects the next incomplete phase, runs discuss → plan → execute → verify, then continues to the next phase.
</purpose>

<required_reading>
Read all files referenced by the invoking prompt's execution_context before starting.

@~/.claude/get-shit-done/references/ui-brand.md
</required_reading>

<process>

<step name="initialize" priority="first">
Load all context in one call:

```bash
INIT=$(node ~/.claude/get-shit-done/bin/arc-tools.js init progress --include state,roadmap,config)
```

Parse JSON for: `project_exists`, `roadmap_exists`, `state_exists`, `phases`, `next_phase`, `completed_count`, `phase_count`, `config_content`.

**File contents (from --include):** `state_content`, `roadmap_content`, `config_content`.

**If `project_exists` is false:**
```
No project initialized. Run `/init` or `/onward` first.
```
Exit workflow.

**If `roadmap_exists` is false:**
```
No roadmap found. Run `/init` or `/onward` first.
```
Exit workflow.

**If all phases complete:**
```
All phases are already complete!

Use `/arc:progress` to verify.
```
Exit workflow.
</step>

<step name="determine_starting_phase">
Parse $ARGUMENTS for phase number and flags:

- **First argument**: Optional phase number (e.g., "3")
- **Flags**: `--from-scratch` (ignore progress, start from phase 1)

**If `--from-scratch` flag present:**
- Set `CURRENT_PHASE` to first phase from roadmap
- Delete any existing checkpoint file:
  ```bash
  rm -f .planning/.auto-checkpoint.json
  ```
- Proceed to `main_loop`

**If phase number provided:**
- Validate phase exists in roadmap
- Set `CURRENT_PHASE` to provided phase
- Proceed to `main_loop`

**If no arguments provided:**
Check for valid checkpoint:

```bash
CHECKPOINT=$(node ~/.claude/get-shit-done/bin/arc-tools.js checkpoint-read)
```

Parse JSON for: `exists`, `is_stale`, `checkpoint`.

**If checkpoint exists and not stale:**
- Set `CURRENT_PHASE` from `checkpoint.phase`
- Display: "Resuming from Phase {CURRENT_PHASE} (checkpointed at {checkpoint.step})"
- Proceed to `main_loop` (skip discuss-phase if checkpoint.step is after planning)

**Otherwise (no checkpoint or stale):**
Use auto-resume:

```bash
RESUME=$(node ~/.claude/get-shit-done/bin/arc-tools.js auto-resume)
```

Parse JSON for: `resume_phase`, `reason`.

**If `resume_phase` is null:**
```
Could not determine next phase to work on.

Use `/arc:progress` to see phase status.
```
Exit workflow.

**Set `CURRENT_PHASE` to `resume_phase` and proceed to `main_loop`.**
</step>

<step name="main_loop">
**For each incomplete phase starting with CURRENT_PHASE:**

Initialize iteration count for gap closure: `gap_iteration = 0`

---

### 1. Update Checkpoint

```bash
node ~/.claude/get-shit-done/bin/arc-tools.js checkpoint-write "${PHASE}" "starting" '{"phase_name":"${PHASE_NAME}","step":"initialization"}'
```

---

### 2. Run Discuss-Phase (skip if resuming from checkpoint after planning)

**Only if checkpoint step is "initialization" or "discussing":**

```bash
node ~/.claude/get-shit-done/bin/arc-tools.js checkpoint-write "${PHASE}" "discussing" '{}'
```

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 Arc knows ► AUTO ► DISCUSSING PHASE {X}
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

Spawn arc-planner (actually runs discuss-phase which is conversational):
```
Task(
  prompt="""Follow the discuss-phase workflow for Phase ${PHASE}.
           @~/.claude/get-shit-done/workflows/discuss-phase.md""",
  subagent_type="general-purpose",
  description="Discuss Phase ${PHASE} decisions"
)
```

**On return:**
- **Discussion complete** — Proceed to step 3
- **Discussion failed** — Log error, STOP workflow

---

### 3. Run Plan-Phase (skip if resuming from checkpoint after planning)

**Only if checkpoint step is before "planning":**

```bash
node ~/.claude/get-shit-done/bin/arc-tools.js checkpoint-write "${PHASE}" "planning" '{}'
```

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 Arc knows ► AUTO ► PLANNING PHASE {X}
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

Spawn arc-planner:
```
Task(
  prompt="""Follow the plan-phase workflow for Phase ${PHASE}.
           @~/.claude/get-shit-done/workflows/plan-phase.md""",
  subagent_type="arc-planner",
  model="{planner_model}",
  description="Plan Phase ${PHASE}"
)
```

**On return:**
- **Planning complete** — Proceed to step 4
- **Planning failed** — Log error, STOP workflow

---

### 4. Run Execute-Phase (skip if resuming from checkpoint after execution)

**Only if checkpoint step is before "executing":**

```bash
node ~/.claude/get-shit-done/bin/arc-tools.js checkpoint-write "${PHASE}" "executing" '{}'
```

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 Arc knows ► AUTO ► EXECUTING PHASE {X}
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

Spawn arc-executor:
```
Task(
  prompt="""Follow the execute-phase workflow for Phase ${PHASE}.
           @~/.claude/get-shit-done/workflows/execute-phase.md""",
  subagent_type="general-purpose",
  description="Execute Phase ${PHASE}"
)
```

**On return:**
- **Execution complete** — Proceed to step 5
- **Execution failed** — Log error, STOP workflow (do not auto-retry execution failures)

---

### 5. Run Verify-Work (in auto-mode)

```bash
node ~/.claude/get-shit-done/bin/arc-tools.js checkpoint-write "${PHASE}" "verifying" '{}'
```

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 Arc knows ► AUTO ► VERIFYING PHASE {X}
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

Spawn verification agent with auto-mode flag:
```
Task(
  prompt="""Follow the verify-work workflow for Phase ${PHASE} in auto-mode.
           @~/.claude/get-shit-done/workflows/verify-work.md

           IMPORTANT: Run in auto-mode (no user prompts). If verification gaps are found:
           1. Automatically run plan-phase with --gaps flag
           2. Automatically run execute-phase with --gaps-only flag
           3. Re-verify automatically
           4. Max 3 iterations before stopping""",
  subagent_type="general-purpose",
  description="Verify Phase ${PHASE} (auto-mode)"
)
```

**On return:**
- **All tests passed** — Proceed to step 6
- **Issues found and fixed** — Proceed to step 6
- **Issues found, max retries exceeded** — STOP workflow, report to user
- **Verification failed (non-retryable error)** — STOP workflow, report to user

---

### 6. Update STATE.md and Checkpoint

Mark phase as complete in STATE.md:

```bash
node ~/.claude/get-shit-done/bin/arc-tools.js phase complete "${PHASE}"
```

Delete checkpoint file (phase complete, no longer needed):

```bash
rm -f .planning/.auto-checkpoint.json
```

---

### 7. Continue to Next Phase

Analyze roadmap for next incomplete phase:

```bash
ROADMAP_ANALYSIS=$(node ~/.claude/get-shit-done/bin/arc-tools.js roadmap analyze)
```

Parse JSON for: `next_phase`.

**If `next_phase` is null:**
```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 Arc knows ► AUTO ► ALL PHASES COMPLETE ✓
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

All {N} phases completed successfully!

Project is ready for deployment.
```
Exit workflow with success message.

**If `next_phase` exists:**
Set `CURRENT_PHASE = next_phase`
Loop back to step 1 (Update Checkpoint)

---

</step>

<step name="handle_stop">
**If workflow stops due to error:**

Display summary:

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 Arc knows ► AUTO ► STOPPED
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

**Phase {X}: {Name}** — {reason for stopping}

Checkpoint saved at: {step}

───────────────────────────────────────────────────────────────

## Resume

Run \`/auto\` again to continue from checkpoint.

Or run \`/auto {next_phase}\` to start from a specific phase.

───────────────────────────────────────────────────────────────
```

Checkpoint file preserves position for seamless resumption.
</step>

</process>

<error_handling>
**Workflow stops on:**

1. **Execution failures** — Code crashes, syntax errors, runtime exceptions
   - DO NOT auto-retry execution failures
   - User must investigate and fix
   - Checkpoint saves position before execution

2. **Verification failures (max retries)** — After 3 gap-closure iterations
   - Gap closure requires human intervention
   - Checkpoint saves position at verification step

3. **Model API failures** — After retries (handled by individual agents)
   - Network issues, rate limits, service outages

4. **User interrupt** — Ctrl+C during execution
   - Checkpoint saves current position
   - Resume with `/auto`

**Workflow continues through:**

1. **Plan verification failures** — Auto-retry with arc-plan-checker (max 3)
2. **Verification gaps** — Auto-plan, auto-execute, re-verify (max 3 iterations)
3. **Agent context refresh** — Each subagent runs in fresh context window

**Workflow bypasses confirmation gates:**

1. **Phase transitions** — No "Ready to continue?" prompts
2. **Plan acceptance** — No "Review plans before executing?" prompts
3. **Verification routing** — No manual intervention for gap closure

This is "yolo" mode — hands-off automation until unrecoverable error.
</error_handling>

<success_criteria>
- [ ] Project initialization validated
- [ ] Starting phase determined (from arg, checkpoint, or auto-resume)
- [ ] For each incomplete phase:
  - [ ] Checkpoint updated at each step
  - [ ] discuss-phase workflow executed (unless resuming)
  - [ ] plan-phase workflow executed (unless resuming)
  - [ ] execute-phase workflow executed (unless resuming)
  - [ ] verify-work workflow executed in auto-mode
  - [ ] Gaps auto-closed (max 3 iterations) or stopped with clear message
  - [ ] STATE.md updated with phase completion
  - [ ] Checkpoint deleted on successful completion
- [ ] Continued to next phase or displayed success message
- [ ] User knows how to resume if stopped
</success_criteria>

<checkpoint_format>
Checkpoint file: `.planning/.auto-checkpoint.json`

```json
{
  "phase": "3",
  "phase_name": "authentication",
  "step": "executing",
  "timestamp": "2025-02-16T14:30:00Z",
  "wave": 2,
  "last_activity": "Running wave 2 (plans 03-02, 03-03)"
}
```

**Step values:**
- `initialization` — Starting phase
- `discussing` — Running discuss-phase workflow
- `planning` — Running plan-phase workflow
- `executing` — Running execute-phase workflow
- `verifying` — Running verify-work workflow

**Checkpoint is stale if:** Age > 1 hour (60 minutes)
</checkpoint_format>
