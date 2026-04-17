# HANDOFF.md (template)

<!--
This is the living handoff doc. It is the single source of truth for this build.
Every worker reads it on first action and updates it before exit.
Copy this template to your workspace root and fill it in before dispatching phase 1.
-->

## Goal

<One paragraph. What are we building and why. Plain language. No jargon.>

## Status legend

- `pending` — not started
- `in_progress` — a worker is actively on this phase
- `done` — worker finished, acceptance criteria met, review passed
- `blocked` — worker escalated, waiting on orchestrator
- `failed` — worker hit a stop, needs redesign

## Phases

### Phase 1: <one-line name>

- **Status:** pending
- **Brief:** `.briefs/phase-1.md`
- **Worker tier:** <tier-1 | tier-2 | tier-3>
- **Prev sha:** <populated by worker>
- **Result sha:** <populated by worker>
- **Inputs read:** <populated by worker — list of files and HANDOFF sections actually consumed>
- **Decisions:** <populated by worker — D-1-1, D-1-2, ..., each with a source citation>
- **Notes from worker:** <populated by worker>

### Phase 2: <one-line name>

- **Status:** pending
- **Brief:** `.briefs/phase-2.md`
- **Worker tier:** <tier>
- **Prev sha:** <populated by worker>
- **Result sha:** <populated by worker>
- **Inputs read:** <populated by worker>
- **Decisions:** <populated by worker>
- **Notes from worker:** <populated by worker>

<!-- Repeat per phase. Numbered. Ordered. -->

## Decision log (appendix)

<!--
Cross-reference of every non-trivial decision made during the build. Populated
as phases complete. Each entry has a short ID (D-<phase>-<n>), the decision
itself, and the source citation (brief line, escalation answer, or review note).

This is the handoff-protocol form of output provenance. See
docs/verification-provenance-breakpoints.md.
-->

| ID | Decision | Source |
|---|---|---|
| D-1-1 | <what was chosen> | <brief line / escalation answer / review note> |

## Worker boot instructions

Every worker, on first action, runs:

1. `cd <absolute path to the correct working directory for this phase>` (see brief)
2. Read this file (`HANDOFF.md`) in full.
3. Read the phase brief (`.briefs/phase-N.md`).
4. Run any `Verify` checks the brief specifies. If any fail, set status to `blocked` and exit.
5. Read any files the brief references.
6. Execute.

## Worker exit instructions

Before ending the session, every worker:

1. Commits its work on the correct branch for this phase.
2. Updates this file with: status, result sha, inputs read, decisions (with IDs and sources), notes.
3. Either:
   a. Runs the dispatch command for the next phase (if brief says to chain), OR
   b. Writes a completion note and exits (if brief says to stop for review), OR
   c. If a breakpoint was hit in `wait` mode: writes the checkpoint path to this file and exits.

## Escalation

If a worker hits a decision not covered by its brief, it does **not** guess.

It writes the question to `.questions/phase-N.md` with:

- The decision point
- The options considered
- What it would do by default

Then sets its phase status to `blocked` and exits.

The orchestrator answers, updates the brief, and re-dispatches.

## Review gates

<Optional. Mark phases where human review is required before the next phase runs.>

- After phase 3 (pre-migration): human confirms backup exists before phase 4 runs.
- After phase 6 (cutover): human confirms smoke tests pass before phase 7 runs.

## Incremental re-dispatch

If a phase needs to be re-run (brief changed, escalation answer resolved, review rejected):

1. Re-dispatch that phase only.
2. Check which downstream phases' `Inputs read` included outputs that this phase will regenerate. Those phases' outputs are now potentially stale.
3. Re-dispatch only those downstream phases whose inputs actually changed. Phases whose inputs did not change do not need to re-run.

This works because the `Inputs read` field is an explicit dependency record. See `docs/protocol.md` "Incremental re-dispatch."

## Glossary

<Optional. Any domain-specific term a worker might misread. Keep short.>

- `tier-1`: premium orchestrator model
- `tier-2`: mid-capability executor model
- `tier-3`: cheap executor model
