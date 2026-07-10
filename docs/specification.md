# Workflow Agent Skill — Full Specification

> The public, cross-platform, full-lifecycle specification for workflow-style Agent Skills.
> Goal: let an Agent use this specification to create, review, upgrade, test, and publicly release reusable Skills.

This is not a prompt collection, not the internal operating manual of some private knowledge base, and not a private format spec for Claude Code, Codex, or any single client. It defines a publicly reusable engineering system for Workflow Agent Skills: how to write the entry point, how to break down steps, how to load context, how to isolate scripts, how to recover state, how to layer configuration, how to declare credentials, how to run tests, and how to audit before release.

## 0. Public-Edition Boundary

This repository is maintained by AI Workflow Pro. The public edition follows four boundaries:

- Keep the methodology, structure, templates, and reusable engineering rules.
- Remove real accounts, secrets, customer data, private paths, and paid content.
- Do not bind to a maintainer's local directories, private knowledge base, or non-public services.
- Treat platform features as compatibility notes, not as a standard that every Agent client is pretended to support.

So the public edition keeps technical identifiers such as `Agent`, `Skill`, `Workflow Agent Skill`, `SKILL.md`, `description`, `workflow/`, `runs/`, and `progress.json`. Specific client names like `Claude Code`, `Codex`, and `Cursor` appear only in compatibility examples, reference sources, or project-entry notes — not as general requirements.

## 1. Design Philosophy

### 1.1 A Skill Is a Workflow Capability Package

In this specification, "Skill" defaults to Workflow Agent Skill. A mature Skill answers at least these questions:

- When should it trigger?
- When should it not trigger?
- Which inputs does it need?
- Which inputs can be inferred, and which must be asked?
- Which context does it read?
- Which actions are judged by the Agent, and which are executed by scripts?
- Where does each step write its output?
- How does it recover after an interruption?
- How is the final artifact verified?
- How are the security boundaries audited before release?

An artifact that has only a prompt — with no inputs, outputs, state, or verification — is a prompt fragment, not a complete Skill.

### 1.2 Keep the Entry Short, Layer the Details

`SKILL.md` is the entry point, not an encyclopedia. The entry file carries only triggering, boundaries, an execution map, and the necessary constraints. Long specs, templates, examples, installation, troubleshooting, and platform differences belong in `docs/`, `workflow/`, `references/`, or `assets/`.

The longer the entry, the harder it is for the Agent to tell which rules actually matter.

### 1.3 Don't Make the Model Guess What a Script Can Do

Prefer scripts or tools for deterministic actions:

- File scanning.
- JSON validation.
- Format conversion.
- Batch renaming.
- Link checking.
- Secret scanning.
- Report template rendering.

The Agent handles understanding, judgment, trade-offs, explanation, and fix suggestions. Scripts handle stability, reproducibility, and testability.

### 1.4 State Must Be Persisted

Long tasks, batch tasks, and multi-step tasks must have a run directory. The run directory records configuration, progress, intermediate artifacts, final output, and recovery hints. That way, after a session ends, context is compressed, or a script fails, another Agent can take over.

### 1.5 Audit Public Releases Like a Software Package

A Skill influences Agent behavior — it may read or write files, access the network, run commands, or touch credentials. Before public release, review it like a small software package:

- Are there real secrets?
- Are there private paths?
- Are there destructive default behaviors?
- Does it depend on hidden services?
- Are there non-reproducible examples?
- Does it promise untested capabilities?

## 2. Organizing Framework

The public edition abstracts the internal full-lifecycle standard into 16 modules. The core modules are the foundation every complex Skill should understand; the supporting modules are read as the scenario requires.

```text
Core modules
  01  SKILL.md entry spec
  02  Workflow step spec
  03  Context-engineering spec
  04  Scripts and tools spec
  05  Platform compatibility spec
  06  Run-state spec
  07  Configuration-layering spec
  08  Prompts and SubAgent spec

Supporting modules
  09  Variables and placeholders spec
  10  Credential declaration spec
  11  Output templates spec
  12  Environment setup spec
  13  User onboarding spec
  14  Troubleshooting and recovery spec
  15  Testing and release spec
  16  Design patterns and anti-patterns
```

