# Workflow Agent Skill Specification

> The main entry point for AI Agents and maintainers.
> This file is the source of truth for the repository's project context. `AGENTS.md` is another client entry point; it points back to this file so there is only one set of instructions to maintain.

## Subdirectory Index

| Directory | Contents | CLAUDE.md |
|-----------|----------|:---------:|
| [docs/](docs/) | Directory index | — |
| [evals/](evals/) | Directory index | — |
| [examples/](examples/) | Directory index | — |
| [schemas/](schemas/) | Directory index | — |
| [scripts/](scripts/) | Directory index | — |
| [templates/](templates/) | Directory index | — |


## File Index

| File | Description |
|------|-------------|
| [AGENTS.md](AGENTS.md) | Top-level file |
| [CHANGELOG.md](CHANGELOG.md) | Top-level file |
| [CONTRIBUTING.md](CONTRIBUTING.md) | Top-level file |
| [LICENSE](LICENSE) | Top-level file |
| [README.md](README.md) | Top-level file |


## Project Purpose

This repository defines a Workflow Agent Skill Specification intended for public release.

It helps maintainers turn repeatable AI Agent workflows into step-based, resumable, verifiable, auditable Workflow Agent Skills, and ensures those Skills:

- Have clear trigger boundaries.
- Have an entry point short enough for an Agent to load.
- Can be distributed publicly and safely.
- Can be tested with real examples.
- Can resume after an interruption.
- Can be audited before release.

This repository contains no private workflows, credentials, account routing, customer data, or maintainer-only local paths.

## Sources of Truth

```text
CLAUDE.md
  Source of truth for the Agent's project context.

AGENTS.md
  Entry point for AGENTS.md-aware Agent clients. Points to this file only; does not duplicate project facts.

README.md
  Entry point for public readers.

docs/specification.md
  Full specification: the 16-module framework, quality tiers, scenario-based reading, and release gates.

docs/skill-file.md
  SKILL.md entry, frontmatter, directory structure, distribution boundaries, and lifecycle.

docs/workflow-steps.md
  Workflow step naming, executors, inputs and outputs, verification, and scheduling.

docs/context-engineering.md
  Context layering, direct reading, retrieval, isolation, and output structure.

docs/scripts.md
  scripts/, CLI, MCP, network API, and external-tool boundaries.

docs/platforms.md
  Agent-client capability differences and compatible writing.

docs/run-state.md
  runs/, state/, output/, progress.json, and the recovery flow.

docs/configuration.md
  L1/L2/L3 configuration layering, defaults, preset data, and runtime declarations.

docs/prompts.md
  Agent prompts, SubAgent prompts, output formats, and failure handling.

docs/variables.md
  Variables, placeholders, source annotation, and latest pointers.

docs/credentials.md
  Credential placeholders, environment variables, permission scopes, and missing-credential behavior.

docs/output-templates.md
  Markdown, JSON, and HTML output templates and variable placeholders.

docs/setup.md
  Install location, dependency preparation, runtime checks, and environment repair.

docs/user-guide.md
  The structure of a getting-started guide for users.

docs/troubleshooting.md
  Error classification, diagnosis order, retry strategy, and recovery checks.

docs/security.md
  Public-release security boundaries.

docs/testing.md
  Trigger, function, recovery, security, regression, cross-model, and release testing.

docs/patterns.md
  Design patterns, composition, and anti-patterns.

docs/quality-rubric.md
  A 100-point score, tier verdicts, and blockers.

docs/glossary.md
  Glossary. Defines the boundaries of terms like Agent, Skill, Workflow Agent Skill, SubAgent, and runtime.

docs/platform-adapters.md
  Conditional adaptation examples for platforms such as Claude Code, Codex, Cursor, OpenCode, and Gemini CLI.

docs/release.md
  Versioning, release gates, changelog, migration, and deprecation rules.

schemas/
  JSON Schemas for Skill metadata, progress, runtime, and trigger eval.

templates/
  Reusable templates.

scripts/
  A dependency-free specification validator.

examples/
  A minimal Skill, a multi-step workflow Skill, and a golden example.

evals/
  Trigger, function, recovery, security, and runtime eval cases.

CHANGELOG.md
  Version changelog.
```

## Editing Rules

- Public safety first: no real credentials, private paths, customer data, private URLs, or paid content.
- Do not write maintainer-only absolute local paths.
- Do not add new destructive-by-default scripts; if truly necessary, they must default to `dry-run` and state the approval boundary clearly.
- Keep `README.md` as a short entry point; put long rules in `docs/`.
- Keep the `SKILL.md` in examples short enough for an Agent to load.
- Prefer verifiable examples over vague concepts.
- When changing a template, update or add the corresponding example.
- When changing release behavior, update `docs/security.md` or `docs/testing.md`.
- When changing the repository structure, update `README.md` and this file.

## Quality Standard

A high-quality Skill should let a future Agent:

1. Decide when to trigger.
2. Decide when not to trigger.
3. Parse required inputs.
4. Execute a documented workflow.
5. Persist state during long tasks.
6. Resume after an interruption.
7. Verify outputs.
8. Audit security boundaries before release.

If a workflow cannot be tested, resumed, or audited, it is not yet a production-grade Skill.

## Validation Commands

Run from the repository root:

```bash
python3.12 scripts/validate-spec.py --root .
gitleaks detect --no-git --source . --redact --no-banner
find . -path ./.git -prune -o -name "*.json" -print0 | xargs -0 -n1 python3.12 -m json.tool >/dev/null
```

Broad security search:

```bash
rg -n "token|secret|password|cookie|api_key|apikey|client|customer|/Users|~/Downloads|private" .
```

This search deliberately hits the example commands in the docs. Review each hit manually before release.

## Agent Reading Order

When maintaining the repository:

1. Read `CLAUDE.md`.
2. Read `README.md`.
3. Read specific files under `docs/`, `templates/`, `examples/`, or `evals/` as the task requires.

When working on a Skill design question:

1. Read `README.md`.
2. Read `docs/specification.md`.
3. Compare against `templates/SKILL.md` and `examples/`.

When doing a release check:

1. Read `docs/security.md`.
2. Read `docs/testing.md`.
3. Run the validation commands.

## Public Release Boundary

This repository may be released publicly only if:

- The secret scan has no unresolved findings.
- Example data is generic placeholder data.
- It does not depend on private local paths.
- The README promises no untested capability.
- The `CLAUDE.md` and `AGENTS.md` needed to maintain this repository exist at the root, and their responsibilities do not conflict.
- The structure described in the templates, examples, and README is consistent.

## Maintainer

AI Workflow Pro

- Website: https://aiworkflowpro.com
- GitHub: https://github.com/aiworkflowpro
