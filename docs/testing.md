# Agent Skill Testing and Release Spec

Skill testing has two goals:

1. The Skill triggers when it should, and stays silent when it should not.
2. The output-quality gain the Skill provides is worth the extra context, time, and complexity.

Testing should come before large-scale implementation. A Skill with no verifiable samples is hard to maintain stably.

## 1. Test Types

```text
Trigger test
  Verifies name, description, and the optional when_to_use.

Function test
  Verifies the workflow, scripts, output, and recovery behavior.

Recovery test
  Verifies that after an interruption it can read progress.json and continue.

Security test
  Verifies the Skill leaks no secrets and runs no unsafe actions.

Runtime test
  Verifies the runtime missing, corrupt, degradation, and repair paths.

Regression test
  Compares the modified Skill with the previous version or a no-Skill baseline.

Cross-model test
  Verifies that key steps still execute under different model capabilities.
```

## 2. Trigger Eval

Create real user prompts and label whether the Skill should trigger.

Prepare about 20 prompts:

- 8 to 10 that should trigger.
- 8 to 10 that should not trigger.

Good trigger prompts should cover:

- Formal phrasing and colloquial phrasing.
- Typos and abbreviations.
- Direct task descriptions and indirect ones.
- Short requests and long requests.
- Single-step requests and multi-step requests.
- Similar-but-out-of-scope tasks.

Store trigger cases in `evals/trigger-queries.json`.

Each case should include:

```json
{
  "id": "should-trigger-001",
  "query": "Check this existing Skill and tell me whether it is safe to release.",
  "should_trigger": true,
  "reason": "The user asks to review the Skill's release safety."
}
```

## 3. Function Eval

Function tests should use real input and verify concrete output.

Minimum checks:

```text
[ ] Required inputs are parsed.
[ ] Missing a required input yields a clear stop condition.
[ ] Expected output files exist.
[ ] JSON output parses successfully.
[ ] Report sections match the documented template.
[ ] Scripts return a non-zero exit code on failure.
[ ] Errors contain enough detail to fix the problem.
```

Avoid vague assertions:

```text
The output is good.
The result is professional.
The writing quality is high.
```

Prefer verifiable assertions:

```text
The output contains a "Blocking Issues" section.
Every finding references a file path.
The generated JSON contains status, current_step, completed_steps, and resume_hint.
When the input file does not exist, the script exits with a non-zero code.
```

## 4. Recovery Eval

Any long-running Skill should prove it can recover from an interruption.

Test order:

1. Start a run.
2. Stop after a middle step.
3. Re-enter using the same run directory.
4. Confirm the Agent reads `state/progress.json`.
5. Confirm completed steps are not repeated, unless the user explicitly asks.
6. Confirm new output lands in the same run directory.

Required recovery fields:

```text
status
current_step
completed_steps
failed_steps
outputs
resume_hint
```

## 5. Script Eval

Scripts should be tested outside the Agent.

Required checks:

```text
[ ] The script has a documented command.
[ ] The script handles missing input.
[ ] The script handles malformed input.
[ ] The script writes output to the specified path.
[ ] The script prints useful errors.
[ ] The script does not depend on maintainer-only paths.
[ ] The script can run in dry-run when it has side effects.
```

## 6. Security Eval

Before release, run:

```bash
gitleaks detect --no-git --source . --redact --no-banner
rg -n "token|secret|password|cookie|api_key|apikey|client|customer|/Users|~/Downloads|private" .
```

Then review each hit manually.

Any one of the following should block release:

- Real credentials.
- Private paths.
- Customer data.
- Hidden external dependencies.
- Destructive default behavior.
- Network calls with no timeout or error handling.

## 7. Runtime Eval

The runtime eval should cover the env, binary, model, cache, and network declared in `config/runtime.json`.

Minimum checks:

```text
[ ] Stop when a required env is missing.
[ ] Stop when a required binary is missing.
[ ] Record skipped_checks when an optional binary is missing.
[ ] Stop when a required model's checksum mismatches.
[ ] Give a repair suggestion when the cache is not writable.
[ ] Stop or record degradation when the network is unavailable.
```

Store runtime cases in `evals/runtime-cases.json`.

## 8. Regression Eval

When improving an existing Skill, compare against a baseline:

```text
Baseline
  The Skill's previous version, or no Skill.

Candidate
  The current, modified version.
```

Record:

```text
pass_rate
duration_ms
token_estimate
failed_assertions
human_feedback
```

A Skill is better only when the quality gain is worth the added complexity. If quality improves little but token usage rises noticeably, the change is usually not worth it.

## 9. Cross-Model Eval

If the Skill targets multiple models or clients, check at least:

- Whether the entry is clear enough.
- Whether steps are clearly numbered.
- Whether the output format is stable.
- Whether complex judgments have examples.
- Whether failure handling is executable.

A weaker model needs clearer steps, examples, and output formats. A stronger model still should not be forced to read redundant background.

## 10. Eval-Driven Development

Recommended flow:

```text
1. Write the trigger eval.
2. Write a minimal SKILL.md.
3. Run the main path.
4. Record failures.
5. Add steps, scripts, or context.
6. Run the eval again.
7. Expand capabilities only after it passes.
```

Do not write a huge Skill first and then add tests by feel.

## 11. Release Gates

Do not release until this checklist passes:

```text
Trigger
  [ ] The trigger eval includes positive and negative cases.
  [ ] description covers the trigger phrasings.
  [ ] description states the exclusion scenario.

Function
  [ ] The main path passes.
  [ ] Expected output files exist.
  [ ] JSON output is parseable.
  [ ] Report sections are complete.

Recovery
  [ ] Recovery behavior is tested where applicable.
  [ ] progress.json is readable.
  [ ] resume_hint is executable.

Scripts
  [ ] Scripts are tested directly where applicable.
  [ ] Scripts return a non-zero exit code on failure.
  [ ] Destructive actions default to dry-run.

Security
  [ ] The security scan has no unresolved findings.
  [ ] No real credentials.
  [ ] No private paths.
  [ ] No customer data.

Runtime
  [ ] config/runtime.json is parseable where applicable.
  [ ] doctor / prepare / repair or a manual equivalent is documented.
  [ ] The runtime missing and corrupt paths have an eval.

Docs
  [ ] The README and examples match actual behavior.
  [ ] Public claims do not exceed tested capabilities.
  [ ] The setup, security, and testing docs are in sync.
```
