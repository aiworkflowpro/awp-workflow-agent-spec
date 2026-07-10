# Step 03: Verify

## Goal

Confirm the review is complete, evidence-backed, and safe to report.

## Input

- `step02-execute/findings.json`
- `step02-execute/notes.md`
- `state/progress.json`

## Actions

1. Check that every blocking issue has file evidence.
2. Check that no suspected secret value is printed in full.
3. Check that suggested fixes stay within the target directory.
4. Check that skipped checks are listed.
5. Write the final release-review report to `output/public-release-report.md`.
6. Update `state/progress.json` to `completed`.

## Output

```text
output/public-release-report.md
state/progress.json
```

## Report Structure

```markdown
# Public Release Review Report

## Verdict

## Blocking Issues

## Non-Blocking Issues

## Suggested Fixes

## Verification

## Skipped Checks

## Remaining Risk
```

## Completion Criteria

- The final report exists.
- The verdict is one of `ready`, `ready-with-notes`, or `blocked`.
- Blocking issues have evidence.
- The verification section states which checks were run.
- The remaining risk is explicit.

## Failure Handling

If the findings are incomplete, return to step 02; do not guess.

If the report cannot be written, return the error and the expected report path.
