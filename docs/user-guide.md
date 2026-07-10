# User Onboarding Spec

> This document specifies how to write a getting-started guide aimed at ordinary users.

## 1. Responsibility

The onboarding doc answers:

- What this Skill can do.
- What it cannot do.
- What the minimal input is.
- How to run it.
- Where the output is.
- Where to look when errors occur.

It is for users, not maintainers.

## 2. Recommended Structure

```markdown
# User Guide

## Applicable Scenarios

## Inapplicable Scenarios

## Minimal Input

## How to Run

## Output Files

## FAQ

## Security Notes
```

## 3. Applicable Scenarios

Be concrete; do not state vague value.

Recommended:

```text
Use it when you have a Markdown file and want to check heading levels, links, image references, and code fences before publishing.
```

Not recommended:

```text
Helps you handle content efficiently.
```

## 4. Inapplicable Scenarios

State the exclusions clearly:

- Does not create from scratch.
- Does not publish automatically.
- Does not read private credentials.
- Does not modify out-of-scope files.

## 5. Minimal Input

Example:

```text
Required:
  - A path to a Markdown file.

Optional:
  - The target platform.
  - Whether to allow automatic fixes.
```

## 6. How to Run

Give a natural-language example:

```text
Using this Skill, check whether /path/to/article.md is ready to publish.
```

If a command is needed, give the command too:

```bash
python scripts/python/check.py --target /path/to/article.md --output runs/example/output/report.json
```

## 7. Output Files

State the artifact location:

```text
runs/{keyword}-{time}/
  output/
    final-report.md
```

State what the report contains:

- Release verdict.
- Blocking issues.
- Non-blocking suggestions.
- Verification performed.
- Remaining risk.

## 8. FAQ

Recommended format:

```text
Problem: No report was generated.
Cause: The target file does not exist or is not readable.
Handling: Confirm the path exists and run again.
```

## 9. Security Notes

The onboarding doc also states security boundaries:

- Will not publish by default.
- Will not delete files by default.
- Will not print full secrets.
- Network access is stated in setup or in the Skill.

## 10. Prohibitions

- Writing it as a marketing page.
- Discussing value only, not boundaries.
- Not stating the minimal input.
- Not stating the output location.
- Writing the maintainer's internal process for users.
- Promising untested platform compatibility.

## 11. Checklist

```text
Clarity
  [ ] What it can do is clear.
  [ ] What it cannot do is clear.
  [ ] The minimal input is clear.
  [ ] The output location is clear.

Executable
  [ ] There is a natural-language invocation example.
  [ ] There is a necessary command example.
  [ ] Common errors have a handling method.

Security
  [ ] States the dangerous actions it will not run by default.
  [ ] Contains no real paths or secrets.
```
