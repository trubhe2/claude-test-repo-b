---
name: brainstorm
description: Iteratively builds an epic PRD for this project through confidence-gated Q&A cycles — creates the PRD if missing, asks multiple-choice open questions grounded in this repo, reconciles real answers into functional/non-functional requirements, user journeys, and acceptance criteria, then dissolves answered questions into a log. Repeats until confidence reaches 90%.
when_to_use: Use when the user runs `/brainstorm <epic name>`, or asks to "brainstorm an epic", "build out a PRD for X", or "flesh out requirements for X".
---

# /brainstorm

## Usage

```
/brainstorm <epic name>
```

Everything typed after `/brainstorm` is the epic name, verbatim. If nothing was given, ask the user for an epic name before doing anything else — do not guess one.

## What this skill does

Builds and iteratively refines a PRD for the named epic through repeated cycles:

1. ask multiple-choice open questions (4 options each) for real, via the user
2. reconcile the actual answers into concrete requirements
3. dissolve the answered questions into a compact log entry
4. re-score confidence
5. repeat until confidence ≥ 90%, then stop

This skill never selects an answer on the user's behalf. Every checked box in the PRD must trace back to something the user actually said.

## Step 0 — Resolve the epic and PRD file

- Slugify the epic name (lowercase; spaces and punctuation → hyphens) to get `<slug>`.
- The PRD lives at `docs/prds/<slug>.md`. Create the `docs/prds/` directory if it doesn't exist yet.
- Check whether that file already exists — this determines whether you're starting a new epic (Step 1) or resuming one (skip straight to Step 2, carrying forward everything already in the file).

## Step 1 — Create the PRD if it doesn't exist

Write `docs/prds/<slug>.md` from this exact template (fill in `<Epic Name>` and today's date; leave every other section as shown):

```markdown
# Epic: <Epic Name>

**Confidence:** 0%
**Status:** Draft — gathering requirements
**Last updated:** <YYYY-MM-DD>

## Summary

<one or two sentence restatement of the epic — grounded only in the epic name and real repo context, nothing invented>

## Functional Requirements

_None yet — resolved from answered questions below._

## Non-Functional Requirements

_None yet — resolved from answered questions below._

## User Journeys

_None yet — resolved from answered questions below._

## Acceptance Criteria

_None yet — resolved from answered questions below._

## Answered Questions Log

_No cycles completed yet._
```

Then continue directly into Step 2 in the same run — don't stop and wait for a re-invocation just because the file was just created.

## Step 2 — Ground the next questions in this actual project

Before writing a single question, gather real signal so the 4 options per question are actually tailored, not generic:

- Re-read the PRD's current state: Summary, whatever is already filled into the four requirement sections, and the full Answered Questions Log — never re-ask something already answered there.
- Look at what actually exists in this repo relevant to the epic: `README.md`, `CLAUDE.md`/`AGENTS.md`, source directories, package/build manifests, other files under `docs/prds/`, and recent `git log`. Use Explore/Grep rather than assuming.
- If this repo genuinely has no established product domain yet (e.g. it's a bare or sandbox project), say so to yourself and ground options in what a reasonable implementation would look like given the epic name plus whatever *does* exist here (existing structure, stated purpose, conventions) — never invent an unrelated product domain to make the options sound more concrete than the repo actually is.

## Step 3 — Ask this cycle's open questions

- Pick 2–4 open questions that most reduce uncertainty about the epic, given what Step 2 turned up and what's already resolved. Each one needs **exactly 4 options**.
- Ask them for real using the `AskUserQuestion` tool (batch multiple questions into one call; split into more calls only if you have more than 4 questions this cycle):
  - Put your recommended option **first** in the list and suffix its label with `(Recommended)`.
  - Do not pre-select or assume an answer — wait for the tool's actual result.
- In parallel, append the same questions to the PRD file under a new heading `## Open Questions (Cycle <N>)`, one block per question:

  ```markdown
  ### Q1. <question text>

  - [ ] <Option A text> (Recommended)
  - [ ] <Option B text>
  - [ ] <Option C text>
  - [ ] <Option D text>
  ```

  Every box stays unchecked at this point — this is a written record of what was asked, not a place to record a decision yet.

## Step 4 — Reconcile the real answers

Once `AskUserQuestion` returns the user's actual choices:

- For each question, check the box (`[x]`) of the option the user picked. If they picked "Other" / gave free text, replace that option line with their actual words and check it.
- Turn every answer into direct, concrete requirement bullets — never restate the question itself — and file each into the right PRD section (one answer can feed more than one section):
  - **Functional Requirements** — what the system must do
  - **Non-Functional Requirements** — performance, security, reliability, compliance, or other constraints
  - **User Journeys** — the concrete step-by-step flow a user takes
  - **Acceptance Criteria** — testable pass/fail statements

## Step 5 — Dissolve this cycle's questions

- Move this cycle's Q&A out of `## Open Questions (Cycle <N>)` entirely and into `## Answered Questions Log`, newest cycle at the top:

  ```markdown
  ### Cycle <N>

  - **Q:** <question text> → **A:** <the option text the user actually chose>
  ```

- Delete the answered question blocks from the `## Open Questions (Cycle <N>)` section **from the bottom of the list upward** (last question first, then the one above it, and so on) so earlier blocks never have to be re-located while you edit. Once all of that cycle's questions are removed, delete the now-empty `## Open Questions (Cycle <N>)` heading too.
- Between cycles, `## Open Questions` must be empty or absent — only the Answered Questions Log accumulates history.

## Step 6 — Update confidence

- Re-assess overall confidence (0–100%) that the Functional/Non-Functional Requirements, User Journeys, and Acceptance Criteria are complete and unambiguous enough to hand to an engineer. Base the number on real, nameable gaps (missing error-handling behavior, no success metric defined, an unresolved edge case) — not a round-number guess.
- Update `**Confidence:** <N>%` and `**Last updated:**` in the PRD header.

## Step 7 — Loop or stop

- If confidence is still below 90%: go back to Step 3 for another cycle (new questions, next cycle number) in this same session — don't make the user re-run the command.
- If confidence has reached 90% or more: set `**Status:** Ready for review`, stop asking questions, and tell the user the PRD is ready, its final confidence score, and the file path.

## Rules

- Every open question gets exactly 4 options, each prefixed with a markdown checkbox (`- [ ]`), with exactly one marked `(Recommended)`.
- Never check a box or otherwise pick an answer yourself — a box is checked only after the user's real response comes back from `AskUserQuestion`.
- Never write PRD content that isn't traceable to the epic name, real repo context gathered in Step 2, or an answered question.
- Keep `## Open Questions` transient — it must never accumulate past cycles; only the Answered Questions Log is permanent.
