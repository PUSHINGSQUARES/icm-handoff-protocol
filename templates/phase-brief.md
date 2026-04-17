# Phase <N>: <one-line name>

<!--
This brief is read by a worker with no conversation history.
Every required section is load-bearing. Fill them all.
Optional sections (Verify, Breakpoint) are for complex phases. Use when they earn their weight.
-->

## Boot

<!-- Exact cd command. Copy-pasteable. -->

```bash
cd <absolute path>
```

## Verify (optional, recommended for phases reading >1 upstream output)

<!--
Cross-phase consistency checks to run BEFORE the task. Names earlier phase
outputs this phase depends on and binary checks to confirm they still match
what this brief was written against.

If any check fails, do NOT proceed to Task. Set status to `blocked` in
HANDOFF.md with a note pointing at the inconsistency, and exit.

See docs/verification-provenance-breakpoints.md for full pattern.
-->

- `<file or HANDOFF.md section>`: <what must be true for this brief to apply>
- `<file>`: <binary check>

## Task

<!-- One line. Imperative. -->

<e.g. "Add unit tests for the executor dispatch function covering all three tier cases.">

## Context

<!-- Two or three bullets. What the worker needs to make local judgment calls. Nothing else. -->

- <non-obvious constraint>
- <recent decision that shapes what "good" means here>
- <known trap in this area>

## Files

<!-- Every file with a verb. "Read", "modify", "create". No ambiguity. -->

- `<path>`: <read | modify | create>. <one line on what.>
- `<path>`: <read | modify | create>. <one line on what.>

## Acceptance

<!-- Binary, runnable, or visually checkable. No subjective language. -->

- `<command>` returns 0 exit code
- `<file>` contains <specific thing>
- <visually checkable outcome, if UI work>

## Breakpoint (optional, for phases with one mid-phase judgment point)

<!--
If this phase has a single complex decision followed by mechanical work, pause
there for async orchestrator review instead of splitting the phase.

Mode `continue`: worker proceeds after writing checkpoint, orchestrator can
redirect at the next review gate.
Mode `wait`: worker stops, sets status `blocked`, exits until released.

See docs/verification-provenance-breakpoints.md for full pattern.
-->

After <specific sub-step>:

1. Write your chosen approach plus rationale to `.checkpoints/phase-<N>-<label>.md`.
2. Add a note to `HANDOFF.md` phase-<N>: "breakpoint hit, <label> written."
3. Mode: `continue` | `wait`

## Escalation

If you hit a decision this brief does not cover, do **not** guess.

Write the question to `.questions/phase-<N>.md`:

```markdown
# Phase <N> question

**Decision point:** <one line>

**Options considered:**
- <option A>
- <option B>

**Default if no answer:** <what you would do>
```

Then set status to `blocked` in `HANDOFF.md` and exit.

## Handoff

<!-- Exact command to trigger next phase, OR explicit stop instruction. No "move to next phase" without the command. -->

When acceptance criteria are met and HANDOFF.md is updated:

- Record any non-trivial decisions made during this phase under phase-<N> in HANDOFF.md with decision IDs (`D-<N>-1`, `D-<N>-2`, ...) citing the source (brief line, escalation answer, or review note). See docs/verification-provenance-breakpoints.md.
- Then run:

```bash
<exact dispatch command for phase N+1>
```

OR (if this is a review gate):

> Stop here. Write a summary to `HANDOFF.md` and exit. Orchestrator reviews before phase N+1 is released.
