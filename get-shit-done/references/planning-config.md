<planning_config>

Configuration options for `.planning/` directory behavior.

<config_schema>
```json
"auto": {
  "enabled": false,
  "verification": true,
  "gap_closure": true,
  "max_verification_retries": 3,
  "checkpoint_interval": "phase"
},
"planning": {
  "commit_docs": true,
  "search_gitignored": false
},
"git": {
  "branching_strategy": "none",
  "phase_branch_template": "gsd/phase-{phase}-{slug}",
  "milestone_branch_template": "gsd/{milestone}-{slug}"
}
```

| Option | Default | Description |
|--------|---------|-------------|
| `auto.enabled` | `false` | Whether auto-mode features are enabled |
| `auto.verification` | `true` | Whether to run automated verification in auto-mode |
| `auto.gap_closure` | `true` | Whether to automatically close verification gaps |
| `auto.max_verification_retries` | `3` | Maximum iterations for gap closure before stopping |
| `auto.checkpoint_interval` | `"phase"` | How often to write checkpoints: `"phase"` |
| `commit_docs` | `true` | Whether to commit planning artifacts to git |
| `search_gitignored` | `false` | Add `--no-ignore` to broad rg searches |
| `git.branching_strategy` | `"none"` | Git branching approach: `"none"`, `"phase"`, or `"milestone"` |
| `git.phase_branch_template` | `"gsd/phase-{phase}-{slug}"` | Branch template for phase strategy |
| `git.milestone_branch_template` | `"gsd/{milestone}-{slug}"` | Branch template for milestone strategy |
</config_schema>

<auto_mode_behavior>

**Auto-mode configuration** controls the behavior of the `/auto` command, which runs the complete GSD pipeline hands-off.

**When `auto.enabled: true`:**
- `/auto` command uses auto-mode settings
- Overrides are still possible via command-line flags

**When `auto.enabled: false` (default):**
- `/auto` command still works but uses default behavior
- Users can override via flags (e.g., `--from-scratch`)

**Verification settings:**

**When `auto.verification: true` (default):**
- Automated verification runs after each phase execution
- Skips interactive UAT, runs programmatic checks instead
- Checks include: test suite execution, syntax validation, type checking

**When `auto.verification: false`:**
- Verification step is skipped entirely in auto-mode
- Not recommended unless you have external validation

**Gap closure settings:**

**When `auto.gap_closure: true` (default):**
- Automatically diagnoses verification gaps
- Creates gap plans via `plan-phase --gaps`
- Executes gap plans via `execute-phase --gaps-only`
- Re-verifies automatically
- Stops after `max_verification_retries` iterations

**When `auto.gap_closure: false`:**
- Verification gaps are logged but not auto-fixed
- `/auto` stops and reports issues for manual resolution

**Checkpoint settings:**

**`auto.checkpoint_interval: "phase"`** (default):
- Writes checkpoint at the start of each phase
- Checkpoint includes: phase number, step, timestamp
- Enables resumption across sessions
- Auto-uses checkpoint if < 1 hour old

**Checkpoint file location:** `.planning/.auto-checkpoint.json`

**Checkpoint format:**
```json
{
  "phase": "3",
  "phase_name": "authentication",
  "step": "executing",
  "timestamp": "2025-02-16T14:30:00Z",
  "wave": 2
}
```

**Resumption behavior:**
- Run `/auto` without arguments → resume from checkpoint (if recent)
- Run `/auto <phase>` → start from specific phase (ignore checkpoint)
- Run `/auto --from-scratch` → start from phase 1 (delete checkpoint)

**Example configuration:**

```json
{
  "auto": {
    "enabled": true,
    "verification": true,
    "gap_closure": true,
    "max_verification_retries": 3,
    "checkpoint_interval": "phase"
  }
}
```

**Example usage:**

```bash
# Auto-detect next phase and continue
/auto

# Start from phase 3
/auto 3

# Start from scratch, ignore progress
/auto --from-scratch
```

**Safety considerations:**

Auto-mode is "yolo mode" — it bypasses confirmation gates:
- No "Ready to continue?" prompts
- No "Review plans before executing?" prompts
- No manual intervention for gap closure

**Only stops on:**
- Execution failures (code crashes, syntax errors)
- Verification failures exceeding max retries
- Model API failures (after retries)
- User interrupt (Ctrl+C)

**Recommended for:**
- Experienced GSD users familiar with the pipeline
- Well-understood projects with clear requirements
- Automated execution of trusted plans
- Hands-off iteration through multiple phases

**Not recommended for:**
- New projects with ambiguous requirements
- Critical production systems without testing
- Situations requiring human review at each step

</auto_mode_behavior>

<commit_docs_behavior>

**When `commit_docs: true` (default):**
- Planning files committed normally
- SUMMARY.md, STATE.md, ROADMAP.md tracked in git
- Full history of planning decisions preserved

**When `commit_docs: false`:**
- Skip all `git add`/`git commit` for `.planning/` files
- User must add `.planning/` to `.gitignore`
- Useful for: OSS contributions, client projects, keeping planning private

**Using arc-tools.js (preferred):**

```bash
# Commit with automatic commit_docs + gitignore checks:
node ~/.claude/get-shit-done/bin/arc-tools.js commit "docs: update state" --files .planning/STATE.md

# Load config via state load (returns JSON):
INIT=$(node ~/.claude/get-shit-done/bin/arc-tools.js state load)
# commit_docs is available in the JSON output

# Or use init commands which include commit_docs:
INIT=$(node ~/.claude/get-shit-done/bin/arc-tools.js init execute-phase "1")
# commit_docs is included in all init command outputs
```

