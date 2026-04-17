---
name: explain-pr
description: >
  Produce a comprehensive markdown document explaining all changes in a pull request,
  branch, or commit range — covering the architecture involved, what each change does
  and why, and how the pieces fit together. Use this skill whenever the user asks to
  "explain this PR", "walk me through these changes", "document what we did in this
  branch", "create a PR explainer", "turn this branch into documentation", or any
  request to understand or document a set of changes as a cohesive narrative. Also use
  it proactively when a branch is ready for review and a written walkthrough would
  help reviewers understand the intent behind the changes.
---

# PR / Branch Explainer

Your job is to produce a document that lets someone who wasn't in the room understand
*what* changed, *why*, and *how* — not just a list of diffs.

Reference skill: the `sst:explain-code` skill in this plugin.
Read it when you need techniques for explaining individual non-obvious code sections
(analogy, ASCII diagram, step-by-step walkthrough, gotcha callout). Use those
techniques selectively — for genuinely complex or surprising sections, not as
boilerplate on every code block.

---

## Step 1: Identify the scope

Determine what to explain. Possibilities:
- **Current branch vs. main**: `git log HEAD --not main --oneline --reverse`
- **Named branch**: `git log <branch> --not main --oneline --reverse`
- **PR number**: `gh pr view <N> --json commits` to get the commits
- **Explicit commit range**: as given by the user

If "the current branch" or "what we just did" is used, default to current branch vs.
`main`. If ambiguous, ask before proceeding.

Filter out noise: dependency bumps and merge commits rarely need narrative explanation.
Note them briefly ("two dependency updates were included but are not covered here") and
move on.

---

## Step 2: Gather commit data

For the remaining commits, run these in sequence per commit:

1. `git show <hash> --stat` — understand which files changed and how much
2. `git show <hash>` — read the full diff, including the commit message carefully.
   The commit message often contains the *why*, which is more valuable than the *what*.

Build a mental model of each commit's purpose before moving to the next.

---

## Step 3: Explore the surrounding architecture

Diffs alone are not enough. You need to understand what the changed code fits *into*.
For each significant file changed:

- Read its current state (not just the diff) to understand the full context
- Read related files that give architectural context: settings, base classes, routers,
  managers, helpers, tests
- Use Grep/Glob to find how changed functions or classes are called elsewhere

The goal is to be able to explain *relationships*, not just changes. A diff that adds
a method to a base class only makes sense once you understand what that class does and
where it sits in the broader system.

---

## Step 4: Assess the audience

If the user stated their familiarity level, use it. Otherwise default to:
- Familiar with the application's domain and data model
- May be less familiar with the specific frameworks or subsystems involved

Don't over-explain application domain knowledge. Do explain framework-specific behavior
(e.g., "Django's router chain is a veto system — each router can abstain by returning
None, or veto by returning True/False"). When in doubt, explain the framework concept
once and then reference it by name afterward.

---

## Step 5: Write the document

Save to `working_docs/<topic-slug>-explainer.md`.

Use this structure, adapting section names to fit the actual content:

```
# [Topic]: A Complete Walkthrough

(Brief preamble: branch, date range, author)

## Background: Why This Work Was Needed
The problem or friction that existed before these changes. What was hard,
broken, or missing? Why did it matter?

## Architecture Overview
The relevant parts of the system that these changes touch — models, routers,
settings, helpers, APIs — with their relationships explained. A table or
ASCII diagram works well here. Explain things the reader needs to know to
follow the commit-by-commit section.

## The Changes, In Order
One section per commit or logical group. For each:

### Commit N: <hash> — <short description>
**<full commit title>**

- What was broken/missing and what this commit fixes/adds
- The specific code changes, explained in context (not just repeated as a diff)
- Why this approach — what alternatives existed and why this one was chosen

Use sst:explain-code techniques here when code is non-obvious:
  - An analogy when a concept is unfamiliar
  - An ASCII diagram when flows or relationships are complex
  - Step-by-step walkthrough for subtle logic
  - A "gotcha" callout for common misconceptions the code avoids

## How It All Fits Together
An end-to-end narrative or flow showing how the pieces interact after all
the changes are in place. This is where you answer "ok but what actually
happens when X does Y?" A flow diagram is often useful here.

## What Changed, By File
A summary table:
| File | What changed |
|---|---|
| path/to/file.ext | One-line summary |

## Key Design Decisions and Tradeoffs
Non-obvious choices and their rationale. For each decision:
- What was chosen
- What alternatives existed
- Why this approach was preferred (performance, safety, simplicity, constraint)

## Quick Reference
How to use, run, or test the new behavior. Commands, flags, workflows.
```

---

## Quality bar

The document should stand alone. Someone who wasn't in the room should be able to read
it and understand:

- Why the changes were made (not just what changed)
- How the changed code works in context
- What tradeoffs were made
- How to use or test the result

Avoid:
- Summarizing diffs without explaining them
- Repeating commit messages verbatim without unpacking them
- Leaving "why" unanswered
- Burying the key insight under too much preamble
