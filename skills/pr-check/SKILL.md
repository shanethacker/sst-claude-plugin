---
name: pr-check
description: >
  This skill should be used when the user asks to "run pre-PR checks", "check before pushing",
  "run quality checks", "verify the branch is ready", "run lint and tests", or "make sure
  CI will pass". Runs the project's quality gates locally to catch failures before they hit
  the pipeline.
disable-model-invocation: true
---

# Pre-PR Quality Gate

Run the project's quality checks locally to catch issues before pushing. The checks and commands to run depend on the project — discover them before running anything.

## Phase 1 — Discover the project's tooling

Before running any checks, determine what tools the project uses:

**Task runner detection** (check in this order):
- `pyproject.toml` with `[tool.poe.tasks]` → use `poe <task>`
- `Makefile` → use `make <target>`
- `justfile` → use `just <target>`
- `package.json` `scripts` → use `npm run <script>` or `yarn <script>`
- No task runner → invoke tools directly

**Quality tools to look for**:
- **Linting**: `ruff`, `flake8`, `pylint` (Python); `eslint` (JS/TS); `clippy` (Rust)
- **Formatting**: `ruff format`, `black`, `isort` (Python); `prettier` (JS/TS)
- **Type checking**: `mypy`, `pyright` (Python); `tsc` (TypeScript)
- **Testing**: `pytest` (Python); `jest`, `vitest` (JS/TS); `cargo test` (Rust)
- **Template/markup linting**: project-specific (check README or CLAUDE.md)

Read `CLAUDE.md` and the task runner config to identify the exact commands used in CI.

## Phase 2 — Identify changed files

Detect the default branch, then diff against it:

```bash
# Detect default branch
git remote show origin | grep 'HEAD branch' | awk '{print $NF}'
# Fall back to: git symbolic-ref refs/remotes/origin/HEAD | sed 's@^refs/remotes/origin/@@'
```

```bash
git diff --name-only <default-branch>...HEAD
```

Use the changed file list to:
- Determine which apps/modules are affected
- Decide which tests are relevant to run
- Skip checks that don't apply (e.g., skip HTML template linting if no `.html` files changed)

## Phase 3 — Run checks sequentially

Stop at the first failure to save time. For each check:

### a. Lint
Run the project's lint command. If auto-fixable issues are found, fix them and report what changed.

### b. Format check
Run the project's format check. If issues are found, run the formatter and report.

### c. Type check
Run the project's type checker. Report errors found in changed files. Pre-existing errors in unchanged files may be noted but not treated as blockers if they were present before this branch.

### d. Relevant tests
Determine which test files correspond to the changed source files and run them with fail-fast and quiet flags:
- pytest: `pytest tests/... -x -q`
- vitest: `vitest run --bail 1`
- jest: `jest --bail 1 --silent`
- cargo: `cargo test -- --quiet`

Note any tests that require external services (databases, caches, APIs) and flag them separately.

## Phase 4 — Report results

Provide a summary:
- ✅ or ❌ for each check
- Details on any failures with suggested fixes
- If external service tests were skipped, note what they require
- If all checks pass, confirm the branch is ready for PR

## Common issues

- If the task runner isn't available, fall back to direct tool invocation
- If tests require a database, note that and suggest starting it via the project's task runner or `docker compose up <service>`
- If tests require a cache/queue, note and suggest starting it
- If type errors exist in unchanged pre-existing code, note them but don't block on them
