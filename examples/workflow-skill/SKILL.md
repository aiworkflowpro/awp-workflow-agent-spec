---
name: public-release-reviewing
description: Use when the user asks to review an existing repository or software package before public release. This Skill is not for creating new packages or publishing releases.
license: MIT
compatibility: Requires shell access to run read-only file checks. Optionally uses gitleaks to strengthen secret scanning.
---

# Public Release Review

## Scope

Review the public-release readiness of an existing repository or software package.

Do not:

- Publish the repository.
- Delete files.
- Rewrite the whole project.
- Read credentials outside the target directory.
- Print suspected secrets in full.

## Input

Required:

- Target directory.

Optional:

- Release channel.
- Risk focus.
- Whether the user wants fix suggestions only, or direct fixes.

## Workflow

1. Read `workflow/step01-prepare.md`.
2. Read `workflow/step02-execute.md`.
3. Read `workflow/step03-verify.md`.

Use a run directory for a non-trivial review:

```text
runs/{keyword}-YYYYMMDD-HHMMSS/
  state/
    config.json
    progress.json
  step01-prepare/
  step02-execute/
  output/
```

## Output

Final reply order:

1. Release verdict.
2. Blocking issues.
3. Non-blocking issues.
4. Suggested fixes.
5. Verification performed.
6. Remaining risk.

## Security

If a suspected secret is found, redact the value and report only the file path, key name, and risk.

If a destructive script or command is found, report it as a blocking issue unless it defaults to dry-run and requires explicit approval.
