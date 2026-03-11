# Supreme Leader Skill

A [Claude Code](https://claude.ai/claude-code) skill that orchestrates a team of parallel Claude Code workers via [cmux](https://www.cmux.dev/). It plans the work, spawns workers in split panes, monitors them autonomously, reviews deliverables, and dispatches fixes — all without human intervention after setup.

## How it works

1. **Plan** — Breaks your goal into independent subtasks, detects file conflicts, balances workloads
2. **Spawn** — Creates cmux panes, launches Claude Code instances, dispatches tasks
3. **Monitor** — Uses `/loop` to autonomously poll worker status, review completed work, unblock stuck workers
4. **Complete** — Writes a final report, cleans up worker sessions

The supreme leader never writes code itself — it delegates, reviews, and coordinates, like an engineering manager.

## Prerequisites

- [Claude Code](https://claude.ai/claude-code) v2.1.71+
- [cmux](https://www.cmux.dev/) — native macOS terminal for running multiple AI agents in parallel

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

## License

MIT
