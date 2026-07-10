# Environment Setup Spec

> This document specifies how a Skill states its install location, dependency preparation, runtime checks, optional model assets, and repair flow.

## 1. Responsibility

The setup doc answers:

- Where the Skill goes.
- Which runtimes are needed.
- Which commands are needed.
- Which environment variables are needed.
- How to verify the environment.
- How to repair missing dependencies.

A public Skill should not modify the system environment by default.

## 2. When a Setup Doc Is Needed

Needs a setup doc:

- The Skill has scripts.
- The Skill has external dependencies.
- The Skill needs a network API.
- The Skill needs a local binary.
- The Skill needs model assets.
- The Skill has optional credentials.

Not necessarily needed:

- Only a single `SKILL.md`.
- No scripts to run.
- No network access.

## 3. Standard Structure

```markdown
# Environment Setup

## Install Location

## Runtime Requirements

## Dependency Checks

## Dependency Installation

## Runtime Contract

## Runtime doctor

## Runtime prepare

## Credential Configuration

## Verification Commands

## Common Errors
```

## 4. Install Location

State where the user should put the Skill, but do not name only one platform.

Example:

```text
Place this directory in the Skill directory your Agent client supports.
Paths differ across clients; defer to the client's docs.
```

If it is a specific note for one Agent client, mark the applicable scope, e.g. Claude Code, Codex, Cursor, or another client.

## 5. Runtime Requirements

If the Skill includes `config/runtime.json`, the setup doc must first state the location and responsibility of the Runtime Contract:

```text
Runtime Contract
  File: config/runtime.json
  Responsibility: declare runtime, binary, model, cache, and network needs.
  Boundary: stores declarations only, not .venv, node_modules, model weights, or provider cache.
```

Example:

```text
Python
  Requires Python 3.11 or higher.

Shell
  Requires bash or zsh.

Optional commands
  gitleaks: for secret scanning.
```

## 6. Runtime doctor

doctor only diagnoses; it does not modify the environment, download models, or write to system directories.

If a runtime manager exists, the setup doc may give a placeholder command:

```bash
<runtime-manager> skill doctor .
```

If there is no runtime manager, give equivalent manual checks:

```bash
python3 --version
node --version
command -v rg
command -v gitleaks
```

doctor output should state:

- Which env is missing.
- Which binary is missing.
- Which model is missing.
- Which cache is not writable or is polluted.
- Whether the missing item blocks the main flow.

## 7. Runtime prepare / repair

prepare and repair must be run explicitly by the user, and must not trigger silently during Skill execution.

```bash
<runtime-manager> skill prepare . --profile standard
<runtime-manager> skill repair . --profile standard
```

Without a runtime manager, the setup doc should give a step-by-step manual method, and state clearly which steps go online, download models, or write to the local cache.

Prohibited:

- Auto-installing global dependencies.
- Auto-downloading large models.
- Auto-modifying the shell config.
- Auto-writing to system directories.
- Temporarily falling back to the system Python inside a script to keep a production flow running.

## 8. Dependency Checks

Check only; do not modify:

```bash
python3 --version
command -v rg
command -v gitleaks
```

The check result should tell the user:

- What is missing.
- Whether it is required.
- Which features are unavailable if missing.

## 9. Dependency Installation

A public Skill may give install suggestions but should not auto-install by default.

Example:

```text
On macOS, install gitleaks via Homebrew.
On Linux, see the official gitleaks install docs.
```

Do not write commands that only run on the author's machine.

## 10. Credential Configuration

Reference [credentials.md](credentials.md).

State clearly:

- Environment variable names.
- Permission scope.
- Missing behavior.
- Whether dry-run is available.

## 11. Model Assets

If the Skill needs a local model or large asset:

- State the asset name.
- State the purpose.
- State the source.
- State the license restriction.
- Do not commit large assets to a public repository.
- Give a clear error when missing.

A public repository does not auto-download large models by default.

If `config/runtime.json` declares a model, the setup doc should list:

```text
Model ID
  Matches models[].id in runtime.json.

Role
  E.g. asr-default, ocr-default, embedding-default.

Kind
  E.g. asr, ocr, embedding, vision.

Required
  When required=true, missing must stop.

Verification
  sha256, license, source must be traceable.
```

## 12. Cache and Network

If the Skill uses a provider cache or network access, the setup doc should state:

- Whether the cache is allowed.
- Whether the cache syncs.
- How to clean or repair a corrupted cache.
- Which hosts must be accessed.
- Whether to stop or degrade when offline.
- Which outputs are no longer guaranteed complete after degradation.

A public Skill should not require the reader to use the author's local cache.

## 13. Verification Commands

Every setup doc should give verification commands.

```bash
python3 -m json.tool templates/progress.json >/dev/null
gitleaks detect --no-git --source . --redact --no-banner
```

Verification commands should be read-only.

## 14. Repair Flow

Recommended structure:

```text
Problem: gitleaks is missing
Impact: cannot run the dedicated secret scanner
Handling: keep using text search, or install gitleaks and retry
```

For runtime errors, use a structured form:

```json
{
  "ok": false,
  "error_type": "runtime_missing",
  "message": "required model is not prepared",
  "fix": "Run the Runtime prepare step in setup.md."
}
```

## 15. Prohibitions

- Auto-installing global dependencies.
- Auto-downloading large models.
- Auto-modifying the shell config.
- Auto-writing to system directories.
- Install steps that need the author's private accounts.
- Writing real credentials into the setup doc.
- Committing runtime artifact directories to the Skill repository.
- Silently falling back to the system environment when runtime is missing.

## 16. Checklist

```text
Installation
  [ ] Install location is clearly stated.
  [ ] Required and optional dependencies are separated.
  [ ] Dependency-check commands are read-only.

Runtime
  [ ] The responsibility of config/runtime.json is stated.
  [ ] doctor is read-only.
  [ ] prepare/repair require explicit user execution.
  [ ] env, binary, model, cache, and network all have missing behavior.
  [ ] Does not commit .venv, node_modules, model weights, or provider cache.

Credentials
  [ ] Environment variable names are explicit.
  [ ] Missing-credential behavior is explicit.

Security
  [ ] Does not auto-modify the system.
  [ ] Does not auto-download large assets.
  [ ] Contains no real secrets.
```
