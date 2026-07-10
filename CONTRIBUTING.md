# Contributing Guide

Contributions should make this specification clearer, safer, or easier to apply.

## Good Contributions

These are welcome:

- Clearer explanations of Skill boundaries.
- Better templates.
- Small, real examples.
- Trigger-behavior test cases.
- Security-review improvements.
- Notes on platform differences.
- Corrections after official Skill behavior changes.

## Contributions We Don't Accept

Do not submit:

- Real credentials.
- Customer or client data.
- Private screenshots.
- Maintainer-specific absolute paths.
- Scripts that depend on private infrastructure.
- Marketing copy.
- "Do-everything assistant" templates with no clear boundaries.
- Examples that cannot be reproduced outside a private environment.

## Style

Write for Agents and maintainers, not for marketing.

Recommended style:

- Short sections.
- Concrete rules.
- Verifiable examples.
- Explicit inputs and outputs.
- Clear security boundaries.

Avoid:

- Vague quality promises.
- Long conceptual introductions.
- Several equivalent options with no default.
- Claiming that every Agent platform supports the same set of fields.

## PR Checklist

```text
[ ] The change is publishable.
[ ] No secrets or private paths added.
[ ] New templates have a matching example or note.
[ ] New scripts document their inputs and failure behavior.
[ ] New platform claims link to official docs where possible.
[ ] README links still point to files that exist.
[ ] If the change affects release behavior, the security and testing docs are updated.
```

## Local Review Commands

Run from the repository root:

```bash
gitleaks detect --no-git --source . --redact --no-banner
rg -n "token|secret|password|cookie|api_key|apikey|client|customer|/Users|~/Downloads|private" .
find . -type f -maxdepth 3
```

Before opening a PR, review each search hit one by one.
