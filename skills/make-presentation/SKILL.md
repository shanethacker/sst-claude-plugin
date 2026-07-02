---
name: make-presentation
description: >
  This skill should be used when the user asks to "create a presentation", "build a deck",
  "make a tech talk", "draft slides on X", "write a presentation about", or wants to develop
  technical presentation content iteratively — architecture/coding decisions for peer
  developers, security or technical plans for leadership, or talks for a community of
  practice. Produces research-backed, cited content following the Assertion-Evidence
  framework as a markdown file by default, with optional PowerPoint export. Supports
  building from scratch or improving an existing outline, and continuing a draft across
  multiple sessions.
disable-model-invocation: true
---

# Make Presentation

Produce technical presentation content: a markdown deck with per-slide claims, evidence,
visual suggestions, and citations — built from scratch or from an existing outline,
reviewed by a fresh pair of eyes, and refined iteratively across sessions.

## Arguments

- No args: start a new presentation, or continue one if you point at its file when asked
- A file path:
  - If the file has `created_by: sst:make-presentation` in frontmatter, treat it as a continuation — jump to Phase 5 (Refine)
  - Otherwise, ask whether the file should be used as an **input outline** (read it as Phase 1 Q4) or as the **output destination** for a new deck

## Phase 0 — Detect new vs. continuing

If given a file path argument, read it.
- If its frontmatter has `created_by: sst:make-presentation`, this is a continuation — skip Phase 1 (its answers are already in the frontmatter) and go to Phase 5.
- Otherwise, ask whether to treat it as an outline input or as the output destination, then proceed to Phase 1 with that choice applied.

If no argument, ask: "New presentation, or continuing one you already started?" If continuing,
ask for the file path, then proceed as above. Otherwise proceed to Phase 1.

## Phase 1 — Intake (new presentations only)

Ask these up front, before any research, one at a time:

1. **Topic** — what is this presentation about?
2. **Audience** — `peer-dev` (fellow developers, architecture/implementation detail welcome),
   `leadership` (business/risk impact framing, minimal implementation detail), or `community`
   (community of practice — broader background-knowledge range, more context-setting)
3. **Duration** — target minutes. Use ~1.5–2 minutes/slide for technical talks as a rough
   guide (e.g. 20 min ≈ 10–13 slides) — state the resulting estimate and let the user adjust
4. **Existing outline?** — a file path to read, or pasted content. If none, this skill builds
   the outline from scratch in Phase 2
5. **Output destination** — where to save the markdown file. Suggest a sensible default (e.g.
   `presentations/<topic-slug>/deck.md` under the current directory) but don't assume it

Record the answers in the output file's frontmatter (Phase 4) so later sessions don't need to
re-ask them.

## Phase 2 — Outline

Build (or critique-and-restructure a provided outline into) the Assertion-Evidence arc:

**context → problem → options considered → recommendation → next steps**

Each planned slide gets a working headline stated as a full-sentence claim, not a topic label
("Caching cut p95 latency 40%", not "Caching Results"). If an outline was provided, use it as
a guide, not a script — flag where it doesn't already fit this arc rather than silently
reshaping it, and say so when presenting the outline back.

Tailor depth and framing to the audience from Phase 1 — not tone, see Voice below:
- `peer-dev`: keep implementation tradeoffs and alternatives considered
- `leadership`: lead every section with business/risk impact; compress or cut implementation detail
- `community`: add a beat of context/background before diving in, since prior familiarity varies

Show the outline to the user before drafting full slides. Wait for a go-ahead or changes.

## Phase 3 — Research

`deep-research` is a bundled Claude Code workflow (`/deep-research <question>`), not a skill —
invoke it via the Workflow tool with `name: "deep-research"`. It ships with Claude Code itself,
so no separate install is needed.

For every claim that needs current information or a citation (statistics, version numbers,
comparisons, security posture, "as of" statements), invoke it with a tightly scoped question per
claim or cluster of related claims — don't dump the whole topic on it at once. Workflows can
fan out to many subagents and spend real tokens per call, so batch related claims into one
well-scoped question rather than firing a separate call per sentence. Fold cited findings back
into the outline as evidence for the relevant slide.

If bundled workflows are unavailable in this environment (e.g. `disableBundledSkills` is set),
say so, then fall back to scoped WebSearch/WebFetch per claim — still one tightly scoped
question at a time, still requiring a citation before a claim goes on a slide. Label findings
gathered this way as unverified by deep-research's adversarial-check pass, so the user knows
the bar was lower for that claim.

