# Supreme Leader Skill

A [Claude Code](https://claude.ai/claude-code) skill that orchestrates a team of parallel Claude Code workers via [cmux](https://www.cmux.dev/). It plans the work, spawns workers in split panes, monitors them autonomously, reviews deliverables, and dispatches fixes — all without human intervention after setup.

## How it works

1. **Plan** — Distributes work across independent subtasks, preserves user-provided lists verbatim, detects file conflicts
2. **Spawn** — Creates cmux panes, launches Claude Code instances, dispatches tasks
3. **Monitor** — Uses `/loop` to autonomously poll worker status, review completed work, unblock stuck workers
4. **Complete** — Writes a final report, cleans up worker sessions

The supreme leader never writes code itself — it delegates, reviews, and coordinates, like an engineering manager.

## Prerequisites

- [Claude Code](https://claude.ai/claude-code) v2.1.71+
- [cmux](https://www.cmux.dev/) — native macOS terminal for running multiple AI agents in parallel

### cmux Permissions (optional)

If you're **not** running in bypass mode (`--dangerously-skip-permissions`), add these to your project's `.claude/settings.local.json` to auto-approve safe read-only cmux commands:

```json
{
  "permissions": {
    "allow": [
      "Bash(cmux --version)",
      "Bash(cmux ping)",
      "Bash(cmux list-workspaces)",
      "Bash(cmux list-panes *)",
      "Bash(cmux list-pane-surfaces *)",
      "Bash(cmux read-screen *)",
      "Bash(cmux surface-health *)",
      "Bash(cmux new-workspace)",
      "Bash(cmux new-split *)",
      "Bash(cmux rename-workspace *)",
      "Bash(cmux select-workspace *)",
      "Bash(cmux set-progress *)",
      "Bash(cmux clear-progress)",
      "Bash(cmux set-status *)",
      "Bash(cmux clear-status *)",
      "Bash(cmux log *)"
    ]
  }
}
```

Dangerous commands (`cmux send`, `cmux send-key`, `cmux close-workspace`) intentionally require manual approval — they can execute arbitrary text in worker terminals.

For fully autonomous orchestration: use `Bash(cmux *)` instead (less safe, but no prompts).

## Install

Clone into your Claude Code skills directory:

```bash
git clone https://github.com/ygrec-app/supreme-leader-skill.git ~/.claude/skills/supreme-leader
```

## Usage

In any Claude Code session running inside cmux:

```
/supreme-leader
```

Or describe what you want naturally:

- "Split this across multiple agents"
- "Orchestrate a team to build this"
- "Use cmux to parallelize this refactor"

## Examples

**Parallel refactoring:**
> "I have 4 files that need snake_case renamed to camelCase. Use your team to do them all in parallel."

**Building a project:**
> "Build a Node.js project with a utility library, an Express API, and unit tests. Split it across workers."

**With dependencies:**
> "Build a todo app: one worker does the data model, another the CLI (depends on data model), and a third writes docs."

## Architecture

### Project Structure

When the skill runs, it creates this structure in your project directory:

```
<project>/
├── tasks/
│   ├── supreme-leader-state.json    # Leader's persisted state (crash recovery)
│   ├── PROGRESS.md                  # Human-readable progress dashboard
│   ├── FINAL_REPORT.md              # Post-completion summary
│   └── worker-<id>.status           # JSON status file per worker
└── work/
    └── worker-<id>/                 # One directory per worker (IDs: a, b, c, ...)
        ├── TASK.md                  # Assignment (generated from template)
        ├── progress.json            # Checkpoint file for crash recovery
        ├── team-lead-feedback.txt   # Review feedback (leader → worker)
        └── team-lead-response.txt   # Answers to questions (leader → worker)
```

### Phases

**Phase 1: PLAN** — Distributes work across independent subtasks. If the user provides a specific list of items, every item is preserved verbatim — no additions, removals, merges, or substitutions. Worker count scales with list length (ceil(N/2-3) workers for N items). Detects file conflicts (no two workers may edit the same file — hard rule). Writes TASK.md files from a template, injecting relevant project conventions from CLAUDE.md. Validates item count matches before proceeding. Presents the plan for human approval.

**Phase 2: SPAWN** — Renames the leader's cmux workspace (e.g., `"Refactor (lead)"`), creates a separate team workspace (`"Refactor (team)"`). Splits panes into a grid (supports 2-8 workers). Launches `claude --dangerously-skip-permissions` in each pane. Dispatches tasks via `cmux send`. Writes `supreme-leader-state.json` with the full worker-to-surface mapping.

**Phase 3: MONITOR** — Sets up `/loop 1m` for autonomous polling. Each iteration: reads all `.status` files, updates `PROGRESS.md`, and reacts based on worker state:

| Status | Action |
|--------|--------|
| `working` | Skip — let them continue |
| `done` | Review the deliverable. Approve (`verified`) or reject (`needs-fix` + feedback file) |
| `blocked` | Read the question, write a response, notify worker via cmux |
| `needs-fix` | Re-check. Escalate to human after 3 rejections |
| `verified` | No action needed |
| `quota-exhausted` | Alert human, log to sidebar, wait for reset |

Dependent workers are spawned only when their prerequisites reach `verified`. Stuck workers (no status change for 3+ iterations) trigger a human alert. Real-time progress is pushed to the cmux sidebar via `set-progress`, `set-status`, and `log`.

**Phase 4: COMPLETE** — Updates `supreme-leader-state.json` with final statuses, writes `FINAL_REPORT.md`, cancels the `/loop` cron job, sends `/exit` to all worker panes, clears sidebar state. The team workspace is left open for inspection.

### Communication Protocol

Workers and the leader communicate exclusively through files — no direct messaging. Workers write JSON status files with a defined FSM:

```
(spawn) → working → done → verified
                      ↓
                 needs-fix → done → verified

working → blocked → (response) → working → done

working → quota-exhausted → (reset) → working → done
```

### Crash Recovery

Both the leader and workers persist their state to disk:
- **Leader:** `tasks/supreme-leader-state.json` — worker mappings, dependency graph, iteration count, timestamps
- **Workers:** `work/worker-<id>/progress.json` — completed/remaining files for multi-file tasks

On restart, the leader reads its state file, checks cmux for surviving panes, re-establishes the `/loop`, and continues without re-planning or re-spawning running workers.

## License

MIT
