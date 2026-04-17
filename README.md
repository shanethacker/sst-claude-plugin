# sst — Shane's Claude Code Plugin

Personal Claude Code toolkit providing generalized development workflow skills and agents.

## Components

### Skills

| Skill | Invoke | Description |
|---|---|---|
| `sst:review-dependabot-prs` | `/sst:review-dependabot-prs` | Reviews open Dependabot PRs for compatibility, breaking changes, and CI status. Produces a structured recommendation report. |
| `sst:pr-check` | `/sst:pr-check` | Runs the project's quality gates (lint, format, type check, tests) locally before opening a PR. Detects tooling automatically. |
| `sst:gen-test` | `/sst:gen-test` | Generates tests that match the project's established conventions. Reads existing tests before writing new ones. |
| `sst:update-runbooks` | `/sst:update-runbooks` | Audits operational runbooks against the current codebase and updates anything that has drifted. |

### Agents

| Agent | Triggers | Description |
|---|---|---|
| `code-reviewer` | After completing a feature, bug fix, or implementation step | Reviews code changes for correctness, conventions, cross-module impact, test coverage, and performance. |
| `security-reviewer` | Before opening PRs touching auth, input handling, file uploads, or external API integrations | Reviews code changes for security vulnerabilities across OWASP Top 10 categories. |

## Prerequisites

- [Claude Code](https://claude.ai/code) CLI
- [`gh`](https://cli.github.com/) — required for `review-dependabot-prs`, `pr-check`, and `update-runbooks`

## Installation

**Step 1 — Add the marketplace**

```
/plugin marketplace add shanethacker/sst-claude-plugin
```

This registers the catalog with Claude Code. No plugins are installed yet.

**Step 2 — Install the plugin**

```
/plugin install sst@sst-claude-marketplace
```

**Step 3 — Activate**

```
/reload-plugins
```

## License

MIT
