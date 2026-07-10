# Security Boundaries

This document defines what a public Agent Skill repository may and may not contain.

The default rule is simple: a public Skill should be understandable, installable, and auditable without accessing the maintainer's private machine, private knowledge base, customer data, or accounts.

## Allowed

A public Skill may contain:

- General workflows.
- Links to public documentation.
- Placeholder configuration.
- Environment variable names.
- Minimal example data.
- Templates and schemas.
- Scripts that run in a clean environment.
- References to user-provided input paths.

Example:

```json
{
  "api_key_env": "EXAMPLE_API_KEY",
  "source_dir": "/path/to/your/project",
  "dry_run": true
}
```

## Not Allowed

A public Skill must not contain:

- Real API keys, tokens, cookies, passwords, or session data.
- Customer names, customer files, client datasets, or private screenshots.
- Personal absolute paths like `/Users/name/...` or private workspace paths.
- Internal service URLs, private endpoints, or non-public CLI names.
- Account routing, credential locations, or copies of paid content.
- Scripts that delete, overwrite, upload, publish, or send data by default.
- Prompts that depend on undisclosed private knowledge.

## Credential Rules

Use environment variables or placeholder JSON files.

Recommended:

```json
{
  "provider": "example",
  "api_key_env": "EXAMPLE_API_KEY"
}
```

Not recommended:

```json
{
  "api_key": "sk-real-key"
}
```

If a Skill needs a credential, record:

1. Which environment variable is needed.
2. Which permission scope is needed.
3. What happens when the credential is missing.
4. Whether it can run in a no-credential dry-run mode.

## File-System Rules

Examples may use placeholder paths only:

```text
/path/to/your/project
/path/to/input.md
```

Do not include:

```text
/Users/example/private-client
~/Downloads/private-workspace
```

If a script needs the Skill directory, use a runtime variable or a script-relative path, not a maintainer-only absolute path.

## Network Rules

Any Skill that performs network access must record:

- Which host or API it accesses.
- Whether the request sends user data.
- Timeout behavior.
- Retry behavior.
- Rate-limit behavior.
- Offline or dry-run fallback.

Network scripts should fail closed. When a request fails, produce a clear error and a recoverable state; do not silently skip the step.

## Destructive Operations

Destructive operations include:

- Deleting files.
- Overwriting user files.
- Moving large directories.
- Uploading files.
- Publishing content.
- Sending mail or messages.
- Modifying remote infrastructure.

Rules:

1. Default `dry_run: true`.
2. Generate a plan before executing.
3. Validate the plan against a source of truth.
4. Require explicit user approval when the Agent client supports it.
5. Write the actions actually taken into the run directory.

## Pre-Release Scan

Before release, run from the repository root:

```bash
python3.12 scripts/validate-spec.py --root .
gitleaks detect --no-git --source . --redact --no-banner
rg -n "token|secret|password|cookie|api_key|apikey|client|customer|/Users|~/Downloads|private" .
find . -name ".env*" -o -name "*cookie*" -o -name "*token*"
```

The search scope is deliberately broad. A hit is not necessarily a leak, but every hit must be reviewed before release.

## Manual Review Checklist

```text
[ ] No real credentials.
[ ] No private paths.
[ ] No customer or client data.
[ ] No private service URLs.
[ ] Basic use does not depend on unreleased internal commands.
[ ] Destructive operations default to dry-run.
[ ] Network behavior is documented.
[ ] Examples use placeholder data.
[ ] Scripts run in a clean environment, or clearly document their dependencies.
[ ] The README does not promise unsupported behavior.
```

## Reporting Security Issues

If you find a security issue in this repository, prefer submitting a private report via a GitHub security advisory. If that is unavailable, create a GitHub Issue that describes only the affected files and the risk — do not paste a still-valid secret.
