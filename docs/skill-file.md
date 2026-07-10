# SKILL.md Entry Spec

> This document specifies the naming, frontmatter, body structure, directory boundaries, and public-distribution requirements of `SKILL.md`.

## 1. Responsibility

`SKILL.md` is the Skill's entry contract. Its job is not to explain all the background, but to let the Agent quickly decide:

- Whether to use this Skill.
- How to start executing.
- Which follow-up files to read.
- What is not allowed.
- What to produce on success.

The entry file should be short, precise, and executable. Push complex details into `workflow/`, `references/`, `docs/`, and `scripts/`.

## 2. Minimal Structure

```markdown
---
name: example-task-reviewing
description: Use when the user asks to review the scope, safety, completeness, and release readiness of an existing task artifact. This Skill is not for creating artifacts from scratch.
license: MIT
compatibility: Designed for cross-platform Workflow Agent Skill clients.
---

# Example Task Review

## Scope

## Input

## Workflow

## Files

## Tools

## Output

## Verification

## Failure Handling
```

## 3. Frontmatter

### 3.1 Common Fields

```text
name
  Required. Stable, short, lowercase, hyphen-separated.

description
  Required. The core of trigger judgment. State when to use, what it does, and when not to use.

license
  Optional. Recommended for public repositories.

compatibility
  Optional. States the runtime environment or client compatibility scope.

metadata
  Optional. Extra key-value metadata.
```

### 3.2 Platform Extension Fields

Different Agent clients may support extra fields such as `when_to_use`, `allowed-tools`, `hooks`, `model`, and `effort`. A public spec should write these as platform extensions, not as a general standard.

Writing principles:

- Put common fields first.
- Platform extensions must state the applicable client.
- Do not rely on extension fields for the core trigger.
- Do not put unverified fields into the template.

## 4. Naming `name`

Recommended format:

```text
{owner}-{domain}-{target}-{action}
```

Notes:

- `owner`: the maintainer, organization, or project identifier.
- `domain`: the domain, e.g. `content`, `dev`, `doc`, `skill`, `video`.
- `target`: the object, e.g. `markdown`, `code`, `pdf`, `github`.
- `action`: the action; prefer a stable gerund or an explicit verb.

Public examples may use:

```text
demo-markdown-release-check
example-task-reviewing
public-release-reviewing
```

Avoid:

```text
helper
assistant
toolkit
do-everything
```

## 5. Writing `description`

`description` decides whether the Agent selects the Skill. It matters more than the body.

A good `description` contains three things:

1. Task type.
2. Trigger scenario.
3. Exclusion scenario.

Recommended:

```yaml
description: Use when the user asks to review an existing Skill's trigger quality, workflow clarity, security boundaries, and release readiness. This Skill is not for creating a new Skill from scratch, and it does not perform publish actions.
```

Not recommended:

```yaml
description: A powerful Skill assistant.
```

Problems:

- No task boundary.
- No trigger scenario.
- No exclusion scenario.
- Prone to false triggers.

## 6. Body Structure

### 6.1 Scope

State what the Skill handles and what it does not.

```markdown
## Scope

This Skill handles:

- Pre-release checks for existing Markdown.
- Checks of links, image references, frontmatter, and code fences.

This Skill does not handle:

- Writing articles from scratch.
- Publishing to external platforms.
- Generating images.
```

### 6.2 Input

Inputs come in three kinds:

```text
Required
  Missing values block execution; ask or stop.

Optional
  Used if provided; runs fine without them.

Inferable
  The Agent can infer from file extension, directory structure, or context.
```

Principles:

- Infer instead of asking when you can.
- Ask only when you cannot reliably infer.
- Do not guess a missing required input.

### 6.3 Workflow

A single-step Skill can write the full flow in `SKILL.md`. Beyond three steps, split into `workflow/`.

```markdown
## Workflow

1. Confirm the target path exists.
2. Read the necessary files.
3. Check for blocking issues.
4. Output a report.
5. If the user asks for fixes, modify only in-scope files.
6. Re-verify.
```

### 6.4 Files

Declare the files that will be read, written, modified, or must not be read.