## 3. Document Routing

```text
docs/
  specification.md
    The overall spec. Covers design philosophy, the module system, quality tiers, and release gates.

  skill-file.md
    SKILL.md entry, frontmatter, directory structure, distribution boundaries, and lifecycle.

  workflow-steps.md
    Naming, executors, inputs and outputs, verification, and scheduling of workflow/step*.md.

  context-engineering.md
    Context layering, direct reading, retrieval, isolation, context budget, and output structure.

  scripts.md
    The scripts/ directory, script return values, dependencies, error handling, and tool-call boundaries.

  platforms.md
    Capability, frontmatter, and compatibility differences across Agent clients.

  run-state.md
    runs/, state/, output/, progress.json, resume_hint, and the recovery flow.

  configuration.md
    L1 runtime parameters, L2 default config, L3 preset data, and the Runtime Contract.

  prompts.md
    Agent prompts, SubAgent prompts, output formats, and failure handling.

  variables.md
    Variable naming, placeholders, source annotation, no variable arithmetic, and latest pointers.

  credentials.md
    Credential placeholders, environment variables, permission scopes, missing-credential behavior, and security isolation.

  output-templates.md
    The structure and variable placeholders of Markdown, JSON, and HTML output templates.

  setup.md
    Install location, dependency preparation, runtime checks, model assets, and environment repair.

  user-guide.md
    The structure of a getting-started guide for ordinary users.

  troubleshooting.md
    Eight-layer error classification, diagnosis order, retry strategy, and recovery checklists.

  testing.md
    Trigger, function, recovery, security, regression, cross-model, and pre-release testing.

  patterns.md
    Common design patterns, composition, and anti-patterns.

  security.md
    Public-release security boundaries.

  quality-rubric.md
    A 100-point score, tier verdicts, and blockers.

  glossary.md
    The Chinese/English term boundaries, stating which proper nouns are not translated.

  platform-adapters.md
    Conditional adaptation examples and fallback paths across Agent clients.

  release.md
    Versioning, release gates, migration, and deprecation rules.
```

## 4. Core Modules

### 4.1 `SKILL.md` Entry Spec

See [skill-file.md](skill-file.md).

The entry spec answers these questions:

- How to name the Skill.
- How to organize the directory.
- How to write frontmatter fields.
- How `description` avoids false triggers.
- Which content to keep in the `SKILL.md` body.
- Where to split long content.
- How to distinguish personal from distributable Skills.

A minimal viable entry must include:

```text
frontmatter
  name
  description
  license or compatibility note

body
  Scope
  Input
  Workflow
  File boundary
  Tool boundary
  Output
  Verification
  Failure handling
```

### 4.2 Step Spec

See [workflow-steps.md](workflow-steps.md).

Step files answer these questions:

- How to name step files.
- How each step declares its executor.
- How to write input and output files.
- How the main Agent, SubAgents, scripts, and external tools divide the work.
- How to do pre-checks and post-verification.
- How to handle branching, parallelism, sync points, and skipped steps.

A step file is a contract, not a description. Each step states at least:

- Why it exists.
- Where it reads input.
- Where it writes artifacts.
- How to stop or recover on failure.
- How to verify after completion.

### 4.3 Context-Engineering Spec

See [context-engineering.md](context-engineering.md).

The context spec answers these questions:

- Which materials the main Agent should read directly.
- Which materials are better distilled by a SubAgent or an isolated context.
- Which materials should be obtained via search or MCP.
- How to compress long materials into execution instructions.
- How to avoid hardcoding private knowledge into the Skill.

A public Skill should not hardcode private knowledge. The right way is to declare "which kind of material is needed" and let the user provide paths, URLs, environment variables, or a reference directory.

### 4.4 Scripts and Tools Spec

