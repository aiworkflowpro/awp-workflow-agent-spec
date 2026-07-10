# Configuration-Layering Spec

> This document specifies a Skill's runtime parameters, default configuration, preset data, and runtime declarations.

## 1. Responsibility

The configuration spec addresses:

- Which parameters differ on every run.
- Which parameters are defaults.
- Which options are static presets.
- How parameters merge.
- What must not go into configuration.

Configuration is not run state, and it is not a credential store.

## 2. Three-Layer Model

```text
L1 Runtime parameters
  Different on every run.
  Stored in state/config.json.

L2 Default configuration
  The Skill's built-in defaults.
  Stored in config/default.json.

L3 Preset data
  Options, enums, template variables.
  Stored in references/presets/ or references/definitions/.
```

Priority:

```text
User input > Agent inference > state/config.json > config/default.json > script built-in default
```

## 3. L1 Runtime Parameters

Suitable for `state/config.json`:

- Target path.
- The current mode.
- The current keyword.
- The user-selected output format.
- The input type inferred by the Agent.
- The current run directory.

Example:

```json
{
  "target": "/path/to/project",
  "mode": "release-review",
  "keyword": "project-review",
  "input_type": "directory",
  "run_dir": "runs/project-review-20260430-091500"
}
```

Rules:

- Do not write runtime parameters back into `config/default.json`.
- Prefer to record the source of an inferred parameter.
- The current path may be a user's private path and should not enter public examples.

## 4. L2 Default Configuration

Suitable for `config/default.json`:

- Timeout.
- Retry count.
- Default output language.
- Batch size.
- Whether to dry-run.
- Maximum file count.
- Default report format.

Example:

```json
{
  "runtime": {
    "timeout_seconds": 60,
    "retry": 2,
    "dry_run": true
  },
  "limits": {
    "max_files": 200,
    "max_file_size_kb": 512
  },
  "output": {
    "language": "en",
    "format": "markdown",
    "write_report": true
  }
}
```

Rules:

- Defaults should let the Skill run as-is.
- Nesting no deeper than two levels.
- No real secrets.
- No run artifacts.
- No this-run user input.

## 5. L3 Preset Data

Suitable for `references/presets/`:

- Mode list.
- Output types.
- Platform enums.
- Report levels.
- Style options.

Example:

```json
{
  "modes": [
    {
      "id": "check",
      "label": "Check only",
      "description": "Output issues only; do not modify files."
    },
    {
      "id": "fix",
      "label": "Check and fix",
      "description": "Modify in-scope files after explicit user authorization."
    }
  ]
}
```

## 6. Parameter Classification

```text
Required
  Cannot execute reliably without it.
  E.g. the target path.

Inferable
  Can be inferred from input, path, or file extension.
  E.g. input_type.

Defaultable
  Can be read from config/default.json.
  E.g. timeout_seconds.

Optional
  Absence does not affect the core flow.
  E.g. report_title.
```

The Agent should infer first, then ask.

## 7. Asking for Parameters

Ask only for missing required parameters. Each question states its purpose.

Recommended:

```text
I need the target directory path to scan for public-release risks.
```

Not recommended:

```text
Give me the parameters.
```

If several parameters are needed, ask all at once.

## 8. Runtime Contract

When a Skill needs a specific runtime, system binary, local model, provider cache, or network capability, use `config/runtime.json` to declare a Runtime Contract.

The Runtime Contract stores only a "needs declaration," not the environment itself. A public Skill should not put `.venv`, `node_modules`, model weights, large caches, or the author's local paths into the repository.

### 8.1 When It Applies

Cases that need `config/runtime.json`:

- There is a script runtime requirement such as Python, Node.js, Deno, Bun, Bash, Rust, or Go.
- A system binary such as `ffmpeg`, `tesseract`, `poppler`, `imagemagick`, or `gitleaks` is required.
- Local model assets such as OCR, ASR, VAD, embedding, rerank, or vision are required.
- A provider SDK's large cache is required, and the cache location affects reproducibility.
- Network access, a fixed API host, a proxy, or an offline-degradation strategy is required.

Cases that do not need `config/runtime.json`:

- Only `SKILL.md`, no scripts to run.
- No network access.
- No local model or system binary needed.
- All capabilities come from the Agent client's built-in tools, with no extra environment requirement.

### 8.2 Standard Structure

Example:

