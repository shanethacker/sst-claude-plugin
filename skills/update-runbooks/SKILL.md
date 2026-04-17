---
name: update-runbooks
description: >
  This skill should be used when the user asks to "update runbooks", "audit runbooks",
  "check if runbooks are accurate", "verify operational docs", "runbook drift", or
  "sync runbooks with the code". Verifies that operational runbooks reflect the current
  state of the codebase and updates anything that has drifted.
disable-model-invocation: true
---

# Update Runbooks

Audit operational runbooks against the current codebase and update anything that has drifted. The goal is to leave each runbook accurate — not exhaustive — so operators can rely on it in production.

## Arguments

The user may optionally specify:
- A specific runbook file to audit
- A lookback window for git history (e.g., "last month", "3 months"); defaults to 3 months if not specified

If no arguments are given, audit all runbooks listed in `CLAUDE.md` (or the project's equivalent documentation index).

## Phase 1 — Identify runbooks

Read `CLAUDE.md` (or the project's primary documentation index) to find the list of operational runbooks and their locations. If no index exists, search for runbook files:

```bash
rg --files -g "*runbook*" -g "*operations*"
```

## Phase 2 — For each runbook, verify accuracy

Read the runbook and check each factual claim against the current code:

### a. Line number references
For every `file:line` reference in the runbook, read that file and verify the line still points to the described code. If lines have shifted, update to the correct line number. If the same reference drifts repeatedly across audits, flag it as a candidate for replacement with a more stable anchor (function name, section heading, or grep-able string) rather than just re-pointing the line number each time.

### b. Environment variables and settings
For every env var or config setting mentioned, grep the codebase to verify:
- The env var still exists
- It maps to the same setting or behavior
- The documented default is still accurate

### c. Code behavior
For described behaviors (e.g., "retries 3 times and logs a warning"), read the referenced code and confirm it still works that way. Pay special attention to error handling, fallback logic, and feature flags.

### d. File paths
Verify all referenced files still exist at the stated paths. Check for renamed or moved files.

### e. Architecture claims
Verify described data flows and component relationships still hold. Check for new components that should be documented.

## Phase 3 — Check for undocumented changes

Run git log to surface recent changes to files the runbook covers:

```bash
git log --oneline --since="<lookback window from arguments, default: 3 months ago>" -- <paths covered by runbook>
```

Review commits that might introduce new operational knowledge not yet captured. Pay attention to:
- New environment variables added
- Changed retry/timeout behavior
- New queues, caches, or external services integrated
- Removed or deprecated functionality still described in the runbook

## Phase 4 — Apply updates

- Fix inaccuracies directly in the runbook
- Add new sections for undocumented operational knowledge discovered
- Remove documentation for features or config that no longer exist
- Preserve the existing structure and tone — don't rewrite, update

## Phase 5 — Report

Summarize what was checked and what changed:
- Number of references verified
- List of corrections made (with before/after where helpful)
- Any new sections added
- Any open questions requiring human input (e.g., "Is this new env var intentional or a WIP?")
- Any areas where the runbook was already accurate and required no changes
