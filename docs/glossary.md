# Glossary

> This document fixes the meaning of the specification's proper nouns. Keep these terms consistent; do not rename them into ambiguous synonyms.

## 1. Canonical Terms

```text
Agent
  The large-model agent that executes tasks.

Skill
  A capability package the Agent can read and execute. As a file or spec name, always "Skill".

Workflow Agent Skill
  A workflow-style Agent Skill. The core object of this specification.

SubAgent
  An Agent execution unit that runs in an isolated context or in parallel.

runtime
  The set of environment, binaries, models, cache, and network capabilities a Skill needs to run.

Runtime Contract
  The runtime-needs contract declared in `config/runtime.json`.

doctor
  Diagnoses runtime or environment only; does not modify the machine or download models.

prepare
  Prepares runtime, dependencies, or model assets after the user explicitly runs it.

repair
  Repairs a broken runtime, a checksum mismatch, or a polluted cache.

resolve
  Returns the actual paths of env, binary, model, and cache so scripts avoid hardcoding.

frontmatter
  The YAML metadata at the top of `SKILL.md`.

description
  The core field by which the Agent decides whether to trigger a Skill.

workflow/
  The directory that holds multi-step execution instructions.

runs/
  The directory that holds run state, intermediate artifacts, and final output.

progress.json
  The state file recording execution status, resume hints, errors, and artifact paths.

dry-run
  Generates a plan or simulates actions only; performs no real side effects.
```

## 2. Standard Terms

```text
trigger
  When the Agent decides to use a Skill.

setup
  Environment setup.

configuration
  Configuration.

credential
  A credential.

output template
  An output template.

release gate
  A pre-release gate.

quality rubric
  The quality scoring rubric.

fallback
  A degradation path.

checkpoint
  A verification checkpoint.
```

## 3. Naming Rules

- File names, directory names, frontmatter fields, and JSON fields stay in English.
- Technical identifiers are not renamed, even inside prose.
- Platform names stay as-is, e.g. Claude Code, Codex, Cursor, OpenCode, Gemini CLI.
- Do not write platform names as a general requirement.
- Do not put an internal brand, course, or private tool into the general spec body.

## 4. Prohibited Usages

```text
Inventing an alternative name for SubAgent
  Do not. Use SubAgent.

Inventing an alternative name for Agent Skill
  Do not. Use Agent Skill or Workflow Agent Skill.

Requiring the runtime directory to be a specific private path
  Instead, declare it in the Runtime Contract; let the target runtime manager or the user's tools resolve it.

Claiming all platforms read CLAUDE.md
  Instead, write "clients that support CLAUDE.md may read it."
```

## 5. Checklist

```text
[ ] Agent, Skill, Workflow Agent Skill, SubAgent stay canonical.
[ ] runtime, Runtime Contract, doctor, prepare, repair, resolve are used consistently.
[ ] Platform names appear only as compatibility examples.
[ ] Prose does not change any technical field name.
```