Don't re-research claims that are the user's own first-hand experience or internal decisions
(e.g. "we chose X because Y") — those don't need external sourcing.

## Phase 4 — Draft

Write the markdown file. Structure, per slide:

```markdown
## Slide N — <Full-sentence claim>

<Minimal supporting text — bullets, not paragraphs. One idea.>

**Visual:** <description of the suggested diagram/chart/screenshot and why it supports the claim>

**Speaker notes:** <optional — what you'd say aloud that isn't on the slide>

Sources: [<label>](<url>), [<label>](<url>)
```

File frontmatter (enables Phase 0 continuation detection and skips re-asking intake):

```yaml
---
title: <topic, as it should read on the title slide>
created_by: sst:make-presentation
topic: <topic>
audience: peer-dev | leadership | community
duration_minutes: <n>
---
```

`title` is separate from `topic` because pandoc reads a markdown file's `title:` frontmatter
field to generate the title slide on export (Phase 6) — `topic` is this skill's own intake
field and may not always read naturally as a slide title verbatim.

**Voice — work-casual, constant across all three audiences** (this is a fixed default; only
deviate if the user explicitly asks for a different register for this specific presentation):
- Contractions are fine ("we're," "don't")
- Plain verbs over corporate jargon — no "leverage," "synergy," "move the needle," "circle back,"
  "utilize" (say "use")
- Direct claims, not hedged into mush ("this cuts p95 latency 40%," not "this may potentially
  help improve latency in some cases")
- Active voice, short sentences
- Dry wit or an apt analogy is welcome if it aids clarity; skip anything needing setup/punchline
  timing you can't guarantee in someone else's meeting room
- Confidence without hype — no exclamation-point enthusiasm, no "amazing"/"exciting" standing in
  for an actual claim
- Casual voice, not casual accuracy — the Assertion-Evidence rigor doesn't loosen with tone

Save the file to the destination from Phase 1 (or the existing file, if continuing).

## Phase 5 — Refine (fresh or continuing)

Automatically dispatch the `presentation-reviewer` agent against the current draft — every
draft or revision pass gets a critique before control returns to the user. Present the draft
and the reviewer's findings together.

The reviewer only reports findings; it does not apply fixes. Ask the user which findings to
act on, then:
- Content/structure changes → back to Phase 2 or 4 as appropriate
- New or updated factual claims → back to Phase 3 for that claim, then re-draft
- No further changes → done for this session; the file on disk is the continuation point next time

On a later invocation pointed at this file, skip straight to this phase: read the draft, ask
what to refine (or re-run research on a specific claim, or address outstanding reviewer
findings), make the change, then automatically re-run the reviewer.

## Phase 6 — Optional PowerPoint export

Only on explicit request. Requires `pandoc` (check with `pandoc --version`; if missing, say so
and stop rather than guessing at an alternative).

**The authoring markdown is not export-ready as-is.** `**Visual:**` and `Sources:` lines are
notes to the presenter, not slide content — piped straight through pandoc they become literal
bullet text on the slide. `**Speaker notes:**` needs to land in the PowerPoint notes pane, which
requires pandoc's `::: notes` fenced-div syntax — bold text does not route there on its own.

Before converting, write a transformed copy (e.g. `<deck>.export.md`, discard after conversion
or keep alongside the source — don't overwrite the authoring file):
- Drop `**Visual:**` lines from slide body text — they're production notes for whoever builds
  the actual visual, not slide content. If the user wants a placeholder on the slide itself
  (e.g. `![](visual-placeholder.png)`), do that explicitly instead of leaving the raw description
- Convert `**Speaker notes:** ...` into:
  ```markdown
  ::: notes
  ...
  :::
  ```
- Drop `Sources:` lines from slide body, or move them into speaker notes (`::: notes`) if the
  user wants citations available while presenting but not on-screen — ask which

Then convert the transformed copy:

```bash
pandoc <deck>.export.md -o <deck>.pptx
```

If the user has an org slide template, use it so the output inherits its styling instead of
pandoc's generic default:

```bash
pandoc <deck>.export.md -o <deck>.pptx --reference-doc <org-template>.pptx
```

**Before handing back the .pptx, open/inspect the generated output** (or at minimum re-read the
transformed markdown) and confirm no `**Visual:**`/`Sources:` text leaked onto a slide — don't
assume the transform was clean without checking.

Markdown stays the source of truth — regenerate the export from the authoring file rather than
hand-editing the pptx and letting the two drift.