```markdown
## Files

May read:

- The target file.
- The README in the same directory.
- Related configuration.

May create:

- A review report the user requested.

May modify:

- Only files the user explicitly specifies.

Must not read:

- Credential files.
- Private account files.
- Unrelated directories.
```

### 6.5 Tools

Declare the commands, scripts, network access, and credentials that will be used.

The tools section can be written like this:

```text
## Tools

May run read-only checks:

  find <target> -maxdepth 3 -type f
  rg -n "token|secret|password|cookie|api_key|/Users|~/Downloads" <target>

Do not run destructive commands.
```

### 6.6 Output

The output structure should be stable.

```markdown
## Output

Final reply order:

1. Release verdict.
2. Blocking issues.
3. Non-blocking issues.
4. Suggested fixes.
5. Verification performed.
6. Remaining risk.
```

### 6.7 Verification

Verification must be concrete.

```markdown
## Verification

- The target path exists.
- Every blocking issue has evidence.
- No real secret was printed in full.
- Modified files were re-read.
```

### 6.8 Failure Handling

Failure handling states stop conditions and recovery methods.

```markdown
## Failure Handling

If the target path does not exist, stop and ask the user for a valid path.

If a suspected secret is found, redact the value and report only the file path, key name, and risk.
```

## 7. Directory Structure

### 7.1 Minimal Skill

```text
my-skill/
  SKILL.md
```

Suits:

- Single-file checks.
- Small reviews.
- Simple format conversion.

### 7.2 Workflow Skill

```text
my-skill/
  SKILL.md
  workflow/
    step01-prepare.md
    step02-execute.md
    step03-verify.md
  runs/
```

Suits:

- Multi-step tasks.
- Long tasks.
- Resumable tasks.

### 7.3 Script-Backed Skill

```text
my-skill/
  SKILL.md
  workflow/
  scripts/
  config/
  references/
  examples/
  runs/
```

Suits:

- Batch processing.
- File scanning.
- Data collection.
- Report generation.

### 7.4 Production-Grade Skill

```text
my-skill/
  SKILL.md
  docs/
    setup.md
    user-guide.md
  workflow/
  scripts/
  config/
  references/
    templates/
    presets/
    definitions/
  evals/
  runs/
```

Suits:

- External distribution.
- Team reuse.
- Long-term maintenance.

## 8. Independence Levels

A public Skill should distinguish two kinds:

```text
personal
  Personal type. May bind to a personal knowledge base, directory habits, and account routing.
  Not suitable for direct public distribution.

portable
  Distributable type. Assumes the reader lacks the author's environment.
  Must be self-contained or declare dependencies explicitly.
```

Examples in a public repository default to a `portable` design.

## 9. Lifecycle

### 9.1 Create

Write a minimal `SKILL.md` first that solves only one clear task.

### 9.2 Verify

Run the main path with real input and record failure points.

### 9.3 Split

When the entry grows, steps multiply, or artifacts get complex, split out `workflow/`, `references/`, and `scripts/`.

### 9.4 Release

Remove internal dependencies; add a README, license, examples, security boundaries, and test notes.

### 9.5 Iterate

Change based on failure cases, not on a whim to rewrite everything.

### 9.6 Deprecate

When deprecating a Skill, state the replacement in the README or an archive note. Don't let a deprecated entry keep triggering the Agent by mistake.

## 10. Prohibitions

- Writing real secrets in `SKILL.md`.
- Keeping a long version history in `SKILL.md`.
- Piling every tutorial into `SKILL.md`.
- Using broad names and broad descriptions.
- Executing destructive operations by default.
- Public examples that depend on the author's local paths.
- Letting the Skill implicitly read private accounts or customer data.

## 11. Checklist

```text
Entry
  [ ] name is stable.
  [ ] description states triggering and exclusion.
  [ ] Scope is clear.
  [ ] Input is split into required, optional, and inferable.

Execution
  [ ] Workflow steps are explicit.
  [ ] File boundaries are explicit.
  [ ] Tool boundaries are explicit.
  [ ] Output structure is stable.

Security
  [ ] No real credentials.
  [ ] No private paths.
  [ ] No customer data.
  [ ] No destructive default actions.

Public
  [ ] Examples are reproducible.
  [ ] The README does not over-promise.
  [ ] Security and testing docs are in sync.
```
