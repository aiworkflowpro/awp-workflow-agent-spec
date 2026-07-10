# Workflow Step Spec

> This document specifies the naming, structure, executors, inputs and outputs, verification, and scheduling of `workflow/step*.md` step files.

## 1. Responsibility

`workflow/` holds step-execution documents. Each step is a standalone contract that defines:

- Why this step exists.
- Who executes it.
- Which inputs it reads.
- Which outputs it writes.
- How to tell it is complete.
- How to handle failure.
- What the next step is.

A step file is not background material. It should let another Agent continue without reading the whole session.

## 2. Directory Responsibility

```text
workflow/
  step01-prepare.md
  step02-execute.md
  step03-verify.md
```

May contain:

- Step-execution instructions.
- Input/output conventions.
- Verification checkpoints.
- Scheduling notes.

Do not put:

- Long prompt templates. Put them in `references/prompts/`.
- Script code. Put it in `scripts/`.
- Large background material. Put it in `references/` or `docs/`.
- Run artifacts. Put them in `runs/`.

## 3. Naming Convention

```text
stepNN-{action}.md
```

Rules:

- `NN` is a two-digit number starting at `01`.
- `{action}` is a short action word, hyphen-separated.
- Sort order is execution order.
- Do not use `step1`, `step02a`, or `step02-1`.

Recommended:

```text
step01-prepare.md
step02-collect.md
step03-review.md
step04-report.md
```

Not recommended:

```text
step1.md
step02a-review.md
review-step.md
final.md
```

## 4. Standard Structure

```markdown
# Step 02: Execute Review

> Executor: Agent / SubAgent / script / external tool
> Input: `state/config.json`, `step01-prepare/file-index.txt`
> Output: `step02-execute/findings.json`

## Goal

State what this step accomplishes.

## Input

- Input file or directory.
- The previous step's artifact.
- Run configuration.

## Actions

1. Concrete action one.
2. Concrete action two.
3. Concrete action three.

## Output

- Output file path.
- Output format.
- Required fields.

## Completion Criteria

- The file exists.
- The JSON is parseable.
- Every finding includes evidence.

## Failure Handling

- Stop when input is missing.
- On insufficient permission, record the path and continue with the accessible parts.
- Redact suspected secrets.

## Next Step

Read `workflow/step03-verify.md`.
```

## 5. Executor Types

```text
Agent
  The main Agent in the current conversation.
  Suits orchestration, judgment, reading configuration, and summarizing conclusions.

SubAgent
  An Agent in an isolated context.
  Suits long-material analysis, parallel review, and noise isolation.

Script
  An executable program.
  Suits deterministic operations, batch processing, format validation, and conversion.

External tool
  MCP, CLI, API, or a browser tool.
  Suits network queries, platform operations, and system-capability calls.
```

Selection rules:

- Strongly deterministic → use a script.
- Requires understanding and judgment → use an Agent or SubAgent.
- Requires isolating a lot of context → use a SubAgent.
- Requires live external data → use an external tool.

## 6. Input Spec

Inputs must state a path or source, not a session-dependent phrase like "the previous step's result."

Recommended:

```text
Input:
  - state/config.json
  - step01-prepare/file-index.txt
  - The user-specified target_dir
```

Not recommended:

```text
Input:
  - The previous analysis result
  - The file we just mentioned
```

## 7. Output Spec

Output paths must be stable.

```text
Step directory
  step{NN}-{action}/

Final output
  output/

State
  state/progress.json

No output
  -
```

Output content should be machine-verifiable:

```json
{
  "status": "completed",
  "items_checked": 12,
  "blocking_issues": [],
  "notes": []
}
```

## 8. Verification Checkpoints

Every step has at least one verification checkpoint.

Example:

```text
2a  step02-execute/findings.json exists.
2b  findings.json is parseable.
2c  Every blocking issue includes file, evidence, fix.
2d  progress.json's current_step is updated.
```

Checkpoints must be concrete; do not write "high-quality result."

## 9. Parameter Collection

Parameters come in three kinds:

```text
Must ask
  Cannot execute reliably without it.

Inferable
  The Agent can infer from path, filename, or context.

Defaultable
  Read from config/default.json or a spec default.
```

Rules:

- Infer instead of asking when you can.
- Ask all questions at once.
- Explain the purpose of each question.
- Write answers into `state/config.json`.

## 10. Pre-Checks

Before executing, check:

- The input path exists.
- Required files are readable.
- The output directory is writable.
- Required commands exist.
- Required credentials are optional or configured.

On failure, do not keep guessing.

## 11. Post-Verification

After executing, check:

- Output files exist.
- Output format is correct.
- State files are updated.
- Errors are recorded.
- The next step can independently read the artifacts and continue.

## 12. Routing

When one Skill supports multiple input types, do a routing step first.

```text
Step 01: identify input type
  Markdown  → step02-markdown.md
  PDF       → step02-pdf.md
  Directory → step02-directory.md
```

Write the routing result into `state/config.json`:

```json
{
  "input_type": "markdown",
  "selected_path": "workflow/step02-markdown.md",
  "reason": "The target file extension is .md"
}
```

## 13. Parallelism and Synchronization

Parallelize only when tasks are independent.

Suits parallelism:

- Independent review of multiple files.
- Independent collection from multiple sources.
- Independent evaluation of multiple candidate options.

Does not suit parallelism:

- The next step depends on the previous step's artifact.
- Multiple executors would write the same file.
- They share one external resource with no lock.

Parallel tasks must have a sync point:

```text
step04-merge/
  inputs:
    step03-batch-a/result.json
    step03-batch-b/result.json
  output:
    step04-merge/merged.json
```

## 14. Skipping Steps

Skipping a step is allowed but must be recorded.

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

Do not skip silently.

## 15. Timeouts and Errors

A step file should state:

- The per-step timeout.
- Whether it is retryable.
- The maximum retry count.
- Whether failure blocks subsequent steps.
- Whether it can degrade.

Example:

```text
Network request failure:
  Retry twice.
  If it still fails, record skipped_checks.
  Do not fabricate a result.
```

## 16. Prohibitions

- Steps that depend on chat memory.
- Steps with no output path.
- Steps with no completion criteria.
- Multiple steps doing the same thing.
- Long background material inside a step.
- Continuing with non-verifiable actions after a failure.
- Parallel executors writing the same file.

## 17. Checklist

```text
Naming
  [ ] stepNN-{action}.md.
  [ ] Numbering starts at 01.
  [ ] action matches the output directory.

Structure
  [ ] Has a goal.
  [ ] Has input.
  [ ] Has actions.
  [ ] Has output.
  [ ] Has completion criteria.
  [ ] Has failure handling.

Execution
  [ ] Executor is explicit.
  [ ] Input paths are stable.
  [ ] Output is verifiable.
  [ ] State is updated.

Recovery
  [ ] Can recover from files after an interruption.
  [ ] Skipped steps record a reason.
  [ ] The next step needs no re-reading of the full session.
```
