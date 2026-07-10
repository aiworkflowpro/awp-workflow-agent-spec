# Output Templates Spec

> This document specifies the structure and variable placeholders of a Skill's Markdown, JSON, and HTML output templates.

## 1. Responsibility

Output templates stabilize artifact structure so results are readable, verifiable, and reusable.

Suits templating:

- Review reports.
- Pre-release check reports.
- JSON results.
- HTML reports.
- User notes.
- Fix plans.

## 2. Template Location

```text
references/templates/
  report.md
  findings.schema.json
  dashboard.html
```

Or for a simple project:

```text
templates/
  report.md
```

Keep it consistent within one repository, and note it in the README or `SKILL.md`.

## 3. Markdown Report Template

```markdown
# {title}

## Verdict

{decision}

## Blocking Issues

{blocking_issues}

## Non-Blocking Issues

{non_blocking_issues}

## Suggested Fixes

{suggested_fixes}

## Verification Performed

{verification}

## Skipped Checks

{skipped_checks}

## Remaining Risk

{remaining_risk}
```

Rules:

- The section order is stable.
- Empty sections still state "None" or "Not checked."
- Review reports write blocking issues first.
- Evidence is a file path; do not print sensitive values.

## 4. JSON Output Template

```json
{
  "decision": "blocked",
  "blocking_issues": [],
  "non_blocking_issues": [],
  "suggested_fixes": [],
  "verification": [],
  "skipped_checks": [],
  "remaining_risk": []
}
```

Rules:

- Fields are stable.
- Types are stable.
- Empty arrays use `[]`.
- Missing information uses `null` or an explicit status; do not omit key fields.

## 5. HTML Template

HTML templates suit visualized reports.

Rules:

- Put templates in `references/templates/`.
- Inject data from JSON.
- Styles are self-contained or explicitly referenced.
- Do not embed remote tracking scripts.
- Do not leak local paths.

Basic structure:

```html
<!doctype html>
<html lang="en">
  <head>
    <meta charset="utf-8">
    <title>{{ title }}</title>
  </head>
  <body>
    <main>
      <h1>{{ title }}</h1>
      <section>{{ summary }}</section>
    </main>
  </body>
</html>
```

## 6. Variable Placeholders

Recommended:

```text
{{ title }}
{{ decision }}
{{ generated_at }}
{{ blocking_issues }}
```

Rules:

- Placeholders have a source.
- Missing values have a default or an error.
- Do not use a credential as a template variable.
- Do not execute arbitrary code in a template.

## 7. List Rendering

If the template engine supports loops, state the field structure clearly.

```text
blocking_issues[]
  severity
  file
  evidence
  impact
  fix
```

If not using a template engine, have the Agent or a script generate the complete Markdown.

## 8. Machine-Verifiable

Output should support automated checks:

- JSON is parseable.
- Required fields exist.
- Report sections are complete.
- File paths exist or are noted as inaccessible.
- Secret values are redacted.

## 9. Prohibitions

- Output structure that changes every time.
- Deleting empty sections outright.
- Rendering a full secret into a report.
- Rendering the author's local paths into public examples.
- HTML templates that load unknown remote scripts by default.
- Template logic too complex to test.

## 10. Checklist

```text
Structure
  [ ] Template location is clear.
  [ ] Section order is stable.
  [ ] JSON fields are stable.

Variables
  [ ] Placeholders have a source.
  [ ] Missing values have a handling strategy.
  [ ] No credential variables.

Verification
  [ ] JSON is parseable.
  [ ] Report sections are complete.
  [ ] Sensitive values are redacted.
```
