# Scripts and Tools Spec

> This document specifies the usage boundaries of `scripts/`, CLI, MCP, network APIs, and external tools inside a Skill.

## 1. Responsibility

Scripts handle deterministic, repeatable, testable work. The Agent handles understanding, judgment, and fix suggestions.

Suits scripts:

- Scanning files.
- Validating JSON.
- Checking links.
- Parsing input.
- Batch conversion.
- Generating reports.
- Merging results.
- Running secret scans.

Does not suit scripts:

- Judging whether an article is good.
- Interpreting the intent of a request.
- Choosing a creative strategy.
- Producing conclusions that need semantic judgment.

## 2. Directory Structure

```text
scripts/
  python/
    *.py
    pyproject.toml
  node/
    *.mjs
    package.json
  shell/
    *.sh
```

Create as needed; do not create empty directories for the sake of completeness.

## 3. Run Principles

- Scripts run standalone.
- Input is passed via arguments or a config file.
- Output is written to the run directory.
- The terminal prints only a summary.
- Return values are machine-parseable.
- On failure, the exit code is non-zero.

## 4. Return Format

Recommended JSON:

```json
{
  "ok": true,
  "output": "step02-execute/findings.json",
  "count": 12
}
```

Failure:

```json
{
  "ok": false,
  "err": "target directory does not exist",
  "recoverable": true
}
```

Rules:

- `ok: true` maps to exit code 0.
- `ok: false` maps to a non-zero exit code.
- `err` does not print secrets or sensitive values.
- Output file paths are relative to the run directory or the Skill root.

## 5. Python Scripts

Recommended structure:

```python
from __future__ import annotations

import argparse
import json
import sys
from pathlib import Path


def run(target: Path, output: Path) -> dict[str, object]:
    if not target.exists():
        return {"ok": False, "err": "target does not exist", "recoverable": True}
    output.parent.mkdir(parents=True, exist_ok=True)
    output.write_text("{}", encoding="utf-8")
    return {"ok": True, "output": str(output)}


def main() -> int:
    parser = argparse.ArgumentParser()
    parser.add_argument("--target", required=True)
    parser.add_argument("--output", required=True)
    args = parser.parse_args()

    result = run(Path(args.target), Path(args.output))
    print(json.dumps(result, ensure_ascii=False))
    return 0 if result.get("ok") else 1


if __name__ == "__main__":
    raise SystemExit(main())
```

## 6. Shell Scripts

Shell scripts must:

- Use a shebang on the first line.
- Use `set -euo pipefail` on the second line.
- Error out on missing arguments.
- Default to read-only or dry-run.
- Output explicit paths.

Template:

```bash
#!/usr/bin/env bash
set -euo pipefail

target="${1:?target required}"
output="${2:?output required}"

find "$target" -maxdepth 3 -type f > "$output"
```

## 7. Dependency Declaration

Dependencies must be declared explicitly.

```text
Python
  pyproject.toml or an inline dependency declaration in the script.

Node.js
  package.json.

Shell
  The setup doc lists the required commands.
```

Do not let a script implicitly depend on global packages on the maintainer's machine.

## 8. Network Requests

Any network access must be documented:

- Which host it accesses.
- Whether it sends user data.
- The timeout.
- The retry count.
- The rate limit.
- The offline or dry-run behavior.

Default rules:

```text
timeout
  Must be set.

retry
  Retry only recoverable errors.

4xx
  Do not retry blindly; output a clear reason.

5xx
  May retry a limited number of times.

rate limit
  Wait per the service's response, or stop and record.
```

## 9. Reading Credentials

Scripts do not write real secrets.

Recommended:

```text
Read from an environment variable.
Read from a user-provided config path.
Read from the client's secure credential system.
```

Example config:

```json
{
  "provider": "example",
  "api_key_env": "EXAMPLE_API_KEY"
}
```

When a credential is missing:

- If the task can dry-run, enter dry-run.
- If the task requires the credential, stop and state which environment variable is missing.
- Do not prompt the user to write a key into a public file.

## 10. Script vs. Agent Division

```text
Scripts do
  - Scanning.
  - Parsing.
  - Conversion.
  - Validation.
  - Merging.
  - Rendering.

Agent does
  - Judging severity.
  - Explaining impact.
  - Choosing a fix.
  - Writing the review report.
  - Handling exceptions that can't be structured.
```

## 11. Idempotency

Scripts should be idempotent where possible:

- The same input run multiple times produces the same output.
- Not dependent on the current working directory.
- Do not overwrite unrelated files.
- Output is written to the specified path.
- Temporary files are written to the run directory.

## 12. Destructive Operations

Deleting, overwriting, uploading, publishing, sending mail, and modifying remote systems are all destructive operations.

Rules:

- Default `dry_run: true`.
- Generate a plan before executing.
- Require explicit user approval.
- Write the actual actions into the run directory.
- Roll back after failure, or state that it cannot be rolled back.

Public examples should not include destructive scripts by default.

## 13. Invocation

In a step file, write only the call command and its inputs/outputs; do not paste the full script code.

```text
## Script Execution

python scripts/python/scan.py --target "$TARGET" --output step02-scan/result.json

## Output

- step02-scan/result.json
```

## 14. Prohibitions

- Scripts that hardcode real secrets.
- Scripts that hardcode the author's absolute paths.
- Scripts that delete or upload by default.
- Scripts that return success after failing.
- Scripts that print full secrets.
- Scripts that treat a wall of logs as the only artifact.
- Scripts that depend on an undeclared global environment.

## 15. Checklist

```text
Structure
  [ ] scripts/ contains only scripts actually in use.
  [ ] Dependencies are declared.
  [ ] Scripts run standalone.

Input/Output
  [ ] Arguments are explicit.
  [ ] Output is written to the specified path.
  [ ] Returns JSON or an explicit status.

Security
  [ ] No hardcoded secrets.
  [ ] No private paths.
  [ ] Network requests have timeouts.
  [ ] Destructive actions default to dry-run.

Errors
  [ ] Failure returns a non-zero exit code.
  [ ] Error messages are actionable.
  [ ] No sensitive values are printed.
```
