# Quality Rubric Spec

> This document defines the 100-point score, tier verdicts, and blockers for a Workflow Agent Skill.

## 1. Scoring Goal

Scoring is not about grading a document; it is about deciding whether a Skill is worth installing, reusing, releasing publicly, or adopting as a team standard.

The score should answer four questions:

- Can the Agent trigger it reliably.
- Can an unfamiliar user run it.
- Can it recover after an interruption.
- Is the public release auditable.

## 2. Tier Verdicts

```text
A 90-100
  Recommendable for public release. Complete structure, reproducible main path, verified security and recovery.

B 75-89
  Releasable. Has non-blocking issues that do not affect the core task.

C 60-74
  Internal trial only. Needs more tests, runtime, recovery, or security boundaries.

Blocked <60
  Should not be released. There are blocking issues in triggering, dependencies, security, or reproducibility.
```

When a blocker exists, the verdict is `blocked` even if the total score is high.

## 3. 100-Point Dimensions

```text
Entry contract 15 pts
  Stable name, clear description, explicit scope and exclusion scenarios.

Workflow structure 20 pts
  Executable workflow steps, each with input, output, completion criteria, and failure handling.

Runtime and setup 15 pts
  Environment, binary, model, cache, and network needs are declarable, diagnosable, and preparable.

State recovery 15 pts
  runs/, state/config.json, progress.json, and resume_hint support interruption recovery.

Security boundaries 15 pts
  No real credentials, no private paths, no customer data, destructive actions default to dry-run.

Testing and evaluation 15 pts
  Trigger, function, recovery, security, runtime, and cross-model evals have samples or execution records.

Documentation experience 5 pts
  README, user guide, examples, and release checklist are consistent; an unfamiliar user can start quickly.
```

## 4. Blockers

Any one of these directly blocks public release:

- The description cannot tell when to trigger.
- Examples depend on the maintainer's local paths, private accounts, or undisclosed services.
- The repository contains real credentials, customer data, or a private work directory.
- Deletion, upload, publishing, payment, mail, or remote modification runs by default.
- A runtime, model, or system binary is required, but with no declaration or diagnosis method.
- A long task has no recovery state, so failure leaves only chat history to rely on.
- The README promises an untested capability.
- A script returns success after failing.

## 5. Score-Record Format

Recommended for use in a review report:

```json
{
  "score": 86,
  "grade": "B",
  "decision": "ready-with-notes",
  "dimensions": {
    "entry_contract": 14,
    "workflow_structure": 18,
    "runtime_setup": 12,
    "run_state": 13,
    "security": 15,
    "testing": 10,
    "documentation": 4
  },
  "blockers": [],
  "notes": [
    "runtime doctor is documented, but no actual output sample is provided."
  ]
}
```

## 6. Checklist

```text
Verdict
  [ ] A total score is given.
  [ ] An A/B/C/Blocked tier is given.
  [ ] Blockers are listed separately.

Evidence
  [ ] Every deduction has file or behavior evidence.
  [ ] "Feels good" is not used in place of a verifiable assertion.

Release
  [ ] blocked is not allowed for public release.
  [ ] B/C must list pre-release or post-release fixes.
```
