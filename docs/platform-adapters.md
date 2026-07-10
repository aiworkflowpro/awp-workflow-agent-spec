# Platform Adaptation Examples

> This document gives cross-platform adaptation examples. The examples are not a general standard; actual capability depends on the target Agent client.

## 1. Adaptation Principle

The same Workflow Agent Skill should first satisfy the general structure:

- `SKILL.md` states triggering and boundaries.
- `workflow/` states the steps.
- `config/runtime.json` declares the runtime.
- `docs/setup.md` states the preparation and verification method.
- `evals/` provides eval samples.

Then add an adaptation layer per client capability.

## 2. Claude Code Example

Suitable to use:

```text
CLAUDE.md
  Project-level navigation and maintenance rules.

SKILL.md
  The Skill entry.

SubAgent
  For isolating long-material analysis or parallel review.
```

Fallback:

```text
If SubAgent is unavailable, the main Agent processes batches serially and records batch status in progress.json.
If hooks are unavailable, turn hook behavior into manual verification commands.
```

## 3. Codex Example

Suitable to use:

```text
AGENTS.md
  The project-level rules entry.

SKILL.md
  The Skill or spec entry.

scripts/
  Handles deterministic checks.
```

Fallback:

```text
If the client has no automatic Skill discovery, hand the README or SKILL.md path directly to the Agent.
If there is no SubAgent, keep the workflow steps serial.
```

## 4. Cursor / OpenCode / Gemini CLI Example

Suitable to use:

```text
README.md
  The entry for humans and general Agents.

Project rules
  Hold only long-term project constraints; do not replace the Skill.

workflow/
  Read by the Agent on demand.
```

Fallback:

```text
If the client does not recognize SKILL.md, read SKILL.md as a plain Markdown spec.
If the client does not support tool pre-authorization, the setup doc lists the commands the user should confirm manually.
```

## 5. General Adaptation-Record Format

```text
Feature: SubAgent parallel review
General capability: isolated-context analysis of multiple batches
Platform enhancement: clients that support SubAgents can execute in parallel
Fallback path: the main Agent processes batches serially
Result impact: slower, but the output structure is unchanged
```

```text
Feature: hooks automatic check
General capability: run verification commands before release
Platform enhancement: clients that support hooks can trigger automatically
Fallback path: the manual commands listed in README and docs/testing.md
Result impact: verification commands must be run manually
```

## 6. Prohibitions

- Do not write "all clients support a certain field."
- Do not write a private field of Claude Code, Codex, or another client as a required item.
- Do not require the reader to install the author's private CLI.
- Do not treat a project-rule file as the Skill's trigger mechanism.
- Do not put platform examples into the minimal template.

## 7. Checklist

```text
[ ] General capability and platform enhancement are separated.
[ ] Every enhancement has a fallback path.
[ ] The README does not over-promise platform compatibility.
[ ] No example writes a single client as the only entry.
```
