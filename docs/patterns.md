# Design Patterns and Anti-Patterns

> This document specifies common Skill architecture patterns, composition, and anti-patterns.

## 1. Three Principles

### 1.1 Progressive Disclosure

Load information on demand:

```text
L1  frontmatter
    name and description, so the Agent can decide whether to trigger.

L2  SKILL.md
    Core rules and the execution map, read after triggering.

L3  External resources
    workflow/, references/, scripts/, assets/, read per step.
```

### 1.2 Single Responsibility

One Skill centers on one kind of task. A complex process can compose several Skills, but do not make one Skill a do-everything toolbox.

### 1.3 Portability

A public Skill assumes the reader lacks the author's environment. So:

- Do not hardcode private paths.
- Do not hardcode private accounts.
- Do not depend on undisclosed services.
- Do not write platform extensions as a general standard.

## 2. Pattern Quick Reference

```text
P1 Sequential orchestration
  Multi-step tasks, Step 1 → Step 2 → Step 3.

P2 Multi-tool coordination
  Several CLIs, MCPs, or APIs working together.

P3 Iterative refinement
  Draft → validate → fix → re-validate.

P4 Context-aware routing
  Choose a different path by input type.

P5 Domain-knowledge encapsulation
  Put professional rules in references/.

P6 Parallel review
  Process independent batches in parallel, then merge.

P7 Visualized output
  JSON data → HTML or Markdown report.

P8 Scheduled execution
  Suits monitoring, patrols, periodic collection.

P9 Chained composition
  Skill A's output as Skill B's input.

P10 Public-release audit
  A unified gate over security, dependencies, docs, and tests.
```

## 3. Typical Patterns

### 3.1 Sequential Orchestration

Suits report generation, pre-release checks, file processing, and review flows.

```text
workflow/
  step01-prepare.md
  step02-execute.md
  step03-verify.md
```

Key points:

- Each step has input and output.
- Each step updates state.
- Failure is localizable.

### 3.2 Multi-Tool Coordination

Suits multi-source collection, cross-platform publishing, and multi-tool audits.

```text
step01-load-config
step02-call-tool-a
step03-call-tool-b
step04-merge-results
step05-report
```

Key points:

- Each tool outputs an intermediate file.
- Tools do not depend on each other directly.
- Error handling is unified.

### 3.3 Iterative Refinement

Suits content optimization, code fixing, and report-quality improvement.

```text
draft
validate
fix
re-validate
finalize
```

Key points:

- There is an exit condition.
- There is a maximum iteration count.
- Each round's artifact has a version or a latest pointer.

### 3.4 Context-Aware Routing

Suits multi-file-type processing, multi-mode review, and multi-platform adaptation.

```text
Input is Markdown  → step02-markdown
Input is PDF       → step02-pdf
Input is directory → step02-directory
```

Key points:

- Routing conditions are objective.
- The routing reason is written into state.
- There is a fallback when nothing matches.

### 3.5 Domain-Knowledge Encapsulation

Suits legal review, security audits, platform-release rules, and brand-format specs.

```text
references/
  rules/
  examples/
  templates/
```

Key points:

- Professional knowledge is readable.
- Rules are verifiable.
- The public edition contains no private customer material.

### 3.6 Parallel Review

Suits independent review of multiple files, multi-candidate evaluation, and multi-source organization.

```text
step01-split/
step02-batch-a/
step02-batch-b/
step03-merge/
```

Key points:

- Batches do not write the same file.
- The merge step de-duplicates uniformly.
- A parallel failure can be retried individually.

### 3.7 Visualized Output

Suits reports, dashboards, and comparison results.

```text
data.json
template.html
report.html
```

Key points:

- Data and template are separated.
- The HTML does not leak local paths.
- It can fall back to a Markdown report.

### 3.8 Chained Composition

Suits end-to-end workflows.

Rules:

- Skill A outputs a stable file.
- Skill B reads that file as input.
- Do not pass data through chat context.

## 4. Anti-Patterns

### 4.1 Do-Everything Assistant

One Skill tries to handle every task, causing trigger confusion.

Fix:

- Split into several Skills.
- Narrow the description.
- State the exclusions clearly.

### 4.2 Bloated Entry

`SKILL.md` holds every tutorial, case, and troubleshooting note.

Fix:

- Keep the entry contract.
- Split long content into `docs/` or `references/`.

### 4.3 Stateless Long Task

A long task with no `runs/` or `progress.json`.

Fix:

- Create a run directory.
- Write artifacts each step.
- Update the recovery hint.

### 4.4 Black-Box Script

The script cannot run standalone and returns success even on failure.

Fix:

- Make arguments explicit.
- Output JSON.
- Return a non-zero exit code on failure.

### 4.5 Private Dependency

A public example depends on the author's local paths or accounts.

Fix:

- Swap in placeholder paths.
- Switch to environment variables.
- State what the user must provide.

### 4.6 No Tests

Relying only on manual gut checks.

Fix:

- Add positive and negative trigger cases.
- Add a main-path test.
- Add a security scan.

## 5. Checklist

```text
Selection
  [ ] The pattern matches the task complexity.
  [ ] No over-engineering for the sake of completeness.
  [ ] Multi-mode has clear routing.

Composition
  [ ] Different modes pass data via files.
  [ ] Parallel tasks write to separate directories.
  [ ] The merge step has verification.

Anti-patterns
  [ ] Not a do-everything assistant.
  [ ] The entry is not too long.
  [ ] Long tasks have state.
  [ ] The public edition has no private dependencies.
```
