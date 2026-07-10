# Prompts and SubAgent Spec

> This document specifies how prompt templates, SubAgent instructions, output formats, and failure handling are written in a Skill.

## 1. Responsibility

Prompts define the judgment tasks of an Agent or SubAgent. They should not carry deterministic work, and should not replace scripts, configuration, and state files.

Suits prompts:

- Analysis.
- Review.
- Summarization.
- Generating suggestions.
- Judging severity.
- Choosing an execution path.

Does not suit prompts:

- Batch file scanning.
- JSON parsing.
- Link checking.
- Renaming.
- Large-scale format conversion.

## 2. Placement

Short prompts can live in a step file. Long prompts go in:

```text
references/prompts/
  prompt-review.md
  prompt-summarize.md
```

The step file references only the path:

```text
Use references/prompts/prompt-review.md.
Input is step02-scan/findings.json.
Write output to step03-review/review.json.
```

## 3. Basic Prompt Structure

```markdown
# Task

What you need to accomplish.

## Input

Which files to read.

## Judgment Criteria

Which rules to judge by.

## Output Format

Which file to write, and its structure.

## Prohibitions

What must not be done.

## Failure Handling

What to do on missing input, malformed format, or insufficient evidence.
```

## 4. SubAgent Output Principle

SubAgent output should be execution instructions the next step can use directly.

Recommended:

```json
{
  "decision": "blocked",
  "blocking_issues": [
    {
      "file": "config/example.json",
      "evidence": "Contains a field that looks like a real token",
      "fix": "Replace with an environment-variable placeholder"
    }
  ],
  "next_actions": [
    "Modify the example config",
    "Add the environment-variable note to docs/setup.md"
  ]
}
```

Not recommended:

```text
The project is decent overall, but there are still some areas to improve.
```

## 5. Output Format

Structure it when you can.

Suits JSON:

- Review results.
- Scores.
- Issue lists.
- Batch results.
- Status information.

Suits Markdown:

- Human-facing reports.
- Final notes.
- Tutorials.
- Long documents.

If both are needed, write JSON first, then render Markdown from it via the Agent or a script.

## 6. Evidence Requirements

Review prompts must require evidence:

- File path.
- Line number or snippet.
- Impact statement.
- Fix suggestion.

An issue with no evidence should be marked "to be confirmed," not written as a firm conclusion.

## 7. Failure Handling

Prompts must define failure behavior:

```text
If the input file does not exist:
  Stop and return missing_input.

If evidence is insufficient:
  Mark needs_manual_review.

If a suspected secret is found:
  Redact the value and report only the key name and path.
```

## 8. Iterative Prompts

When multiple rounds of improvement are needed, there must be an exit condition.

```text
Iterate at most 3 rounds.
Each round writes feedback_round_N.md.
Stop once all checks pass.
If it still fails, output remaining_risk.
```

Do not loop forever.

## 9. Main Agent vs. SubAgent Division

```text
Main Agent
  - Read SKILL.md.
  - Orchestrate steps.
  - Manage the run directory.
  - Merge results.
  - Produce the final reply.

SubAgent
  - Read the specified material.
  - Make a local judgment.
  - Output a short result.
  - Do not modify files outside the main flow.
```

A SubAgent should not receive unnecessary global context.

## 10. Prohibitions

- Prompts with no output format.
- Prompts that ask to "do your best" with no acceptance criteria.
- Prompts that let a SubAgent read unrelated directories.
- Prompts that ask for a long background analysis.
- Prompts that print full secrets.
- Prompts that bypass user authorization to run destructive actions.
- Prompts that hardcode private knowledge.

## 11. Checklist

```text
Structure
  [ ] Has a task.
  [ ] Has input.
  [ ] Has judgment criteria.
  [ ] Has an output format.
  [ ] Has failure handling.

Evidence
  [ ] Review conclusions have a path and evidence.
  [ ] Insufficient evidence is marked uncertain.

Division
  [ ] Deterministic tasks are not handed to prompts.
  [ ] The SubAgent reads only the necessary material.
  [ ] Output can be used directly by the next step.
```
