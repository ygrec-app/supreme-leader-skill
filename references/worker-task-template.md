# Worker Task Template

Use this template when writing TASK.md files for each worker. Replace all placeholders.

---

# Your Task — Worker {ID}

You are Worker {ID} in a multi-agent team. Your team lead (the Supreme Leader) will check on you periodically.

## Task

{CLEAR DESCRIPTION OF WHAT TO BUILD}

## Requirements

{BULLETED LIST OF SPECIFIC REQUIREMENTS}

## Deliverable

Write your output to: `{ABSOLUTE_PATH_TO_DELIVERABLE}`

## Status Protocol

You MUST follow this protocol so the team lead can track your progress.

### Status File Location

`{ABSOLUTE_PATH}/tasks/worker-{id}.status`

### When You START

Write this JSON to your status file:

```json
{"worker":"{ID}","task":"{SHORT_TASK_DESCRIPTION}","status":"working"}
```

### When You FINISH

Update your status file to:

```json
{"worker":"{ID}","task":"{SHORT_TASK_DESCRIPTION}","status":"done","deliverable":"{RELATIVE_PATH_TO_DELIVERABLE}"}
```

### If You Get BLOCKED

If you have a question that prevents you from continuing, update your status file:

```json
{"worker":"{ID}","task":"{SHORT_TASK_DESCRIPTION}","status":"blocked","question":"Your specific question here"}
```

Then wait. The team lead will write an answer to `{ABSOLUTE_PATH}/work/worker-{id}/team-lead-response.txt`. Check for this file, read the answer, then continue working.

### If You Need to FIX Something

If the team lead sets your status to `"needs-fix"`, read the feedback at:
`{ABSOLUTE_PATH}/work/worker-{id}/team-lead-feedback.txt`

Fix the issues described, then update your status file back to `"done"`.

## Progress Tracking (for multi-file tasks)

{INCLUDE THIS SECTION ONLY IF THE TASK INVOLVES MORE THAN ~15 FILES}

Maintain a progress file at: `{ABSOLUTE_PATH}/work/worker-{id}/progress.json`

After completing each file (or batch of 5-10 files), update the progress file:

```json
{
  "total_files": 0,
  "completed_files": 0,
  "completed": ["path/to/file1.swift", "path/to/file2.swift"],
  "remaining": ["path/to/file3.swift"],
  "last_updated": "ISO timestamp"
}
```

**If you are resuming after a crash:** Read your progress file FIRST and skip already-completed files.
Do not redo work. Pick up exactly where you left off.

## Project Conventions

{INCLUDE THIS SECTION ONLY IF THE PROJECT HAS A CLAUDE.md OR SIMILAR CONFIG FILES}
{EXTRACT ONLY THE RULES RELEVANT TO THIS WORKER'S TASK — 5-15 LINES MAX}

Follow these project conventions in all code you write:

{RELEVANT RULES FROM CLAUDE.md, .editorconfig, linter configs, etc.}

## Important Rules

- Do NOT ask the human questions. Work autonomously.
- Write the status file FIRST (set to "working"), then do the work, then update status to "done".
- If you get stuck on a decision, use your best judgment rather than blocking.
- Keep your output focused — deliver exactly what's asked, nothing more.
- For multi-file tasks: update your progress file regularly so you can resume after interruptions.
