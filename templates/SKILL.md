---
name: example-task-reviewing
description: Use when the user asks to review the scope, safety, completeness, and release readiness of an existing task artifact. This Skill is not for creating artifacts from scratch.
license: MIT
compatibility: Designed for cross-platform Workflow Agent Skill clients. Platform extension fields must be marked with their applicable scope and a fallback.
---

# Example: Task Review

## Scope

Use this Skill to review an existing artifact.

Do not use this Skill to:

- Create a new artifact from scratch.
- Publish or upload anything.
- Modify user files, unless the user explicitly asks for an edit.
- Read credentials or private account data.

## Input

Required:

- Path to the artifact or directory to review.

Optional:

- Target audience.
- Release channel.
- A specific risk focus.

When it can be inferred safely:

- Infer the artifact type from the file extension or directory structure.
- Infer release readiness from surrounding files.

Ask the user only when missing information blocks a reliable review.

## Workflow

1. Resolve the target path.
2. Read only the minimal set of files needed to understand the artifact.
3. Identify scope, input, output, side effects, and release boundaries.
4. Check for security risks: secrets, private paths, destructive behavior, implicit dependencies.
5. Check for quality risks: unclear trigger conditions, missing output, missing verification, weak recovery.
6. Write blocking issues first, then everything else.
7. If the user asks for edits, patch only the files needed to resolve blocking issues.
8. Verify the result before replying.

## Files

May read:

- The target artifact.
- A surrounding README or config file.
- Referenced templates or examples.

May create:

- A review report when the user asks.

May modify:

- Only files explicitly in scope.

Must not read:

- Credential files.
- Private account files.
- Unrelated user directories.

## Tools

Use shell commands only for deterministic checks:

```bash
find <target> -maxdepth 3 -type f
rg -n "token|secret|password|cookie|api_key|/Users|~/Downloads" <target>
```

Do not run destructive commands.

## Output

Return findings in this order:

1. Blocking issues.
2. Non-blocking issues.
3. Suggested fixes.
4. Verification performed.
5. Remaining risk.

Each finding should include:

- Severity.
- File path.
- Evidence.
- Impact statement.
- A concrete fix.

## Verification

Before finishing, confirm:

- The target path exists.
- Every blocking issue has evidence.
- No real secret is printed in full.
- Suggested fixes are still in scope.
- Modified files have been re-read or re-checked.

## Failure Handling

If the target path does not exist, stop and ask the user for a valid path.

If a required file cannot be read, report the exact path and reason.

If a suspected secret is found, redact it in the reply and suggest removing it.