See [scripts.md](scripts.md).

The scripts spec answers these questions:

- Which actions suit a script.
- How a script declares dependencies.
- How a script reads input and writes output.
- How return values become machine-parseable.
- How errors are classified.
- How network requests set timeouts, retries, and fail closed.
- How to keep scripts from reading real credentials.

### 4.5 Platform Compatibility Spec

See [platforms.md](platforms.md).

The platform spec answers these questions:

- Which rules are general Agent Skill principles.
- Which fields apply only to specific clients.
- How Agent clients differ in tools, hooks, SubAgents, context loading, project-entry files, and install directories.
- How to write public notes that don't bind to one platform.

### 4.6 Run-State Spec

See [run-state.md](run-state.md).

The run spec answers these questions:

- How to name the `runs/` directory.
- How to generate the `keyword`.
- Which fields `state/progress.json` must have.
- How to organize `output/` and step directories.
- How to recover from `resume_hint` after an interruption.
- How a multi-mode Skill avoids artifact confusion.

### 4.7 Configuration-Layering Spec

See [configuration.md](configuration.md).

The configuration spec answers these questions:

- Where L1 runtime parameters are written.
- Where L2 default parameters are written.
- Where L3 preset options are written.
- How configuration priority merges.
- What must not be written into config files.
- How runtime environments, binaries, model assets, and caches are declared.

### 4.8 Prompts and SubAgent Spec

See [prompts.md](prompts.md).

The prompts spec answers these questions:

- Which parts a prompt template should contain.
- Why SubAgent output should be execution instructions, not a research report.
- When prompt files go into `references/prompts/`.
- How to write failure handling, output formats, and quality standards.
- How to keep the main Agent from reading too much raw material.

## 5. Supporting Modules

### 5.1 Variables and Placeholders

See [variables.md](variables.md).

The variable spec defines:

- Variable naming.
- Variable source.
- Default values.
- No variable arithmetic.
- Latest-pointer files.
- Path placeholder conventions.

### 5.2 Credential Declaration

See [credentials.md](credentials.md) and [security.md](security.md).

The credential spec defines:

- Environment variable names.
- Permission scopes.
- dry-run behavior.
- Stop conditions when a credential is missing.
- How to use placeholder values in public examples.

### 5.3 Output Templates

See [output-templates.md](output-templates.md).

The output-template spec defines:

- Markdown templates.
- JSON structure.
- HTML templates.
- Variable placeholders.
- Report section order.
- Machine-verifiable fields.

### 5.4 Environment Setup

See [setup.md](setup.md).

The environment spec defines:

- Install location.
- Dependency checks.
- Runtime preparation.
- Model assets.
- Optional network access.
- Error-repair commands.

### 5.5 User Onboarding

See [user-guide.md](user-guide.md).

The onboarding spec defines:

- What the Skill does.
- What the Skill does not do.
- Minimal input.
- Minimal run method.
- Common errors.
- Where the output is.

### 5.6 Troubleshooting and Recovery

See [troubleshooting.md](troubleshooting.md).

The troubleshooting spec defines:

- Entry-layer errors.
- Parameter-layer errors.
- Dependency-layer errors.
- Credential-layer errors.
- Network-layer errors.
- Path-layer errors.
- Progress-layer errors.
- Recovery order.

### 5.7 Testing and Release

See [testing.md](testing.md).

The testing spec defines:

- Trigger tests.
- Function tests.
- Recovery tests.
- Script tests.
- Security tests.
- Regression tests.
- Release gates.

### 5.8 Design Patterns and Anti-Patterns

See [patterns.md](patterns.md).

The patterns spec defines:

- Sequential orchestration.
- Multi-tool coordination.
- Iterative refinement.
- Context-aware routing.
- Domain-knowledge encapsulation.
- Visualized output.
- Scheduled execution.
- Chained composition.
- Common anti-patterns.

## 6. Quality Tiers

