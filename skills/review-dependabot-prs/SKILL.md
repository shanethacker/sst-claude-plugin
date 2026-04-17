---
name: review-dependabot-prs
description: >
  This skill should be used when the user mentions "dependabot", "dependency updates",
  "review dependency PRs", "check dependabot PRs", or asks about open PRs from Dependabot.
  Reviews open Dependabot pull requests for compatibility, urgency, breaking changes, and
  CI/CD status, producing a structured recommendation report. Default reviews all open
  Dependabot PRs; accepts specific PR numbers as arguments.
disable-model-invocation: true
---

# Review Dependabot PRs

Analyze Dependabot PRs and produce an evidence-based recommendation report. Do NOT merge or close PRs — only report.

## Arguments

- No args: review all open Dependabot PRs
- PR numbers (e.g., `1755 1754`): review only those PRs

## Workflow

### Phase 1 — Discover PRs

```bash
# All open dependabot PRs (default)
gh pr list --author "app/dependabot" --state open --json number,title,headBranch,labels,createdAt,url

# Or fetch specific PRs
gh pr view <number> --json number,title,headBranch,labels,createdAt,url,body,statusCheckRollup
```

If no open Dependabot PRs exist, report that and stop.

### Phase 2 — Parallel research (use subagents)

Launch one **Agent subagent per PR** (all in parallel). Each subagent receives the full context below and investigates a single PR.

#### Per-PR subagent instructions

For each PR, the subagent must gather and return ALL of the following:

**A. PR & diff details**
- `gh pr view <number> --json body,files,commits,statusCheckRollup`
- `gh pr diff <number>` — read the actual diff to see what changed
- Identify: package name, old version, new version, ecosystem and directory

**B. Lockfile check**
- If the diff modifies a dependency manifest (`pyproject.toml`, `package.json`, `Cargo.toml`, etc.), check whether the corresponding lockfile was also updated in the same PR
- Examine the repo to determine which lockfile format is in use (`requirements.txt`, `uv.lock`, `poetry.lock`, `package-lock.json`, `yarn.lock`, `pnpm-lock.yaml`, `Cargo.lock`, etc.)
- If the lockfile was NOT updated, flag this — the consumer may need to regenerate it before merging

**C. CI/CD status** (critical — do not skip)
- From `statusCheckRollup`, report each check's name, status, and conclusion
- If any checks are still running, say so — do not speculate on outcome
- If checks failed, fetch failure logs: `gh run view <run-id> --log-failed` and summarize

**D. Dependency research** (use Context7 MCP and/or WebSearch for current info)
- Search for the package's changelog / release notes between old and new version
- Identify: bug fixes, new features, deprecations, **breaking changes**
- Check for CVEs or security advisories fixed in this range
- Check if the new version is the latest stable release or a pre-release
- Note any runtime or framework compatibility requirements

**E. Cross-dependency compatibility**
- Read the project's dependency manifest to find other packages that interact with this one
- Flag any version constraints that might conflict
- Check if other pinned packages have known incompatibilities with the new version

**F. Codebase impact scan**
- Search the codebase for imports of the package and usage of any APIs marked as changed/deprecated in the changelog
- If breaking changes exist, estimate the scope of code changes needed (files, lines, complexity)

**G. Confidence and evidence**
- Label each statement as one of: **Confirmed** (verified from docs/changelog), **Likely** (strong indirect evidence), or **Speculative** (reasonable inference, not verified)

### Phase 3 — Cross-PR compatibility analysis

After all subagents return, analyze **interactions between PRs**:
- Do any PRs update packages that depend on each other?
- Could merging one PR affect the viability of another?
- Is there a recommended merge order?
- Are there groups that should be merged together or kept separate?

### Phase 4 — Generate report

Produce a single structured report. Use this format for each PR:

```
## PR #<number>: <title>
**Package**: <name> | **From**: <old> | **To**: <new> | **Ecosystem**: <dir>

### Lockfile sync
⚠️ **Lockfile not updated** — manifest changed but lockfile was not updated in this PR
(or: ✅ Lockfile updated in PR / N/A — no manifest change)

### CI/CD Status
<status of each check — pass/fail/running>

### What changed (old → new)
- <key changes grouped: security fixes, bug fixes, features, breaking changes>

### Security
- <CVEs fixed, severity, whether the vulnerability affects this project's usage>
- If none: "No known security advisories in this version range"

### Compatibility
- Runtime: <compatible? evidence>
- Framework: <compatible? evidence>
- Cross-dependencies: <any conflicts with other pinned packages>

### Codebase impact
- <imports/usage found, any API changes affecting the codebase>
- Effort estimate: None / Low / Medium / High (with explanation)

### Recommendation
**Action**: Merge / Merge with caution / Hold / Close
**Urgency**: Critical (security) / High / Normal / Low
**Reasoning**: <1-3 sentences>
**Confidence**: <Confirmed / Likely / Speculative — and why>
```

After all individual PR sections, add:

```
## Cross-PR Analysis
- <dependency interactions between PRs>
- <recommended merge order, if any>
- <groups that should be merged together>

## Summary Table
| PR | Package | Action | Urgency | CI | Lockfile | Effort | Confidence |
|----|---------|--------|---------|----|----------|--------|------------|

## Manual testing suggestions
<If any PR warrants manual testing, describe what to look for>
```

## Important rules

- **Thoroughness over speed**: Use `model: "opus"` for subagents. Research deeply.
- **Evidence first**: Always cite where information was found (changelog URL, package registry page, GitHub release, etc.)
- **Ask when unsure**: If ambiguity exists (e.g., unclear if a breaking change affects this project), stop and ask rather than guessing silently.
- **No merging**: Produce a report. The human decides what to merge.
- **Mark confidence levels**: Every claim about compatibility or safety must be tagged Confirmed/Likely/Speculative.

## Additional Resources

- **`references/research-guide.md`** — Where to find changelogs, security advisories, compatibility information, and effective search strategies for dependency research
