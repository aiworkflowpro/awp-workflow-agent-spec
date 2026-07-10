# Troubleshooting and Recovery Spec

> This document specifies a Skill's error classification, diagnosis order, retry strategy, and interruption-recovery flow.

## 1. Responsibility

The troubleshooting spec addresses:

- Which layer an error belongs to.
- Whether it is recoverable.
- Whether it should be retried.
- How to record the error.
- How to continue execution.

Do not use a broad prompt to paper over a structural problem.

## 2. Eight-Layer Error Classification

```text
L0 Runtime-asset layer
  Runtime env, local model, provider cache, or system binary missing or corrupt.

L1 Entry layer
  The Skill did not trigger, triggered wrongly, or the description is unclear.

L2 Parameter layer
  A required input is missing, a path is invalid, or the mode cannot be determined.

L3 Dependency layer
  A command is missing, a package is missing, or the runtime version is wrong.

L4 Credential layer
  An environment variable is missing, permission is insufficient, or a credential is invalid.

L5 Network layer
  Timeout, rate limit, service error, offline.

L6 File layer
  A file does not exist, permission is insufficient, or the output is not writable.

L7 Progress layer
  progress.json is corrupt, state is inconsistent, or an artifact is missing.
```

## 3. Diagnosis Order

```text
1. If config/runtime.json exists, check the Runtime Contract first.
2. Whether runtime env, binary, model, cache are prepared.
3. Whether the correct Skill triggered.
4. Whether required inputs are complete.
5. Whether the target path exists.
6. Whether dependency commands are available.
7. Whether credentials are needed and configured.
8. Whether the network is available.
9. Whether progress.json is parseable.
10. Whether the previous step's artifact exists.
```

Check in order; do not simply re-run everything.

## 3.1 Runtime-Asset Layer

Runtime-asset errors should be diagnosed before script-dependency errors. Typical errors:

```text
runtime_missing
  An env, binary, model, or cache declared in config/runtime.json is not prepared.

runtime_corrupt
  Model checksum mismatch, broken environment, polluted cache.

binary_missing
  A system binary is missing, e.g. ffmpeg, tesseract, gitleaks.

model_license_required
  The model license requires the user to download or confirm manually.

cache_not_writable
  The provider cache is not writable or fell into the wrong directory.
```

Structured error example:

```json
{
  "type": "runtime_missing",
  "message": "required model asr-default is not prepared",
  "recoverable": true,
  "suggested_fix": "Prepare the model asset via the Runtime prepare step in docs/setup.md."
}
```

Do not handle runtime errors in these ways:

- Silently downloading a large model inside a script.
- Temporarily switching to the system Python to keep a production flow running.
- Hardcoding the author's local runtime/cache paths.
- Copying the provider cache into the Skill repository.
- Using a local model without verifying its checksum.

## 4. Error Records

Write into `state/progress.json`:

```json
{
  "status": "failed",
  "current_step": "step02-execute",
  "failed_steps": ["step02-execute"],
  "errors": [
    {
      "type": "runtime_missing",
      "message": "gitleaks not found",
      "recoverable": true,
      "suggested_fix": "Install gitleaks or keep using text search."
    }
  ],
  "resume_hint": "After installing gitleaks, continue from step02-execute; you may also skip the dedicated secret scan."
}
```

## 5. Retry Strategy

Retryable:

- Network timeout.
- 5xx service error.
- Temporary rate limit.
- Recoverable file lock.

Do not retry blindly:

- 4xx permission error.
- Target path does not exist.
- JSON format error.
- Missing credential.
- Output directory not writable.
- Runtime checksum mismatch.
- Model license requires user confirmation.

## 6. Degradation Strategy

Degradation is allowed but must be recorded.

Example:

```text
gitleaks unavailable
  Degrade to rg text search.
  Record in skipped_checks that the dedicated scanner did not run.
```

Do not degrade silently.

Runtime degradation must also be recorded:

```text
asr-default model missing
  If the model is required=true, stop.
  If required=false, skip ASR analysis and record it in skipped_checks.
```

## 7. Interruption Recovery

Recovery steps:

1. Read `state/progress.json`.
2. Check `completed_steps`.
3. Verify the completed steps' artifacts exist.
4. Continue from `current_step`.
5. Do not repeat completed steps.
6. Write a new `updated_at` and a recovery record.

## 8. Corrupt progress.json

Handling:

1. Check the run-directory structure.
2. Rebuild state from the existing step artifacts.
3. If it cannot be determined, stop and ask the user which step to continue from.
4. Do not guess and run destructive actions.

## 9. Failure Report

On failure, the final reply should include:

- The failed step.
- The error type.
- Recoverability.
- Preserved artifacts.
- The suggested next step.
- Whether user input is needed.

## 10. Prohibitions

- Skipping tests right after an error.
- Turning every error into "failed" with a blanket try/except.
- Not updating `progress.json`.
- Re-running everything and overwriting artifacts.
- Fabricating results after a network failure.
- Continuing real requests after a credential is missing.

## 11. Checklist

```text
Classification
  [ ] Errors have a layer.
  [ ] Errors have a type.
  [ ] Recoverability is explicit.

Runtime
  [ ] When config/runtime.json exists, do the Runtime check first.
  [ ] runtime_missing, runtime_corrupt, binary_missing have a clear fix path.
  [ ] Stop when a required model or env is missing; do not fall back silently.
  [ ] Cache, model, and environment paths are not hardcoded.

Records
  [ ] progress.json is updated.
  [ ] resume_hint is executable.
  [ ] No sensitive values are printed.

Recovery
  [ ] Completed steps are not repeated.
  [ ] Missing artifacts cause a stop.
  [ ] Degradation records a reason.
```