```text
Tier 0  Prompt fragment
  Only a prompt, with no inputs, outputs, state, or verification.
  Not recommended for public release.

Tier 1  Minimal Skill
  Has SKILL.md with name, description, scope, input, output, and verification.
  Suits short tasks, review tasks, and format-check tasks.

Tier 2  Workflow Skill
  Has workflow/ step files, each with input, output, executor, and completion criteria.
  Suits multi-step tasks and repeatable processes.

Tier 3  Script-backed Skill
  Has scripts/; deterministic tasks run in scripts, and scripts run standalone.
  Suits collection, conversion, batch processing, validation, and report generation.

Tier 4  Production-grade Skill
  Has runs/ state, progress.json, recovery, test cases, security boundaries, and a pre-release audit.
  Suits long tasks, batch tasks, cross-tool tasks, and public distribution.
```

The minimum for public release is Tier 1. Multi-step or batch tasks should reach Tier 2. When there are side effects — scripts, network, credentials, deletion, upload, publishing — audit at Tier 4.

## 7. Scenario-Based Reading

```text
Writing only a minimal Skill
  Must read: skill-file.md
  As needed: testing.md

Building a multi-step workflow
  Must read: skill-file.md, workflow-steps.md, run-state.md
  As needed: configuration.md, testing.md

Adding scripts
  Must read: scripts.md, run-state.md, configuration.md
  As needed: setup.md, troubleshooting.md

Needing external material or a knowledge base
  Must read: context-engineering.md, variables.md
  As needed: prompts.md, credentials.md

Releasing publicly
  Must read: security.md, testing.md, user-guide.md
  As needed: platforms.md, troubleshooting.md

Building a complex system or toolkit
  Must read: patterns.md, configuration.md, platforms.md
  As needed: output-templates.md, setup.md
```

## 8. Development Flow

### 8.1 Define Boundaries First

Write down:

- What task the Skill handles.
- What task it does not handle.
- How the user should phrase the request.
- Which inputs must be provided.
- Which capabilities must not be promised publicly.

If the boundaries aren't clear, don't move on to script and template design.

### 8.2 Then Design the Entry

`description` is the core of trigger quality. It must contain all of:

- The task type.
- The trigger scenario.
- The exclusion scenario.

Example:

```yaml
description: Use when the user asks to review an existing Skill's trigger quality, workflow clarity, security boundaries, and release readiness. This Skill is not for creating a new Skill from scratch.
```

### 8.3 Then Break Down the Workflow

A process with more than three steps should be split into `workflow/`. Each step gets its own file; do not cram the whole process into `SKILL.md`.

A step file must have:

- Goal.
- Input.
- Actions.
- Output.
- Completion criteria.
- Failure handling.

### 8.4 Then Add State

Establish a run directory at the start of a long task, rather than adding one after it grows complex:

```text
runs/{keyword}-YYYYMMDD-HHMMSS/
  state/
    config.json
    progress.json
  step01-prepare/
  step02-execute/
  output/
```

### 8.5 Then Separate Scripts

Write repeatable, deterministic, testable actions as scripts. Scripts output JSON or file paths, so the Agent doesn't parse results from a wall of terminal logs.

### 8.6 Finally, Test and Do a Public Audit

Before release, complete at least:

- A positive trigger test.
- A negative trigger test.
- A main-path function test.
- A recovery test.
- A secret scan.
- A README-versus-actual-behavior consistency check.

## 9. Release Gates

Before public release, all of the following must pass:

```text
Structure
  [ ] SKILL.md exists.
  [ ] name and description are clear.
  [ ] README states purpose and boundaries.
  [ ] Multi-step tasks have workflow/.
  [ ] Long tasks have runs/ and progress.json.
  [ ] config/runtime.json exists when an environment, binary, model, or cache is required.
  [ ] The Runtime Contract declares needs only, not environment artifacts.

Security
  [ ] No real secrets.
  [ ] No private paths.
  [ ] No customer data.
  [ ] No private service URLs.
  [ ] No destructive default actions.
  [ ] Network behavior is documented.

Testing
  [ ] Trigger eval includes positive and negative cases.
  [ ] Function, recovery, security, and runtime evals have samples.
  [ ] The main path runs end to end.
  [ ] Errors and interruptions can be recovered.
  [ ] Scripts run standalone.
  [ ] The README promises no untested capability.

Quality
  [ ] A score has been given per docs/quality-rubric.md.
  [ ] There are zero blocked items.
  [ ] Terminology matches docs/glossary.md.
  [ ] The release record matches docs/release.md.
```

