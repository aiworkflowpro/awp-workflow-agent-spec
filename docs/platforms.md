# Cross-Platform Compatibility Spec

> This document specifies how a Workflow Agent Skill handles differences across Agent clients, tool systems, frontmatter fields, project entries, and runtime capabilities.

## 1. Responsibility

The platform spec addresses:

- Which rules are general principles.
- Which fields are platform extensions.
- How to avoid treating a single platform's capability as a general standard.
- How to write public notes compatible with multiple Agent clients.

## 2. General Spec Layer

Most Workflow Agent Skills should follow these:

- A clear entry file.
- A `description` that states triggering and exclusion.
- Long content read on demand.
- `workflow/` steps with input, output, executor, and completion criteria.
- Recoverable state.
- No hardcoded credentials.
- Auditable script and tool boundaries.
- Testable triggering, function, recovery, and security.
- Reproducible public examples.

These are general engineering principles that do not depend on a specific client.

## 3. Platform Adaptation Layer

Different clients may support:

- Extra frontmatter fields.
- Pre-approved tools.
- hooks.
- SubAgents or isolated contexts.
- Plugin packaging.
- Upload packages.
- Project-rule files.
- Browser or system tools.

A public spec should write these as conditional capabilities:

```text
If the client supports SubAgents or isolated contexts, use them to isolate long-material analysis.
If not, the main Agent reads and compresses context in stages.
```

Do not write:

```text
All Skills must use a certain platform's SubAgent capability.
```

## 4. Frontmatter Compatibility

Keep common fields few and stable:

```text
name
description
license
compatibility
metadata
```

Platform extension fields should be documented separately:

```text
allowed-tools
  Some clients use this to pre-approve tools, declare tools, or hint capabilities; meaning per the client's docs.

hooks
  Some clients support lifecycle hooks.

model
  Some clients support a model override.

effort
  Some clients support a reasoning-effort override.
```

Rules:

- Core capability does not depend on extension fields.
- Extension fields have a fallback.
- Docs state the applicable platform.
- Do not put extension fields into the minimal template until the target client's support is confirmed.

## 5. Tool Compatibility

Tools come in three kinds:

```text
Local commands
  find, rg, python, node, gitleaks, etc.

Client tools
  File, terminal, browser, SubAgent, and MCP tools provided by the Agent client.

External services
  GitHub, CMS, search, database, API.
```

A public Skill should declare:

- Required tools.
- Optional tools.
- The fallback path when a tool is missing.
- Whether a tool reads/writes files or accesses the network.

## 6. Client Examples Are Not the Standard Itself

A public spec may be compatible with Claude Code, Codex, Cursor, OpenCode, Gemini CLI, or other Agent clients, but do not write any one client's private capability as a general requirement.

Recommended:

```text
Some clients may support extra frontmatter fields, hooks, or a model override.
Some clients may read context via AGENTS.md, CLAUDE.md, a project-rules directory, or a local skills directory.
Rely on the current client's actual support.
```

Prohibited:

```text
All clients read CLAUDE.md.
All clients read AGENTS.md.
All clients support hooks, allowed-tools, or SubAgents.
```

## 7. Project-Rule Files

Common optional entries:

```text
README.md
  For human readers.

AGENTS.md
  For AGENTS.md-aware Agent clients.

CLAUDE.md
  For CLAUDE.md-aware Agent clients.

Other rule files or directories
  For the matching client, e.g. project rules, plugin config, or a local skills directory.

SKILL.md
  The Skill entry.
```

Responsibility boundaries:

- Long-term project rules go in the project-context entry the client supports.
- Task-type capabilities go in `SKILL.md`.
- Reader notes go in `README.md`.

Do not cram all project rules into the Skill.

## 8. Recording Platform Differences

If a feature depends on a platform capability, state:

```text
Feature: SubAgent parallel review
Depends on: the client supports SubAgents, parallel tasks, or isolated contexts
Fallback: the main Agent processes each batch serially
Risk: slower, but the result structure is unchanged
```

## 9. Public Distribution

When distributing publicly:

- Do not assume the reader uses the same client.
- Do not assume the reader has the same tools.
- Do not assume the reader has the same model.
- Do not assume the reader has the author's accounts.
- Do not assume the reader has the author's directory structure.

Turn all of these into explicit dependencies or optional capabilities.

## 10. Prohibitions

- Claiming that all platforms support the same field.
- Depending on hooks with no fallback.
- Accessing the network with no notice.
- Calling an external API with no notice.
- Requiring a paid tool with no notice.
- Treating project rules as the Skill's trigger rules.

## 11. Checklist

```text
General
  [ ] Core capability does not depend on a single client's extension.
  [ ] Extension fields are marked with the applicable platform.
  [ ] Required and optional tools are written separately.

Fallback
  [ ] There is a serial plan when SubAgents or isolated contexts are unavailable.
  [ ] There is a stop-or-degrade note when an external tool is unavailable.
  [ ] No fabricated results when the network is unavailable.

Public
  [ ] Does not require the author's local environment.
  [ ] Does not require the author's private accounts.
  [ ] The README does not over-promise compatibility.
```
