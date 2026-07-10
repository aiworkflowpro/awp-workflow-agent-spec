# Context-Engineering Spec

> This document specifies how a Skill loads, filters, isolates, and compresses context, so it does not stuff private knowledge or large amounts of irrelevant material directly into the main Agent.

## 1. Responsibility

Context engineering solves three problems:

- Which materials the Agent needs to do the task well.
- When those materials should be read.
- In what form they enter the main flow after being read.

A public Skill should not hardcode the author's private knowledge. It should declare the material type, the reading method, and the output format, and let the user supply their own source.

## 2. Core Principles

### 2.1 The Entry Carries No Long Context

`SKILL.md` states only "which kind of context is needed" and "when to read it." Do not stuff long reference material into the entry.

### 2.2 Deterministic Rules Can Be Read Directly

Deterministic rules — format red lines, naming rules, output templates, field definitions — are suitable for the main Agent to read directly and follow.

### 2.3 Isolate Creative Material First

Material that needs filtering and judgment — competitor cases, past work, user feedback, industry material — is suitable to hand to a SubAgent to distill into execution instructions.

### 2.4 The Output Is an Execution Instruction

A context step's output is not a research report; it is operating instructions for later steps.

Wrong output:

```text
I read 12 articles and found they all emphasize problem awareness.
```

Correct output:

```text
Start this report directly with the user's pain point; do not write industry background. Use a "problem + result" title structure. Do not use generic slogans.
```

## 3. Four Loading Methods

```text
Direct read
  Suits deterministic rules.
  E.g. format specs, output templates, field definitions.

Fixed path
  Suits material where the user or config can provide an explicit file path.
  E.g. references/style.md, docs/policy.md.

Retrieval discovery
  Suits scenarios needing keyword-based discovery across a large corpus.
  E.g. past cases, a knowledge base, web material.

Hybrid mode
  Read a fixed analysis framework first, then retrieve cases, then analyze cases with the framework.
  Suits competitor analysis, content strategy, and expanding review rules.
```

## 4. Material Dimensions

A Skill can declare these dimensions and choose as needed:

```text
identity
  Identity, role, brand positioning.

style
  Language, structure, format, expression style.

audience
  Target readers, user personas, pain points.

rules
  Platform rules, format red lines, compliance requirements.

examples
  Positive examples, negative examples, expected output.

history
  Existing work, existing records, avoiding duplication.

reference
  External material, public documents, research material.

constraints
  Budget, time, tools, permissions, platform limits.
```

A public Skill should make these dimensions configurable inputs, not hardcode the author's own material.

## 5. Context Step

Loading long context should be its own step.

```text
workflow/
  step01-prepare.md
  step02-load-context.md
  step03-execute.md
```

`step02-load-context.md` should output:

```text
step02-load-context/
  context.md
  sources.json
  skipped-sources.json
```

Recommended structure for `context.md`:

```markdown
# Execution Context

## Load Status

- Loaded:
- Skipped:
- Failed:

## Constraints for This Task

## Must Follow

## Recommended Strategy

## Prohibitions

## How Later Steps Read It
```

## 6. sources.json

Context sources should be traceable.

```json
{
  "sources": [
    {
      "id": "style",
      "path": "references/style.md",
      "mode": "direct-read",
      "status": "loaded"
    }
  ],
  "skipped": [
    {
      "id": "history",
      "reason": "The user did not provide a history directory."
    }
  ]
}
```

## 7. Direct-Read Rules

Suits direct reading:

- Field definitions.
- Output templates.
- Platform format rules.
- Security red lines.
- Short documents the user explicitly provides.

Does not suit direct reading:

- Large numbers of cases.
- Multiple long articles.
- Material requiring synthesized judgment.
- Raw material that would pollute the main Agent's judgment.

## 8. SubAgent Distillation Rules

A SubAgent suits:

- Reading large amounts of material.
- Compressing context.
- Extracting the task-relevant parts.
- Generating executable instructions for the next step.

SubAgent output must be short, concrete, and executable.

Recommended output:

```text
Instructions for this run
  1. Use only public material; do not reference internal paths.
  2. List blocking issues first, then suggestions.
  3. Every blocking issue must include a file path and evidence.

Prohibited
  - Do not restate all the material.
  - Do not output unverifiable judgments.
```

## 9. Context Budget

Default principles:

- The entry file is under 500 lines.
- A single step file should be under 300 lines where possible.
- Long reference material is read on demand.
- SubAgent output stays within about 2000 words where possible.
- Do not sacrifice execution clarity for "completeness."

Budget is not only a token issue; it is an attention issue. Even with a large context window, low-relevance material lowers execution stability.

## 10. Configuration Method

Declare context sources in `config/default.json` or the runtime configuration:

```json
{
  "context": {
    "style_path": "references/style.md",
    "examples_dir": "examples/",
    "history_dir": null,
    "max_sources": 10
  }
}
```

Rules:

- Put only paths, switches, limits, and defaults in the config.
- No real credentials.
- No private, non-distributable paths.
- Write the actual paths resolved for this run into `state/config.json`.

## 11. Public-Release Boundary

A public Skill may write:

```text
If the user provides references/style.md, read and follow it.
If not provided, use this Skill's built-in default template.
```

Do not write:

```text
Read some knowledge-base path on the author's machine.
Read the author's private brand files.
Read a customer's past cases.
```

## 12. Failure Handling

When context loading fails:

1. Decide whether it blocks the main task.
2. If blocking, stop and state the missing material.
3. If not blocking, record it in `skipped-sources.json`.
4. State the skipped checks or material in the later report.

Do not ignore silently.

## 13. Checklist

```text
Structure
  [ ] Long context has its own step.
  [ ] context.md outputs execution instructions.
  [ ] sources.json records sources.

Loading
  [ ] Deterministic rules are read directly.
  [ ] Creative material is isolated and distilled.
  [ ] Large material is not stuffed into the main Agent.

Public
  [ ] No hardcoded private paths.
  [ ] No hardcoded customer data.
  [ ] Does not require the reader to have the author's knowledge base.

Failure
  [ ] Missing material has a stop-or-degrade strategy.
  [ ] Skipped material records a reason.
```
