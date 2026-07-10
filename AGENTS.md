# Workflow Agent Skill Specification

This file is the entry point for AGENTS.md-aware Agent clients.

After entering this repository, read `CLAUDE.md` first. `CLAUDE.md` is the source of truth for this repository's project context.

## Required Context

- Project purpose: the public Workflow Agent Skill Specification.
- Core use: let an Agent use this repository to create, review, and upgrade cross-platform workflow-style Skills.
- Public safety boundary: no secrets, private paths, customer data, private URLs, or maintainer-only dependencies.
- Public reader entry: `README.md`.
- Full specification: `docs/specification.md`.
- Core modules: `docs/skill-file.md`, `docs/workflow-steps.md`, `docs/context-engineering.md`, `docs/scripts.md`, `docs/run-state.md`, `docs/configuration.md`.
- Supporting modules: `docs/variables.md`, `docs/credentials.md`, `docs/output-templates.md`, `docs/setup.md`, `docs/user-guide.md`, `docs/troubleshooting.md`, `docs/patterns.md`.
- Governance modules: `docs/quality-rubric.md`, `docs/glossary.md`, `docs/platform-adapters.md`, `docs/release.md`.
- Security rules: `docs/security.md`.
- Testing rules: `docs/testing.md`.
- Machine contracts: `schemas/`.
- Local validation: `scripts/validate-spec.py`.
- Reusable templates: `templates/`.
- Examples: `examples/`.
- Eval cases: `evals/`.

## Maintenance Rules

- Treat `CLAUDE.md` as the source of truth.
- Do not duplicate long project descriptions in this file.
- When the repository structure changes, update `CLAUDE.md` and `README.md` in sync.
- When release behavior changes, update `docs/security.md` or `docs/testing.md` in sync.
- When a template changes, update or add the corresponding example.
- Before release, run the validation commands listed in `CLAUDE.md`.