## 10. Anti-Patterns

### 10.1 Do-Everything Assistant

Named `assistant`, `helper`, or `toolkit`, with fuzzy boundaries. The Agent can't tell when to use it.

### 10.2 Oversized Entry File

Cramming every tutorial, background note, template, and troubleshooting tip into `SKILL.md`. This pollutes context and lowers execution stability.

### 10.3 Prompt Only

No inputs, outputs, verification, or state. It may be useful, but it can't pass as a production-grade Skill.

### 10.4 Hidden Private Dependencies

The example runs on the author's machine but breaks once it leaves the private path, account, knowledge base, or internal service.

### 10.5 Side Effects by Default

Deleting, overwriting, uploading, publishing, paying, sending mail, or modifying remote systems by default. A public Skill must default to read-only or dry-run.

### 10.6 Testing by Vibes

Using "looks good" or "reads professionally" as acceptance criteria. A public Skill needs verifiable assertions.

## 11. Relationship to the Internal Standard

The internal standard serves the maintainer's personal production environment and may bind to a personal knowledge base, runtime library, account routing, and local tools. The public standard serves open-source readers, who by default have none of these resources.

So the public edition follows the principle of "same structure, de-identified dependencies":

- Keep the 16-module framework.
- Keep the full-lifecycle constraints.
- Keep the automation, recovery, testing, and audit ideas.
- Remove private directories, private credentials, private tools, real business prompts, and customer data.
- Rewrite internal runtime-library capabilities as general conventions or optional implementations.

## 12. Minimal Runnable Example

A minimal Skill can be just:

```text
my-skill/
  SKILL.md
```

`SKILL.md`:

```markdown
---
name: markdown-release-check
description: Use when the user asks to check whether an existing Markdown file is ready to publish. This Skill is not for writing new articles, uploading content, or generating images.
license: MIT
---

# Markdown Pre-Release Check

## Scope

Check whether an existing Markdown file is ready to publish.

## Input

- Path to a Markdown file.

## Workflow

1. Confirm the file exists.
2. Read the file.
3. Check headings, frontmatter, links, image references, code fences, and empty sections.
4. Report blocking issues first.
5. When the user asks for fixes, patch only the target file.
6. Re-read and verify.

## Output

- Blocking issues.
- Non-blocking suggestions.
- Applied fixes, if any.
- Remaining risk.

## Verification

- Every finding references the target file path.
- Code fences are paired and closed.
- No publish action was performed.
```

## 13. Production-Grade Example Structure

```text
public-release-reviewing/
  SKILL.md
  docs/
    setup.md
    user-guide.md
  workflow/
    step01-prepare.md
    step02-scan.md
    step03-review.md
    step04-report.md
  scripts/
    scan-secrets.sh
    check-links.sh
  config/
    default.json
  references/
    templates/
      public-release-report.md
  evals/
    trigger-queries.json
  runs/
```

This structure suits long processes such as public-release review, source-package review, and pre-publish content review.

## 14. Maintenance Principles

- Keep the entry short.
- Split the spec by module.
- Keep examples runnable.
- Write security boundaries separately.
- Write test methods separately.
- Claim no more publicly than what has been verified.
- Leave version history to git; don't write it into `SKILL.md`.

## 15. Conclusion

A good Skill should let a future Agent understand it, run it stably, resume when interrupted, diagnose when it errors, and audit it before release. Achieve those, and the Skill graduates from a prompt into a reusable workflow capability package.
