# Step 01: Prepare

## Goal

Resolve the target directory, create a run directory if needed, and record the review configuration.

## Input

- The user request.
- The target directory.

## Actions

1. Confirm the target directory exists.
2. List top-level files and folders.
3. Identify the package type from visible files.
4. Create a run directory for a review that must check multiple files.
5. Write `state/config.json`.
6. Initialize `state/progress.json`.

## Output

```text
state/config.json
state/progress.json
step01-prepare/file-index.txt
```

## Completion Criteria

- The target directory exists.
- The review scope is clear.
- `current_step` in the progress state is `step02-execute`.
- The next step can run without re-reading the full session.

## Failure Handling

If the target directory does not exist, stop and ask the user for a valid path.

If the directory is very large, check only the top-level structure first; ask the user before a broad scan.
