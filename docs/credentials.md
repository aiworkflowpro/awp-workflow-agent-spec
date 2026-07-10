# Credential Declaration Spec

> This document specifies how a Skill declares, reads, and handles credentials. A public Skill writes only placeholders and reading methods; it stores no real secrets.

## 1. Responsibility

The credential spec addresses:

- Which credentials are needed.
- Which environment variable provides each credential.
- Which permission scope is needed.
- What to do when a credential is missing.
- Whether dry-run is available.
- How to avoid leaks.

## 2. Basic Principles

- A public repository stores no real credentials.
- Examples use placeholder values only.
- The credential-reading method must be explicit.
- Missing credentials must not fail silently.
- Logs and reports do not print full secrets.

## 3. Recommended Declaration

Declare in the setup doc or in a `credentials/` placeholder file:

```text
# Credential Notes

## EXAMPLE_API_KEY

Purpose: access the Example API.

Permission: read-only.

How to obtain: create a read-only key in the service dashboard.

How to configure:

  export EXAMPLE_API_KEY="your-key"

Missing behavior: enter dry-run; stop when a real request is required.
```

Note: no real key should appear in public examples.

## 4. Environment Variables

Recommended:

```json
{
  "api_key_env": "EXAMPLE_API_KEY"
}
```

Not recommended:

```json
{
  "api_key": "sk-real-key"
}
```

## 5. Permission Scope

Credential notes must state the permission clearly:

```text
Read-only
  For querying, downloading, and reading public or authorized data.

Write
  For creating, modifying, and uploading.

Publish
  For publishing content publicly.

Admin
  For modifying accounts, permissions, and infrastructure.
```

Request the least privilege by default.

## 6. dry-run

A Skill with external writes must support dry-run.

```json
{
  "dry_run": true
}
```

dry-run should:

- Send no real write requests.
- Upload no files.
- Publish no content.
- Output the plan it would execute.
- State which credentials are needed.

## 7. Missing Credentials

Handling rules:

```text
Optional credential
  Skip the related check and record skipped_checks.

Required credential but dry-run supported
  Enter dry-run and output the plan.

Required credential and dry-run not possible
  Stop and state the missing environment variable.
```

## 8. Log Redaction

Show only a few leading/trailing characters, or hide entirely.

```text
EXAMPLE_API_KEY=sk-...abcd
```

Safer:

```text
EXAMPLE_API_KEY=<redacted>
```

## 9. Prohibitions

- Writing a real key in `SKILL.md`.
- Writing a real key in an example JSON.
- Printing a full key in run logs.
- Echoing request headers in error messages.
- Using a high-privilege credential by default.
- Faking success when a credential is missing.

## 10. Checklist

```text
Declaration
  [ ] Every credential has an environment variable name.
  [ ] Every credential has a purpose note.
  [ ] Every credential has a permission scope.
  [ ] Missing behavior is explicit.

Security
  [ ] No real secrets.
  [ ] Logs are redacted.
  [ ] Examples use placeholder values.
  [ ] Least privilege by default.

Execution
  [ ] Write operations support dry-run.
  [ ] Missing credentials do not fail silently.
```
