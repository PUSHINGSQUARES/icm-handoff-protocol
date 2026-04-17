# The Handoff Protocol

## The problem

Every AI workflow longer than one session has the same failure mode.

You run a brilliant first session. The agent understands the goal, the constraints, the trade-offs. You end the session. A week later, you open a new one to continue. The new session is starting from zero. You paste your notes, your last few messages, a summary of what you did. The context compression is lossy. The new agent makes a small, wrong assumption. You do not catch it. Two sessions later the wrong assumption has compounded into a bad build.

This is the seam. Every long workflow lives or dies at the seam.

## The insight

The conversation is not the state. The conversation is the runtime. If you let the runtime carry the state, you lose the state the moment the runtime ends.

The handoff protocol moves the state out of the conversation and into a file. A specific kind of file, with a specific shape, that any fresh session can pick up cold and execute against.

## The rules

### 1. The handoff doc is the single source of truth

One file per multi-phase build. Call it `HANDOFF.md`, `PLAN.md`, whatever. Location at the workspace root.

The doc contains:

- The goal (one paragraph)
- The phases (ordered, with clear boundaries)
- The acceptance criteria per phase
- The current status per phase (pending, in progress, done, blocked)
- The boot command a worker runs on first action to enter the correct working directory and load the right context
- The dispatch command a worker runs at exit to trigger the next phase

If a fact about the build is not in this doc, it does not exist. Sessions do not pass state to each other by any other channel.

### 2. One worker, one phase

Each phase is sized for one fresh session to complete in one sitting. If a phase is too big, split it.

"Too big" usually means one of three things:

- The phase requires more than one major architectural decision
- The phase requires reading more than roughly ten files to understand
- The phase requires more than one full build and test cycle

If any of those are true, split the phase.

### 3. Briefs are self-contained

Each phase has its own brief. The brief is what the worker reads on first action. It is written for a reader who has never met you and has no conversation history.

A brief contains:

- The one-line task
- The cd command to enter the correct working directory
- The two or three bullets of context the worker needs to make judgment calls
- The exact files to read or modify
- The acceptance criteria
- The exact command the worker runs to hand off to the next phase when done

See [writing-briefs.md](writing-briefs.md) for the full shape, and [verification-provenance-breakpoints.md](verification-provenance-breakpoints.md) for the optional sections a brief can carry when a phase benefits from them.

### 4. Every worker reads, updates, and hands off

Three mandatory actions for every worker:

- **Read** the handoff doc as the first action. Not the second. The first.
- **Update** the handoff doc as the last action before exit. Status changes, decisions made, blockers found, all recorded.
- **Hand off** by calling the next phase's dispatch command, unless the phase is a review gate or the doc explicitly says stop.

### 5. The orchestrator reviews at phase boundaries only

The human does not sit in the loop between phases. The human reviews when a phase lands, decides whether to approve, reject, or redirect, and releases the next phase.

Phase boundaries are where judgment lives. Inside a phase, workers execute. At boundaries, humans review. That separation is what keeps humans in high-value work.

### 6. Executors can be cheap, judgment has to be expensive

A worker executing a well-written brief does not need to be a premium model. A cheaper model with good instructions outperforms a premium model on a vague task every time.

Reserve the premium tier for:

- Writing the handoff doc
- Writing the phase briefs
- Reviewing phase outputs
- Judgment calls escalated by a worker mid-phase

See [executor-tiers.md](executor-tiers.md) for the tiering pattern.

### 7. Workers escalate rather than guess

A worker that hits a decision it cannot make from the brief alone does not proceed. It writes a question to a known location and exits. The human or the orchestrator answers. The phase is re-dispatched with the answer in the brief.

This is slower in the short term and much faster in the long term, because you do not spend two phases unwinding a worker's wrong guess.

### 8. Recurring errors are debugging information about the brief, not the phase

If the same kind of worker mistake appears in two or more phases, the fix does not go in the handoff doc or the acceptance criteria of the failing phase. The fix goes upstream: into the brief template, the protocol itself, or the reference material the briefs cite.

Editing a phase's output to correct a recurring pattern is patching the binary. The source is the brief that produced the pattern. Fix the source and every future run is correct.

This rule is the handoff-protocol analog of ICM's edit-source principle (Section 6.3 of the ICM paper). See [verification-provenance-breakpoints.md](verification-provenance-breakpoints.md) for how to surface these patterns before they compound.

## How it composes with ICM

ICM (the Interpretable Context Methodology: Van Clief & McDermott, 2026) already gives you:

- Folder structure as orchestration logic
- Layered context loading (L0 through L4)
- One stage, one job
- Plain text interface between stages
- Every output as an edit surface

The handoff protocol adds one thing: a sequencing contract for stages that run in **different sessions**, not just different folders.

ICM says "each stage loads only the context it needs." The handoff protocol says "each stage loads that context from a fresh session, with no memory of the previous stage, and still ships."

Together they form a complete system. ICM defines the spatial structure of the workspace. The handoff protocol defines the temporal structure of execution across sessions.

## Incremental re-dispatch

A property the protocol gets for free, worth stating explicitly because it is easy to miss.

Because the handoff doc records what each phase read (its Inputs) and what each phase produced (its output artifacts), it is an implicit dependency graph. This matches the multi-pass incremental compilation pattern ICM describes in Section 6.1 of the paper.

Practical consequence: if phase 5 is wrong and you fix the phase-5 brief, you do not have to re-run phases 1 through 4. You re-dispatch phase 5, and if its output does not invalidate what phase 6 and 7 read, they stay valid. If it does, they get re-dispatched too.

The rule: re-run only the phases whose Inputs changed, or whose upstream Inputs changed. The handoff doc tells you which those are, because the handoff doc records what each phase read.

## When to use this

Use the handoff protocol when:

- Your build is more than a few hours of focused work
- Your build involves multiple distinct concerns (scaffolding, tests, integration, etc.) that naturally split into phases
- You want to run phases overnight or while you are doing something else
- You want to use cheaper models for mechanical phases and reserve premium models for decisions

Do not use it when:

- The task is a single focused session (no seams, no protocol needed)
- The task requires continuous back-and-forth with a human (the human is the orchestrator, no automation possible)
- The task is exploratory and the phases cannot be predicted in advance (write the plan first, then protocol)

For the full list of places this protocol does not work, see the README's "Where this does not work" section.

## What this protocol buys you

- **Resilience.** A crashed session, a compacted context, a moved file: none of these break the build. The next worker reads the doc and continues.
- **Parallelism at the build level.** You can run phase three while reviewing phase two's output, because each phase is independent of the session that ran the previous one.
- **Cost control.** Each phase is scoped, so the premium-tier cost is bounded to the phases that need it. Everything else runs on a cheaper tier.
- **Auditability.** The handoff doc is a ledger. Every phase's decisions, outputs, and blockers are recorded in the one place. Post-mortem is reading one file.
- **Incremental re-dispatch.** Fix one phase, re-run only that phase plus whatever depended on its output. See the section above.

## What it costs you

- **Up-front planning.** You cannot skip the handoff doc. The doc is the contract.
- **Discipline.** Workers that skip the "update the doc" step silently break the chain. Enforce this in the brief, or with a hook.
- **Some flexibility.** You are committing to a plan before execution. Mid-phase pivots require rewriting the doc, not just redirecting a conversation.

That cost is the point. The protocol trades flexibility for predictability. Predictability is what lets you sleep while a build ships.
