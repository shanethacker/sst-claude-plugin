# sst Plugin

Personal Claude Code toolkit. Namespace: `sst`.

## What's in This Plugin

| Component | Type | Purpose |
|---|---|---|
| `sst:review-dependabot-prs` | Skill | Reviews open Dependabot PRs — compatibility, breaking changes, CI status |
| `sst:pr-check` | Skill | Runs quality gates (lint, format, type check, tests) before a PR |
| `sst:gen-test` | Skill | Generates tests following the project's established conventions |
| `sst:update-runbooks` | Skill | Audits operational runbooks for drift against the current codebase |
| `code-reviewer` | Agent | Reviews code changes for correctness, conventions, and coverage |
| `security-reviewer` | Agent | Reviews code changes for security vulnerabilities |

## Design Principles

**Generalized, not generic.** Skills in this plugin were developed against a specific
Django/Python project and then generalized. They discover project conventions
at runtime rather than assuming a particular stack. Project-specific overrides belong
in the project's own `.claude/` directory.

**Procedural skills use `disable-model-invocation: true`.** The skills that orchestrate
workflows (`review-dependabot-prs`, `pr-check`, `update-runbooks`) set this flag;
`gen-test` does not because it writes output directly.

## Adding New Components

Follow the conventions in the parent directory's `CLAUDE.md` (`../CLAUDE.md`).

When generalizing a project-specific skill:
1. Replace hardcoded tool names, paths, and framework versions with discovery steps
2. Move project-specific content to the project's `.claude/` directory
3. Keep only the workflow pattern in the plugin skill

## Prerequisites

- [`gh`](https://cli.github.com/) — required by `review-dependabot-prs`, `pr-check`, and `update-runbooks`
