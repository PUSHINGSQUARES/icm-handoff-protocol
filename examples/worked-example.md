# Worked Example: Rewriting a Background Dispatch Layer

An eight-phase build, run overnight across seven fresh worker sessions, chaining on the handoff protocol. Anonymized. Names and paths changed. The pattern is the point.

## The build

A small tool (an internal dispatch script that spawns long-running background workers) needed a rewrite. It had grown organically. It had concurrency bugs. Its tests were thin. It needed a new executor tier system and a different IPC mechanism for worker questions.

Rough scope: maybe fifteen to twenty hours of focused work. Too large for one session. A natural fit for the handoff protocol.

## The plan (written first, by the orchestrator, premium tier)

The orchestrator (a top-tier model, running in a dedicated session) spent ninety minutes producing:

1. A `HANDOFF.md` at the repo root with eight phases
2. Eight phase briefs in `.briefs/phase-1.md` through `.briefs/phase-8.md`
3. A dispatch command that could launch any phase with a single line
4. An `.questions/` directory for escalations
5. A review gate after phase 3 (migration prep) and after phase 6 (cutover)

No code was written in this session. The output was just the plan.

## The phases

| # | Phase | Tier | Gate after |
|---|-------|------|------------|
| 1 | Add failing tests for new executor tier behavior | tier-2 | no |
| 2 | Scaffold new executor tier module | tier-2 | no |
| 3 | Write migration plan and safety checks | tier-1 | **yes** |
| 4 | Implement executor tier module | tier-2 | no |
| 5 | Wire executor tier into dispatcher | tier-2 | no |
| 6 | Cutover: old path deprecated, new path default | tier-2 | **yes** |
| 7 | Integration tests against live worker spawn | tier-2 | no |
| 8 | Update docs, archive old code, final commit | tier-3 | no |

Each phase had a brief that was roughly two hundred to four hundred words. Each brief was self-contained. Each brief had a boot command, a task, two or three context bullets, a file list, binary acceptance criteria, an escalation section, and a handoff command.

Phases 4, 5, 6, and 7 also included a `Verify` section pointing back at the migration plan from phase 3, because those phases all implemented decisions made there.

## The overnight run

After the orchestrator released phase 3 (having reviewed phase 2's output the previous evening), the remaining chain ran without human involvement until the phase 6 gate.

Here is what actually happened in each worker session:

**Phase 3 worker** (tier-1, because migration design is judgment-heavy) read HANDOFF.md, wrote the migration plan as a new doc in `docs/migration.md`, updated HANDOFF with a summary and decision IDs `D-3-1` through `D-3-4` citing specific brief lines, set status to done with a "review gate" flag, exited. Orchestrator (me, the human, morning review) read the migration plan, approved it, released phase 4.

**Phase 4 worker** (tier-2) read HANDOFF, ran its `Verify` block (confirmed `docs/migration.md` existed and named the executor tier module), read the migration plan, implemented the executor tier module. Acceptance criteria: all new tests from phase 1 pass. They did. The worker committed on a phase branch, updated HANDOFF with the sha, logged decisions `D-4-1` and `D-4-2` (both citing phase-4 brief lines), ran the dispatch command for phase 5.

**Phase 5 worker** (tier-2) wired the module into the dispatcher. Hit a real decision point: two valid signatures for the dispatch function, depending on whether to keep the legacy call path available. Escalated to `.questions/phase-5.md`. Set status to blocked. Exited.

I answered the question in the morning, updated the phase 5 brief to name the chosen signature, re-dispatched. Phase 5 worker (a fresh session, no memory of the blocked session) read the updated brief, ran `Verify`, wired it in, committed, logged `D-5-1` citing the escalation answer timestamp, handed off.

**Phase 6 worker** executed the cutover (removing the old code path in favor of the new default). Acceptance criteria: test suite green, integration test runs against a live worker spawn. The worker ran both. Both passed. The worker set the review gate flag and exited.

Morning: I reviewed the diff, approved.

**Phase 7 worker** (tier-2) added tests for the new integration paths. Used a `Breakpoint` mid-phase: after finalizing the test-case shape but before writing all test bodies, the worker wrote the test-case schema to `.checkpoints/phase-7-test-shape.md` in `continue` mode. I reviewed asynchronously while the worker kept going. The schema was fine, no redirect needed.

**Phase 8 worker** (tier-3, because it was purely mechanical) updated docs, archived the old module, wrote a summary to HANDOFF. Exited clean.

## What went right

- Total wall clock from phase 4 to phase 8: about nine hours. Total focused orchestrator time in that span: about twenty minutes (two review gates, one escalation answer, one asynchronous breakpoint review).
- No session ever lost the plot. Every session read HANDOFF as its first action. Every session knew exactly what it was doing and what was done before it.
- Costs were tier-weighted. The orchestrator tier ran for maybe ten percent of the total token spend. Everything else ran on tier-2 or tier-3.
- The decision log at the bottom of HANDOFF (D-1-1 through D-8-3, roughly fifteen entries) meant the post-mortem two weeks later took ten minutes instead of an hour. Every non-trivial choice had an explicit source.

## What went wrong (and what the protocol did about it)

- **Phase 5 escalation**: the worker hit a decision it could not make from the brief. Instead of guessing, it escalated. This is exactly what the escalation path is for. The cost of the escalation was about ninety seconds of my morning plus a brief edit. The cost of a guess would have been rewinding at least one phase.
- **Phase 7 worker initially misread an acceptance criterion** (ran the wrong test file). The acceptance section was too loose. The worker ran something, got a green, declared done. Morning review caught it. The fix: re-dispatch phase 7 with a tighter acceptance block. Two hours lost, no ambiguity survived.

That second incident triggered rule 8 in the protocol (recurring errors are debugging information about the brief, not the phase). Two prior phases had had similar acceptance-looseness issues. The fix went into the phase-brief template itself: the Acceptance section now explicitly requires runnable commands with expected exit codes. Every future build benefits.

That is the edit-source principle: when the same mistake keeps happening, the fix goes upstream.

## The meta pattern

The dispatch layer being rebuilt was itself a layer that existed to run background workers. The rewrite was executed by background workers handing off to each other via the handoff protocol. The system was the test of the system. When it shipped without a human in the execution loop between phases, that was evidence the protocol survived its own scale.

The lesson: if your handoff protocol can ship a rewrite of the very thing that dispatches the workers running the handoff, it is probably real.

If it cannot, it is a preference.
