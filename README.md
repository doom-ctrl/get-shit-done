<div align="center">

# Arc knows

**Build software with AI — reliably.**

Claude Code is powerful. Arc knows makes it trustworthy.

[![npm version](https://img.shields.io/npm/v/arc-knows?style=for-the-badge&logo=npm&logoColor=white&color=CB3837)](https://www.npmjs.com/package/arc-knows)
[![npm downloads](https://img.shields.io/npm/dm/arc-knows?style=for-the-badge&logo=npm&logoColor=white&color=CB3837)](https://www.npmjs.com/package/arc-knows)
[![Discord](https://img.shields.io/badge/Discord-Join-5865F2?style=for-the-badge&logo=discord&logoColor=white)](https://discord.gg/5JJgD5svVS)
[![GitHub stars](https://img.shields.io/github/stars/glittercowboy/get-shit-done?style=for-the-badge&logo=github&color=181717)](https://github.com/glittercowboy/get-shit-done)

</div>

---

## What It Does

Arc knows fixes the main problem with AI coding: **context rot**. As Claude fills its context window, quality degrades and it starts making mistakes.

Arc knows solves this by:
- Breaking work into **atomic tasks** (each fits in a fresh context window)
- Using **subagents** for heavy lifting (keeps your session fast)
- Making **atomic git commits** (every task, reversible history)
- **Verifying results** (did it actually build what we said?)

You describe what you want. Arc knows handles the context engineering. Claude Code builds it.

---

## Quick Start

### Install

```bash
npx arc-knows@latest
```

Prompts you to choose:
1. **Runtime** — Claude Code, OpenCode, Gemini, or all
2. **Location** — Global (all projects) or local (current project)

**Recommended:** Run Claude Code with `--dangerously-skip-permissions` to avoid constant approval prompts.

### Verify

```
/arc:help
```

---

## How It Works

One simple loop: **discuss → plan → execute → verify**

```
┌─────────────────────────────────────────────────────────────┐
│ 1. INIT PROJECT                                              │
│ /arc:new-project                                            │
│                                                             │
│ Answer questions → Extract requirements → Create roadmap    │
└─────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────┐
│ 2. FOR EACH PHASE                                           │
│                                                             │
│ ┌──────────────────┐  ┌──────────────────┐  ┌─────────────┐ │
│ │ /arc:discuss-    │  │ /arc:plan-phase  │  │ /arc:execute-│ │
│ │   phase          │  │                 │  │   phase      │ │
│ │                  │  │                 │  │             │ │
│ │ Shape how       │  │ Research +      │  │ Execute all │ │
│ │ to build it     │  │ create plans    │  │ plans in    │ │
│ └──────────────────┘  └──────────────────┘  │ parallel    │ │
│                                         │  └─────────────┘ │
│                                         │         ↓         │
│                                         │ ┌─────────────┐ │
│                                         │ │/arc:verify- │ │
│                                         │ │ work        │ │
│                                         │ │             │ │
│                                         │ │ Test that   │ │
│                                         │ │ it works    │ │
│                                         │ └─────────────┘ │
│ ┌──────────────────┐                    │         ↓         │
│ │ /arc:complete-   │◄───────────────────┴─── repeat ────┘
│ │   milestone      │
│ │                  │
│ │ Archive & tag    │
│ │ release          │
│ └──────────────────┘
└─────────────────────────────────────────────────────────────┘
```

### Step 1: Initialize Project

```bash
/arc:new-project
```

**What happens:**
1. **Questions** — Asks until it understands your idea
2. **Research** — Spawns parallel agents to investigate the domain
3. **Requirements** — Extracts what's v1, v2, and out of scope
4. **Roadmap** — Creates phases mapped to requirements

**Creates:** `PROJECT.md`, `REQUIREMENTS.md`, `ROADMAP.md`, `STATE.md`

---

### Step 2: Discuss Phase (Optional but Recommended)

```bash
/arc:discuss-phase 1
```

**Why this matters:** A roadmap phase is just a sentence. That's not enough to build what *you* imagine.

This step captures your preferences:
- **Visual features** → Layout, interactions, animations
- **APIs/CLIs** → Response format, flags, errors
- **Content systems** → Structure, tone, depth

The deeper you go here, the more it builds *your* vision instead of reasonable defaults.

**Creates:** `{phase}-CONTEXT.md`

---

### Step 3: Plan Phase

```bash
/arc:plan-phase 1
```

**What happens:**
1. **Research** — Investigates how to implement (guided by your CONTEXT.md)
2. **Plan** — Creates 2-3 atomic task plans
3. **Verify** — Checks plans against requirements

Each plan is small enough to execute in a fresh context window. No quality degradation.

**Creates:** `{phase}-RESEARCH.md`, `{phase}-{N}-PLAN.md`

---

### Step 4: Execute Phase

```bash
/arc:execute-phase 1
```

**What happens:**
1. **Runs plans in waves** — Parallel where possible
2. **Fresh context per plan** — 200k tokens for implementation only
3. **Commits per task** — Every task gets its own commit
4. **Verifies** — Checks code delivers what was promised

Walk away. Come back to completed work with clean git history.

**Creates:** `{phase}-{N}-SUMMARY.md`

---

### Step 5: Verify Work

```bash
/arc:verify-work 1
```

**Does it actually work?** This is where you confirm.

1. **Extracts testable outcomes** — What you should be able to do now
2. **Walks through testing** — "Can you log in?" Yes/no, or describe what's wrong
3. **Diagnoses failures** — Spawns debug agents to find root causes
4. **Creates fix plans** — Ready for re-execution

If everything passes → next phase. If something's broken → re-execute with fix plans.

**Creates:** `{phase}-UAT.md`, fix plans if issues found

---

### Step 6: Complete Milestone

When all phases are done:

```bash
/arc:complete-milestone
/arc:new-milestone  # Start next version
```

Each milestone is a clean cycle: **define → build → ship**.

---

## Simplified Commands

### The Essentials

| Command | What it does |
|---------|--------------|
| `/arc:new-project` | Start new project (questions → roadmap) |
| `/arc:discuss-phase <N>` | Shape implementation decisions |
| `/arc:plan-phase <N>` | Research + create task plans |
| `/arc:execute-phase <N>` | Execute all plans, commit per task |
| `/arc:verify-work <N>` | Test that it actually works |
| `/arc:complete-milestone` | Archive milestone, tag release |
| `/arc:new-milestone` | Start next version |

### Brownfield Projects

Already have code? Map it first:

```bash
/arc:map-codebase    # Analyze existing codebase
/arc:new-project      # Then initialize as usual
```

### Quick Mode (Small Tasks)

```bash
/arc:quick
```

For bug fixes, small features, config changes. Same quality, faster path.

### Navigation

| Command | What it does |
|---------|--------------|
| `/arc:progress` | Where am I? What's next? |
| `/arc:help` | Show all commands |

---

## Why It Works

### Fresh Context Every Time

Each plan executes in a fresh 200k token context. No accumulated garbage, no "I'll be more concise now."

### Atomic Git Commits

```bash
abc123f feat(08-02): create login endpoint
def456g feat(08-02): add password hashing
hij789k feat(08-02): implement email confirmation
```

Each task independently revertible. Git bisect finds exact failing commit.

### Multi-Agent Architecture

Your session stays fast (30-40% context). Heavy work happens in subagents:
- 4 parallel researchers investigate domain
- Planner creates and refines plans
- Executors build in parallel
- Debuggers diagnose failures

You orchestrate. Agents execute.

---

## Configuration

Project settings in `.planning/config.json`. Configure during `/arc:new-project` or update with `/arc:settings`.

### Model Profiles

| Profile | Planning | Execution | Verification |
|---------|----------|-----------|--------------|
| `quality` | Opus | Opus | Sonnet |
| `balanced` (default) | Opus | Sonnet | Sonnet |
| `budget` | Sonnet | Sonnet | Haiku |

Switch: `/arc:set-profile budget`

### Workflow Toggles

| Setting | Default | What it does |
|---------|---------|--------------|
| `workflow.research` | `true` | Research before planning |
| `workflow.plan_check` | `true` | Verify plans before execution |
| `workflow.verifier` | `true` | Verify delivery after execution |

---

## Auto Mode (Hands-Off)

Want zero interaction? Use the simplified commands:

```bash
/init          # Initialize project
/plan          # Discuss + plan all phases
/go 1          # Execute + verify phase 1
/go 2          # Execute + verify phase 2
...
/auto          # Execute ALL remaining phases automatically
```

`/auto` runs the full pipeline: **discuss → plan → execute → verify → next phase**

Continues until all phases complete or error detected. Checkpoint-based resumption across sessions.

---

## Security

**Protect sensitive files** by adding to Claude Code's deny list:

```json
{
  "permissions": {
    "deny": [
      "Read(.env)",
      "Read(.env.*)",
      "Read(**/secrets/*)",
      "Read(**/*credential*)",
      "Read(**/*.pem)",
      "Read(**/*.key)"
    ]
  }
}
```

This prevents Claude from reading these files entirely.

---

## Update & Uninstall

```bash
# Update
npx arc-knows@latest

# Uninstall
npx arc-knows --claude --global --uninstall
```

---

<div align="center">

**Claude Code is powerful. Arc knows makes it reliable.**

</div>