```json
{
  "schema_version": "1.0",
  "skill": "public-release-reviewing",
  "profiles": ["minimal", "standard", "full"],
  "envs": [
    {
      "id": "python-main",
      "kind": "python",
      "required": true,
      "version": ">=3.11",
      "manager": "uv",
      "lockfile": "scripts/python/uv.lock"
    }
  ],
  "binaries": [
    {
      "name": "gitleaks",
      "required": false,
      "purpose": "Secret scanning",
      "install_hint": "See the official gitleaks install docs."
    }
  ],
  "models": [
    {
      "id": "example/asr-small",
      "role": "asr-default",
      "kind": "asr",
      "required": false,
      "size_mb": 142,
      "sha256": "fill in a publicly verifiable sha256",
      "license": "fill in the model license",
      "source": {
        "type": "url",
        "url": "https://example.com/model.bin"
      }
    }
  ],
  "cache_policy": {
    "root": "runtime",
    "sync": false,
    "allow_provider_cache": true
  },
  "network": {
    "required": false,
    "hosts": [],
    "offline_behavior": "stop-or-degrade-with-record"
  }
}
```

### 8.3 Field Notes

```text
schema_version
  The Runtime Contract version. Start at "1.0".

skill
  The Skill name; should match the directory name and the frontmatter name.

profiles
  Optional run profiles, e.g. minimal, standard, full, gpu, offline.

envs
  Script-runtime declarations. Declare only language, version, package manager, and lockfile — no environment artifacts.

binaries
  System-binary declarations. State required, purpose, install_hint.

models
  Local model-asset declarations. State role, kind, required, size, checksum, license, source.

cache_policy
  Whether the cache is allowed, whether it syncs, and who manages it. Public repos default to sync=false.

network
  Network needs, host allow-list, offline behavior, and degradation strategy.
```

Recommended values for `models[].kind`:

```text
ocr
asr
vad
diarization
vision
segmentation
embedding
rerank
other
```

### 8.4 Runtime Artifact Boundary

```text
May go into the repository
  config/runtime.json
  lockfile
  Small example configs
  Preparation notes in setup.md

Does not go into the repository
  .venv/
  node_modules/
  Large model weights
  Provider cache
  The author's local absolute paths
  Real credentials
```

The runtime environment, models, and cache should be prepared by the target client, a runtime manager, or the user's local tools. The public spec only requires the Skill to state its needs and verification method; it does not require using a particular private runtime tool.

### 8.5 doctor / prepare / repair / resolve

If a Skill depends on a runtime manager, split the actions into four kinds, and do not write a particular private CLI as a general requirement:

```text
doctor
  Diagnose only; do not create environments, download models, or modify the system.

prepare
  Prepare the environment, dependencies, and model assets after the user explicitly runs it.

repair
  Repair a broken environment, a checksum mismatch, or a polluted cache.

resolve
  Return the actual paths of env, binary, model, and cache so scripts do not hardcode paths.
```

Public docs may use placeholder commands:

```bash
<runtime-manager> skill doctor .
<runtime-manager> skill prepare . --profile standard
<runtime-manager> skill repair . --profile standard
<runtime-manager> resolve model --skill public-release-reviewing --role asr-default
```

If there is no runtime manager, the setup doc must give equivalent manual checking steps.

## 9. Reading Configuration

Script reading order:

```text
1. Read state/config.json.
2. Read config/default.json.
3. Merge parameters.
4. Command-line arguments override configuration.
5. Fail if a required parameter is missing.
```

## 10. Prohibitions

- Writing real secrets into configuration.
- Writing run artifacts into the default configuration.
- Writing this-run user input back into the repository default configuration.
- Deep nesting that makes scripts hard to read.
- Using configuration to hide a non-publishable dependency.
- Configuration that executes destructive actions by default.
- Writing runtime artifact directories into configuration.
- Silently downloading when a model or environment is missing.
- Scripts bypassing the Runtime Contract to hardcode model/cache paths.

## 11. Checklist

```text
Layering
  [ ] L1 is written to state/config.json.
  [ ] L2 is written to config/default.json.
  [ ] L3 is written to references/presets/ or definitions/.

Security
  [ ] No real secrets.
  [ ] No private paths.
  [ ] No customer data.

Usability
  [ ] Defaults are runnable.
  [ ] Missing required parameters have a clear stop condition.
  [ ] Inferred parameters record a source.

Runtime
  [ ] config/runtime.json exists when an environment, binary, model, or cache is required.
  [ ] runtime.json stores declarations only, not artifacts.
  [ ] Model declarations include role, kind, required, checksum, license, source.
  [ ] cache_policy.sync defaults to false.
  [ ] setup.md documents doctor, prepare, repair, or an equivalent manual step.
```
