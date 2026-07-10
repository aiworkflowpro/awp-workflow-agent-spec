# Run-State Spec

> This document specifies `runs/`, `state/`, `output/`, `progress.json`, and the interruption-recovery mechanism.

## 1. Responsibility

Run state lets a Skill resume execution across sessions, contexts, and Agents.

Applicable scenarios:

- Multi-step tasks.
- Batch tasks.
- Longer-running tasks.
- Tasks that produce multiple intermediate artifacts.
- Tasks that must continue after a failure.

A short task may skip the run directory, but if restarting after an interruption wastes noticeable time, create one.

## 2. Directory Structure

```text
runs/
  {keyword}-YYYYMMDD-HHMMSS/
    state/
      config.json
      progress.json
    step01-prepare/
    step02-execute/
    step03-verify/
    output/
```

Fixed directories:

```text
state/
  Run config, progress, errors, and recovery hints.

output/
  Final artifacts.
```

Dynamic directories:

```text
stepNN-{action}/
  Aligned with workflow/stepNN-{action}.md.
```

## 3. Run-Directory Naming

Format:

```text
{keyword}-YYYYMMDD-HHMMSS
```

Multi-mode:

```text
{mode}-{keyword}-YYYYMMDD-HHMMSS
```

Examples:

```text
markdown-release-20260430-091500
audit-agent-skill-20260430-101200
fix-readme-20260430-103000
```

Do not use a timestamp alone. The `keyword` identifies the task on recovery.

## 4. keyword Rules

The `keyword` comes from this task's core input:

- Article title.
- Repository name.
- File name.
- Search term.
- User name.
- Product name.
- Mode name.

Normalization:

- Lowercase.
- Spaces to hyphens.
- Strip special characters.
- Truncate if too long.
- For non-Latin text, keep the core romanization or the English part.

## 5. config.json

`state/config.json` holds this run's parameters.

```json
{
  "target": "/path/to/project",
  "mode": "release-review",
  "keyword": "project-review",
  "created_at": "2026-04-30T09:15:00Z",
  "run_dir": "runs/project-review-20260430-091500"
}
```

Rules:

- Write this run's input here.
- No real secrets.
- No private paths beyond publishable files, unless the user explicitly provided them for this run and they will not enter a public repository.
- Annotating the source of each inferred value is better.

## 6. progress.json

Minimal fields:

```json
{
  "status": "processing",
  "current_step": "step02-execute",
  "completed_steps": ["step01-prepare"],
  "failed_steps": [],
  "skipped_steps": [],
  "outputs": {
    "config": "state/config.json",
    "file_index": "step01-prepare/file-index.txt"
  },
  "resume_hint": "Continue with step02-execute; read state/config.json and step01-prepare/file-index.txt."
}
```

Recommended fields:

```text
status
current_step
completed_steps
failed_steps
skipped_steps
outputs
errors
resume_hint
created_at
updated_at
```

## 7. status

```text
pending
  Created, not yet started.

processing
  Currently executing.

completed
  Done.

failed
  Failed and needs handling.
```

Do not use fuzzy statuses like `done?` or `maybe_failed`.

## 8. Update Timing

Must update:

- After the run directory is created.
- Before each step starts.
- After each step completes.
- After each step fails.
- When a step is skipped.
- On final completion.

Principles:

- Write the artifact first, then update state.
- Keep state updates as atomic as possible.
- Also write `progress.json` on failure.

## 9. Recovery Flow

After an interruption:

1. Find the latest or the specified run directory.
2. Read `state/progress.json`.
3. Check `status`.
4. Check that the artifacts of `completed_steps` exist.
5. Continue from `current_step`.
6. Do not re-run completed steps unless the user explicitly asks.

Do not rely on chat history during recovery.

## 10. skipped_steps

Skipped steps must record a reason.

```json
{
  "skipped_steps": [
    {
      "step": "step03-fix",
      "reason": "The user asked for a review only, not changes."
    }
  ]
}
```

## 11. errors

An error record should include:

```json
{
  "errors": [
    {
      "step": "step02-execute",
      "type": "permission_denied",
      "path": "private-file.txt",
      "recoverable": true,
      "message": "File not readable; skipped."
    }
  ]
}
```

Do not record real secret values.

## 12. output/

Write all final artifacts into `output/`.

Recommended:

```text
output/
  final-report.md
  blockers.json
  summary.json
```

The final reply should reference these artifacts, not just output the result in the conversation.

## 13. Latest Pointers

Use stable pointers when iterating repeatedly:

```text
draft_latest.md
feedback_latest.md
report_latest.json
```

Do not write variable arithmetic in prompts, e.g. `{round_num-1}`. To read the previous round's result, read the latest-pointer file.

## 14. Multi-Mode Runs

A multi-mode Skill distinguishes runs by a mode prefix:

```text
runs/
  check-article-a-20260430-091500/
  fix-article-a-20260430-093000/
  audit-project-b-20260430-100000/
```

Do not let different modes write into the same run directory.

## 15. Cleanup Strategy

A public Skill may recommend:

- Keeping the last N runs.
- Letting the user manually delete old run directories.
- Not deleting user data automatically.
- Generating a to-delete list before cleanup.

Do not clear `runs/` by default.

## 16. Completion Report

On completion, report:

```text
Status: completed
Run directory: runs/project-review-20260430-091500
Final artifact: output/final-report.md
Verification: ran JSON parsing, link checking, secret scanning
Remaining risk: network check not run because the user disabled network access
```

## 17. Prohibitions

- Recording progress only in the chat.
- A run directory with a timestamp but no keyword.
- Not writing `progress.json` on failure.
- Skipping steps without recording a reason.
- Final output scattered across step directories.
- Automatically deleting old run data.
- Writing real credentials into run state.

## 18. Checklist

```text
Directories
  [ ] runs/ exists, or a short task explicitly does not need it.
  [ ] state/ exists.
  [ ] output/ exists.
  [ ] Step directories align with workflow files.

State
  [ ] progress.json is parseable.
  [ ] status is valid.
  [ ] current_step is accurate.
  [ ] completed_steps match the actual artifacts.
  [ ] resume_hint is executable.

Recovery
  [ ] No reliance on chat history after an interruption.
  [ ] Completed steps are not re-run.
  [ ] Errors and skipped items are recorded.
```