**Auto-detection:** If `.planning/` is gitignored, `commit_docs` is automatically `false` regardless of config.json. This prevents git errors when users have `.planning/` in `.gitignore`.

**Commit via CLI (handles checks automatically):**

```bash
node ~/.claude/get-shit-done/bin/arc-tools.js commit "docs: update state" --files .planning/STATE.md
```

The CLI checks `commit_docs` config and gitignore status internally — no manual conditionals needed.

</commit_docs_behavior>

<search_behavior>

**When `search_gitignored: false` (default):**
- Standard rg behavior (respects .gitignore)
- Direct path searches work: `rg "pattern" .planning/` finds files
- Broad searches skip gitignored: `rg "pattern"` skips `.planning/`

**When `search_gitignored: true`:**
- Add `--no-ignore` to broad rg searches that should include `.planning/`
- Only needed when searching entire repo and expecting `.planning/` matches

**Note:** Most GSD operations use direct file reads or explicit paths, which work regardless of gitignore status.

</search_behavior>

<setup_uncommitted_mode>

To use uncommitted mode:

1. **Set config:**
   ```json
   "planning": {
     "commit_docs": false,
     "search_gitignored": true
   }
   ```

2. **Add to .gitignore:**
   ```
   .planning/
   ```

3. **Existing tracked files:** If `.planning/` was previously tracked:
   ```bash
   git rm -r --cached .planning/
   git commit -m "chore: stop tracking planning docs"
   ```

</setup_uncommitted_mode>

<branching_strategy_behavior>

**Branching Strategies:**

| Strategy | When branch created | Branch scope | Merge point |
|----------|---------------------|--------------|-------------|
| `none` | Never | N/A | N/A |
| `phase` | At `execute-phase` start | Single phase | User merges after phase |
| `milestone` | At first `execute-phase` of milestone | Entire milestone | At `complete-milestone` |

**When `git.branching_strategy: "none"` (default):**
- All work commits to current branch
- Standard GSD behavior

**When `git.branching_strategy: "phase"`:**
- `execute-phase` creates/switches to a branch before execution
- Branch name from `phase_branch_template` (e.g., `gsd/phase-03-authentication`)
- All plan commits go to that branch
- User merges branches manually after phase completion
- `complete-milestone` offers to merge all phase branches

**When `git.branching_strategy: "milestone"`:**
- First `execute-phase` of milestone creates the milestone branch
- Branch name from `milestone_branch_template` (e.g., `gsd/v1.0-mvp`)
- All phases in milestone commit to same branch
- `complete-milestone` offers to merge milestone branch to main

**Template variables:**

| Variable | Available in | Description |
|----------|--------------|-------------|
| `{phase}` | phase_branch_template | Zero-padded phase number (e.g., "03") |
| `{slug}` | Both | Lowercase, hyphenated name |
| `{milestone}` | milestone_branch_template | Milestone version (e.g., "v1.0") |

**Checking the config:**

Use `init execute-phase` which returns all config as JSON:
```bash
INIT=$(node ~/.claude/get-shit-done/bin/arc-tools.js init execute-phase "1")
# JSON output includes: branching_strategy, phase_branch_template, milestone_branch_template
```

Or use `state load` for the config values:
```bash
INIT=$(node ~/.claude/get-shit-done/bin/arc-tools.js state load)
# Parse branching_strategy, phase_branch_template, milestone_branch_template from JSON
```

**Branch creation:**

```bash
# For phase strategy
if [ "$BRANCHING_STRATEGY" = "phase" ]; then
  PHASE_SLUG=$(echo "$PHASE_NAME" | tr '[:upper:]' '[:lower:]' | sed 's/[^a-z0-9]/-/g' | sed 's/--*/-/g' | sed 's/^-//;s/-$//')
  BRANCH_NAME=$(echo "$PHASE_BRANCH_TEMPLATE" | sed "s/{phase}/$PADDED_PHASE/g" | sed "s/{slug}/$PHASE_SLUG/g")
  git checkout -b "$BRANCH_NAME" 2>/dev/null || git checkout "$BRANCH_NAME"
fi

# For milestone strategy
if [ "$BRANCHING_STRATEGY" = "milestone" ]; then
  MILESTONE_SLUG=$(echo "$MILESTONE_NAME" | tr '[:upper:]' '[:lower:]' | sed 's/[^a-z0-9]/-/g' | sed 's/--*/-/g' | sed 's/^-//;s/-$//')
  BRANCH_NAME=$(echo "$MILESTONE_BRANCH_TEMPLATE" | sed "s/{milestone}/$MILESTONE_VERSION/g" | sed "s/{slug}/$MILESTONE_SLUG/g")
  git checkout -b "$BRANCH_NAME" 2>/dev/null || git checkout "$BRANCH_NAME"
fi
```

**Merge options at complete-milestone:**

| Option | Git command | Result |
|--------|-------------|--------|
| Squash merge (recommended) | `git merge --squash` | Single clean commit per branch |
| Merge with history | `git merge --no-ff` | Preserves all individual commits |
| Delete without merging | `git branch -D` | Discard branch work |
| Keep branches | (none) | Manual handling later |

Squash merge is recommended — keeps main branch history clean while preserving the full development history in the branch (until deleted).

**Use cases:**

| Strategy | Best for |
|----------|----------|
| `none` | Solo development, simple projects |
| `phase` | Code review per phase, granular rollback, team collaboration |
| `milestone` | Release branches, staging environments, PR per version |

</branching_strategy_behavior>

</planning_config>
