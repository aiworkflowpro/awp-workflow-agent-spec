---
name: markdown-release-check
description: Use when the user asks to check an existing Markdown file before publishing. This Skill is not for writing new articles, uploading content, or generating images.
license: MIT
---

# Markdown Pre-Release Check

## Scope

Check an existing Markdown file before publishing.

Do not:

- Write new articles.
- Publish to platforms.
- Generate images.
- Rewrite the topic.
- Read unrelated directories.

## Input

Required:

- Path to a Markdown file.

Optional:

- Target platform.
- Whether the user wants automatic fixes.

## Workflow

1. Confirm the file exists.
2. Read the Markdown file.
3. Check the title, heading levels, frontmatter, links, image references, code fences, and empty sections.
4. Report blocking issues first.
5. If the user asks for fixes, patch only the target file.
6. Re-read the modified file and verify the fixed items.

## Output

Return:

- Blocking issues.
- Non-blocking suggestions.
- Applied fixes, if any.
- Remaining risk.

## Verification

- Every finding references the target file path.
- Code fences are paired and closed.
- Link and image-reference syntax is valid.
- No publish action was performed.
