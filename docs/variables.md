# Variables and Placeholders Spec

> This document specifies variables, placeholders, path references, source annotation, and latest pointers in a Skill.

## 1. Responsibility

Variables connect user input, configuration, step artifacts, and templates. The variable spec addresses:

- What a variable is named.
- Where a variable comes from.
- When a variable is resolved.
- What to do when a variable is missing.
- How a variable avoids ambiguity.

## 2. Variable Sources

```text
User input
  Provided by the user in this request.

Agent inference
  Inferred from path, filename, extension, or context.

Default configuration
  From config/default.json.

Preset data
  From references/presets/ or definitions/.

Step artifacts
  From runs/{keyword}/stepNN-*/.
```

Every key variable should be traceable to a source.

## 3. Naming Rules

Recommended:

```text
target_path
target_dir
input_type
output_format
run_dir
current_step
keyword
mode
```

Rules:

- Use lowercase.
- Use underscores.
- Avoid abbreviations.
- Use one variable name per meaning.
- Path variables end in `_path` or `_dir`.

## 4. Placeholder Convention

In documents, you may use:

```text
{target_path}
{run_dir}
{keyword}
{mode}
```

A placeholder must state its source in the same document or a related config.

Not recommended:

```text
{x}
{data}
{thing}
```

## 5. No Variable Arithmetic

Do not write:

```text
{round_num-1}
{version+1}
{step-previous}
```

To reference the previous round or the latest artifact, use a pointer file:

```text
feedback_latest.md
draft_latest.md
report_latest.json
```

## 6. Path Variables

Path variables should state what they are relative to.

```text
skill_dir
  The Skill root directory.

run_dir
  This run's directory.

target_path
  The user-specified target.

output_dir
  The final output directory, usually {run_dir}/output.
```

Public examples use placeholder paths:

```text
/path/to/your/project
/path/to/input.md
```

Do not use the author's real paths.

## 7. Source Annotation

Recommended: record the source in `state/config.json`:

```json
{
  "target_path": "/path/to/project",
  "target_path_source": "user",
  "input_type": "directory",
  "input_type_source": "inferred_from_path",
  "output_format": "markdown",
  "output_format_source": "default"
}
```

## 8. Missing Variables

Handling rules:

```text
Required variable missing
  Stop and ask.

Inferable variable missing
  Try to infer from context.

Defaultable variable missing
  Use config/default.json.

Optional variable missing
  Skip, and record the default behavior.
```

## 9. Template Variables

Template variables should have a default or a failure strategy.

```text
{{ title }}
  Required. Error if missing.

{{ summary }}
  Optional. Do not render the section if missing.

{{ generated_at }}
  Defaults to the current run time.
```

## 10. Prohibitions

- Variables with no source note.
- Multiple names for one variable.
- Using variable arithmetic to represent the previous round.
- Using real paths in public examples.
- Passing credential values into templates as variables.
- Hiding complex logic inside a variable name.

## 11. Checklist

```text
Naming
  [ ] Variable names are clear.
  [ ] Path variables have an explicit suffix.
  [ ] No duplicate names for one meaning.

Source
  [ ] Required variables have a source.
  [ ] Inferred variables have a source annotation.
  [ ] Defaultable variables have a default config.

Security
  [ ] No real paths.
  [ ] No credential values.
  [ ] No variable arithmetic.
```
