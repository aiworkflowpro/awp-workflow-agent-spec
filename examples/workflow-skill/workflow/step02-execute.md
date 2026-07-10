# Step 02: Execute Review

## Goal

Check the target directory for public-release risks.

## Input

- `state/config.json`
- `step01-prepare/file-index.txt`

## Actions

1. Read the README, license, package config, scripts, examples, and install files when present.
2. Search for suspected secrets and private paths.
3. Check whether install and usage instructions are reproducible.
4. Check whether examples use placeholder data.
5. Check whether scripts have destructive default behavior.
6. Check whether required dependencies are declared.
7. Write findings to `step02-execute/findings.json`.

Recommended read-only commands:

```bash
find <target> -maxdepth 3 -type f
rg -n "token|secret|password|cookie|api_key|apikey|/Users|~/Downloads|client|customer" <target>
```

Optional secret scan:

```bash
gitleaks detect --no-git --source <target> --redact --no-banner
```

## Output

```text
step02-execute/findings.json
step02-execute/notes.md
state/progress.json
```

## Completion Criteria

- Findings distinguish blocking issues from non-blocking issues.
- Every blocking issue has evidence and a concrete fix.
- Suspected secrets are redacted.
- `current_step` in the progress state is `step03-verify`.

## Failure Handling

If `gitleaks` is unavailable, keep using text search and record that the dedicated secret scanner did not run.

If a command fails due to permissions, record the path and continue checking only accessible files.
