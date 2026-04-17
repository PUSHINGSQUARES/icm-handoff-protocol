# Writing Briefs for Stateless Workers

A brief is the single document a fresh worker reads on first action to execute a phase. The reader has no conversation history. The reader has never met you. The reader has exactly the text in the brief plus the files the brief points to.

Write accordingly.

## The shape

Every brief has the same six required sections, in this order, plus two optional sections for complex phases.

```markdown
# Phase <N>: <one-line name>

## Boot

<exact cd command>

## Verify (optional)

<cross-phase consistency checks to run before Task>

## Task

<one line>

## Context

- <bullet>
- <bullet>
- <bullet>

## Files

- <path>: <what to do with it>
- <path>: <what to do with it>

## Acceptance

- <binary check the worker runs to know it's done>
- <binary check>

## Breakpoint (optional)

<pause-and-checkpoint markers within this phase>

## Escalation

<how to pause the phase when the brief is not enough>

## Handoff

<exact command to trigger next phase, or "stop and report">
```

The optional `Verify` and `Breakpoint` sections earn their weight when a phase is complex. Skip them when the base sections already cover the phase. Full guidance in [verification-provenance-breakpoints.md](verification-provenance-breakpoints.md).

## Rules for each section

### Boot

Give the exact command. Not a description. The command.

Bad: "Enter the worktree for this phase."
Good: `cd /absolute/path/to/workspace/worktrees/phase-3-integration`

A worker that has to infer the path will infer wrong sometimes. A worker that copy-pastes a command will not.

### Verify (optional)

Cross-phase consistency checks to run before Task. Each check names an earlier phase's output or a section of the handoff doc that this brief depends on, and a binary test for whether that dependency is still what the brief was written against.

Bad: "Make sure earlier phases are still relevant."
Good:
- `HANDOFF.md` phase-3 "migration plan" still names `executor_v2.py` as the module to implement. If the module name changed, stop and escalate.
- `output/phase-4/` contains `dispatch_v2`. If not, phase 4 drifted. Escalate.

If any check fails, the worker does not proceed to Task. It sets status to `blocked` and exits. The brief that named a stale dependency needs updating before the phase can run.

Use when a phase reads more than one upstream output, implements a decision made two or more phases upstream, or when a silent mismatch would not surface until the final review.

### Task

One line. Imperative. No hedging.

Bad: "We should probably look at adding some tests to the executor layer."
Good: "Add unit tests for the executor dispatch function covering all three tier cases."

If you cannot write the task in one line, the phase is too big. Split it.

### Context

Two or three bullets. The minimum the worker needs to make judgment calls without asking.

Include:

- Non-obvious constraints the worker cannot see from the code
- Recent decisions that shape what counts as a good solution
- Traps known to exist in this area

Exclude:

- Things the worker will discover by reading the code
- Backstory that does not affect execution
- Your feelings about the build

If the bullet does not change what the worker does, cut it.

### Files

Every file the worker will touch, with a verb. Not "here are some relevant files." A directive.

Bad:
- `src/executor.py` (the executor)
- `tests/test_executor.py` (tests)

Good:
- `src/executor.py`: read, do not modify
- `tests/test_executor.py`: add new test class `TestTierDispatch`

If the worker might need to read a file for context but not modify it, say so. Ambiguity here causes scope creep.

### Acceptance

Binary, runnable checks. The worker should be able to run them and get a yes or a no.

Bad: "Make sure the tests pass and the code looks good."
Good:
- `pytest tests/test_executor.py` returns 0 exit code
- `ruff check src/executor.py` returns 0 warnings
- New test class `TestTierDispatch` has at least 3 test methods

If the acceptance criteria are not runnable, the worker cannot know they are done. They will either over-shoot or under-shoot. Both are failure modes.

### Breakpoint (optional)

One mid-phase pause point for a complex judgment call. Worker writes its intended approach to a checkpoint file, notes it in the handoff doc, and either continues (orchestrator reviews asynchronously) or waits (worker exits until released).

Use when a phase has a single decision cliff followed by mechanical work, where splitting into two phases is overkill but waiting until the final review is too late.

### Escalation

Every brief has one. Every one has either a `.questions/` file path or an explicit alternative location. Every one describes the format: decision point, options considered, default.

Non-negotiable. A worker that hits ambiguity without an escalation path will guess, and guesses compound.

### Handoff

The exact command the worker runs to trigger the next phase. Or the exact instruction for what to do instead (stop, wait for review, escalate).

Bad: "When done, move to the next phase."
Good: `./dispatch.sh phase-4-migration --prev-sha $(git rev-parse HEAD)`

The worker should not have to infer the handoff command any more than the boot command.

The Handoff section also reminds the worker to log any non-trivial decisions to the handoff doc's decision log with IDs (D-N-1, D-N-2, ...) and source citations. This is the provenance pattern. See [verification-provenance-breakpoints.md](verification-provenance-breakpoints.md).

## Common failure patterns

### Briefs that assume a reader who already knows

The most common failure. You write the brief after a long thinking session. You have all the context loaded. You forget that the worker does not.

Fix: after drafting the brief, read it as if you are a stranger. If any bullet requires knowledge the brief does not provide, add that knowledge or cut the bullet.

### Briefs that overflow with context

The second most common failure. You overcompensate for the first failure and dump everything you know into the brief. The worker reads five thousand words of context before reaching the task.

Fix: the brief is not documentation. It is a contract. Anything that does not directly change what the worker does is noise. Move it to a referenced file if needed. Cut it otherwise.

### Vague acceptance criteria

The third most common. "Make sure it works" is not acceptance criteria. Workers cannot verify subjective outcomes.

Fix: every acceptance line must be runnable or visually checkable (for UI work). If you cannot make it binary, split the phase until you can.

### Missing handoff

The worker finishes, has no idea what to do, exits. The chain breaks. You discover this the next morning.

Fix: every brief has a handoff section. Every handoff section has either a command or an explicit stop instruction. Never imply.

### Silent cross-phase drift

The fifth and subtlest failure. The brief is clear, the acceptance criteria are binary, the worker ships. But a phase two phases back has quietly changed in a way the current brief assumed would not happen. The output is internally consistent and externally wrong.

Fix: add a `Verify` section to the brief for any phase that depends on multiple upstream outputs or on decisions made more than one phase back.

## The self-check

Before launching a phase, read the brief out loud. Ask yourself:

1. If I had never seen this project, could I execute this brief?
2. If I hit a surprise, does the brief tell me what to do?
3. When I am done, does the brief tell me how to know?
4. When I am done, does the brief tell me what to do next?
5. If an upstream phase has drifted since this brief was written, will this brief catch it?

If any answer is no, the brief is not ready. Fix it before you dispatch.

## The brief is the test

A well-written brief is proof that you understand the phase well enough to delegate it. If you cannot write the brief, you do not understand the phase yet. Do that thinking first. The brief is the output of that thinking.

This is why the handoff protocol makes workflows better even when no one is handing off. The discipline of writing the brief forces you to make the phase concrete. Concrete phases ship. Vague phases rot.
