# Phase <N>: <one-line name>

<!--
This brief is read by a worker with no conversation history.
Every section is load-bearing. Fill them all.
-->

## Boot

<!-- Exact cd command. Copy-pasteable. -->

```bash
cd <absolute path>
```

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

```bash
<exact dispatch command for phase N+1>
```

OR (if this is a review gate):

> Stop here. Write a summary to `HANDOFF.md` and exit. Orchestrator reviews before phase N+1 is released.
