# Release and Versioning Spec

> This document defines the versioning, release gates, migration, and deprecation rules for the Workflow Agent Skill specification repository and for individual Skills.

## 1. Version Number

Use semantic versioning:

```text
MAJOR.MINOR.PATCH
```

Meaning:

```text
MAJOR
  A breaking spec change, e.g. incompatible directory structure, required fields, or runtime schema.

MINOR
  A backward-compatible capability addition, e.g. a new template, eval type, or optional field.

PATCH
  A documentation fix, example fix, or validator bugfix.
```

## 2. Release Gates

Before release, all of the following must pass:

```text
Structure
  [ ] README, SKILL.md, docs, templates, examples, and evals are consistent.
  [ ] Schemas are parseable.
  [ ] The validator passes.

Security
  [ ] gitleaks passes.
  [ ] Private-path and credential keywords are manually reviewed.
  [ ] Public examples contain no customer data.

Run
  [ ] The golden-example script runs.
  [ ] The runtime template is parseable.
  [ ] progress.json matches the schema.

Evaluation
  [ ] The trigger eval has both positive and negative cases.
  [ ] The function / recovery / security / runtime evals have samples.
  [ ] The README promises no unverified capability.
```

## 3. Changelog

Each release should record:

```text
Added
  New specs, templates, schemas, examples.

Changed
  Changes to existing spec behavior.

Fixed
  Fixes to errors, ambiguities, or example issues.

Security
  Changes related to security boundaries, credentials, or the public audit.

Migration
  What users must do to migrate from the old version to the new one.
```

## 4. Breaking Changes

A breaking change must:

- Bump the MAJOR version.
- Be noted in the CHANGELOG.
- Provide migration steps.
- Keep the old field documented for at least one minor-version cycle.
- Produce an understandable validator error.

## 5. Deprecation Rules

When deprecating a field, file, or directory:

```text
Deprecated
  State the version from which it is deprecated.

Replacement
  Provide the alternative.

Removal
  State the earliest removal version.
```

Do not delete content that examples or templates still reference.

## 6. Checklist

```text
[ ] The version number matches the change type.
[ ] The CHANGELOG is updated.
[ ] All release gates pass.
[ ] Breaking changes have migration notes.
[ ] Deprecated items have a replacement.
```
