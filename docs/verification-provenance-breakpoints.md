# Verification, Provenance, and Breakpoints

Three optional patterns that strengthen the handoff protocol for complex builds. All three are adaptations of directions proposed in the ICM paper (Van Clief & McDermott, 2026), Section 6 "Future Directions: Compilation, Debugging, and Source Integrity."

ICM names these as speculative at the time of publication. This document ports them from that paper's compilation framing into concrete, copy-pasteable additions to a handoff-protocol brief.

None of these are mandatory. Use them when a phase is complex enough to benefit. Skip them when the phase is tight and the base brief sections already cover it.

---

## 1. The `Verify` section (cross-phase trace verification)

**Problem.** A worker executes a phase correctly against its brief, but the output drifts silently from an assumption made in an earlier phase. The drift does not surface until the human review at the final phase, by which point two or three more phases have built on the drifted output.

**Pattern.** Add a `Verify` section to the phase brief, before `Task`. The Verify section names earlier phase outputs the worker should cross-check for consistency with the current phase's brief, and the specific criteria to check.

```markdown
## Verify (before Task)

Before starting the task, check:

- `HANDOFF.md` phase-3 "migration plan" notes still describes the integration
  point this phase will touch. If the integration point has moved, stop and
  escalate.
- `output/phase-4/` contains a `dispatch.py` exporting `dispatch_v2`. If not,
  phase 4 did not land what phase 5 was briefed to build on. Escalate.

If any check fails, set status to `blocked` in HANDOFF.md with a note
pointing at the inconsistency, and exit without running Task.
```

**What it buys you.** Catches drift before work compounds on it. The cost is a few seconds of worker time at the start of each phase. The saving is not rewinding three phases when the drift is discovered at the final review.

**When to use it.** Any phase that reads more than one prior phase's output. Any phase that implements a decision made two or more phases upstream. Any phase where a silent mismatch would not surface until the final review.

**When to skip it.** Phases that read only the immediately prior phase's output, where the handoff doc's own update ritual already surfaces the relevant consistency.

---

## 2. Decision IDs (output provenance)

**Problem.** Three weeks after a build ships, you want to know why phase 4 chose approach X instead of approach Y. The handoff doc says "chose X, see escalation answer." You have to hunt across escalation answers, brief versions, and review notes to reconstruct why.

**Pattern.** Every recorded decision in the handoff doc carries a short identifier and a link to the source that produced it. The source is one of:

- A specific line in the phase brief
- A specific escalation answer in `.questions/phase-N.md`
- A specific review gate note from the orchestrator

```markdown
### Phase 4: Implement executor tier module

- **Status:** done
- **Result sha:** a3f7b2c
- **Decisions:**
  - `D-4-1`: chose `dispatch_v2` over `dispatch_v1_compat` signature.
    Source: `.questions/phase-5.md` answer (2026-04-17 14:22), cited in
    phase-5 brief line 18.
  - `D-4-2`: put the tier lookup table in `executor/tiers.py` rather than
    in `executor/__init__.py`. Source: phase-4 brief line 12 ("keep
    __init__.py empty for forward compat").
```

**What it buys you.** Post-mortem is reading one doc. Regression analysis has explicit provenance. If a future phase wants to revisit a decision, it can cite the decision ID in its own brief and the orchestrator knows exactly what is being reopened.

**When to use it.** Any phase that makes a non-trivial choice between two viable alternatives. Any phase where an escalation happened. Any phase whose output will be referenced by phases more than two hops downstream.

**When to skip it.** Mechanical phases (rename, test-add, doc-update) that produce no decisions worth citing later.

This is the handoff-protocol port of ICM Section 6.2a (output provenance through identifiers). ICM proposes this at the sub-paragraph level of stage outputs. The handoff-protocol version is coarser: decisions per phase, recorded in the handoff doc. Finer-grained provenance inside stage outputs is a compatible extension.

---

## 3. Breakpoints (in-phase checkpoints)

**Problem.** A phase involves one complex judgment call partway through (a schema design, an API shape, an architectural choice), then a lot of mechanical work after the call. Splitting the phase into two creates ceremony (two briefs, two dispatches). Keeping it as one phase means the orchestrator cannot review the judgment until the phase finishes, by which point the mechanical work is already done.

**Pattern.** Add a breakpoint section to the brief. The worker executes up to the breakpoint, writes its intended approach to a checkpoint file, and optionally waits (or continues with a note in the handoff doc that the orchestrator can veto at the next pass).

```markdown
## Breakpoint (optional)

After completing the schema design but before implementing the endpoint:

1. Write your chosen schema plus rationale to
   `.checkpoints/phase-6-schema.md`.
2. Add a note to HANDOFF.md phase-6 saying "breakpoint hit, schema written."
3. Mode: `continue` (continue executing, orchestrator reviews the
   checkpoint asynchronously and can redirect before the review gate) OR
   `wait` (stop here, set status to `blocked`, exit until orchestrator
   releases).
```

**What it buys you.** A single complex phase becomes a sequence of verifiable sub-steps without the overhead of a full phase split. The orchestrator can review the hard part of the phase while the worker grinds through the easy part.

**When to use it.** Phases where a single judgment point would otherwise force a split. Phases with known decision-cliff moments. Phases where the cost of the wrong choice mid-phase is high enough to warrant review but low enough that pausing is wasteful.

**When to skip it.** Phases with multiple judgment points (split them instead). Mechanical phases (nothing to break on). Phases short enough that the full handoff doubles as the checkpoint.

This is the handoff-protocol port of ICM Section 6.2c (breakpoints in markdown). ICM describes it as a sub-step verifier inside a single session. The handoff-protocol version treats the checkpoint file as a minimal handoff artifact that the orchestrator can review without stopping the worker.

---

## Composition

The three patterns compose. A complex phase might use all three:

- `Verify` at the top: confirm upstream phase outputs still match what this brief was written against.
- Decision IDs in the handoff doc update at the end: record the non-trivial choices made during the phase.
- `Breakpoint` mid-phase: surface a hard judgment point for asynchronous orchestrator review before the mechanical tail of the phase runs.

A mechanical phase that renames files might use none of them.

Match the weight of the pattern to the weight of the phase. Ceremony that exceeds the phase's complexity is waste.

---

## Relationship to ICM Section 6

These three patterns correspond to ICM's Section 6.2 (Toward Semantic Debugging):

| Handoff protocol | ICM Section 6 direction |
|---|---|
| `Verify` section in brief | 6.2b Cross-stage trace verification |
| Decision IDs in handoff doc | 6.2a Output provenance through identifiers |
| Breakpoints in brief | 6.2c Breakpoints in markdown |

ICM Section 6.1 (multi-pass incremental compilation) is reflected in the main [protocol.md](protocol.md) under "Incremental re-dispatch." ICM Section 6.3 (edit-source principle) is reflected in protocol rule 8 ("Recurring errors are debugging information about the brief, not the phase").

Read the ICM paper for the compiler-engineering analogy that motivates all of these. This doc extracts the practical forms for a session-handoff setting.
