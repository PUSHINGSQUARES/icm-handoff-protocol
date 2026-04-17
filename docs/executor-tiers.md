# Executor Tiers

A handoff protocol works because each phase is scoped, contained, and briefed. That scoping is also what lets you run phases on models cheaper than the one that designed them.

## The core idea

The orchestrator (premium tier) designs the build, writes the handoff doc, and writes the phase briefs. This is the expensive work. This is where judgment lives.

The workers (cheaper tiers) execute phases against briefs. This is the mechanical work. It is the majority of tokens spent in the build. It should not be paid at premium rates.

## A four-tier pattern

Concrete tiering varies by provider. The pattern is consistent.

**Tier 1: Orchestrator.** The most capable model you have access to. Used for: writing the handoff doc, writing briefs, reviewing phase outputs, answering escalations. Cost per token is high. Token count is low because the work is mostly planning and reviewing.

**Tier 2: Mid-capability executor.** A strong general model that can handle novel code, make local design decisions inside a brief's constraints, and write tests. Used for: most implementation phases. The workhorse tier.

**Tier 3: Cheap executor.** A fast, cheap model. Used for: mechanical phases where the brief removes all ambiguity. Boilerplate generation. File splitting. Rename passes. Running and interpreting test output. Anything the brief specifies so tightly that creativity is not required.

**Tier 4: Read-only reviewer.** A cheaper model, often with write tools stripped at the harness level. Used for: reading a phase's output and flagging concerns. Cannot mutate, so the worst case is a false alarm, not a broken build.

## How to pick a tier per phase

Ask two questions per phase:

**Q1. What is the worst outcome if the worker makes a bad judgment call?**

If the answer is "a broken build I catch in review," tier 2 or tier 3 is fine.

If the answer is "a subtle architectural choice that compounds across the rest of the build," tier 1 or tier 2.

If the answer is "nothing, because the worker cannot mutate anything," tier 4.

**Q2. How tightly does the brief constrain the worker?**

Very tightly constrained (every file named, every acceptance criterion binary): tier 3 works.

Moderately constrained (the worker has to decide a few things inside the phase): tier 2.

Loosely constrained (the phase itself is design work): tier 1.

The tighter the brief, the cheaper the tier you can get away with. This is another reason to write tight briefs. It is not just for correctness. It is for cost.

## The escalation path

A worker on any tier can escalate. The brief tells them how.

```markdown
## Escalation

If you hit a decision the brief does not cover, do not guess. Write the
question to `.questions/<phase>.md` with:

- The decision point
- The options you considered
- What you would do by default

Then exit cleanly. The orchestrator will answer and re-dispatch.
```

A cheap-tier worker that escalates every five decisions is still cheaper than a premium-tier worker that answers all five inline, because the escalations are short and the work between them is long. The math favors cheap plus escalate.

## What this buys you

- **Cost bounded by difficulty.** The build's total cost tracks the hardness of the design work, not the volume of the code written.
- **Surface area for review.** Every cheap-tier phase produces a diff. You or the orchestrator can glance at it before dispatching the next phase. You catch bad work before it compounds.
- **Stability across provider changes.** The protocol does not depend on a specific vendor's pricing. When a new tier becomes cheap, you move phases down into it.

## Common mistakes

### Running every phase on the top tier "to be safe"

This is paying for insurance you do not need. A top-tier model on a tightly-scoped brief is doing the same work as a cheap-tier model on the same brief. The output is often indistinguishable. You are burning budget for a feeling of safety.

Fix: tier down one level for mechanical phases and see if the output quality actually drops. Often it does not.

### Running the orchestrator on a cheap tier to save money

The orchestrator's cost is already low (planning is few tokens). The orchestrator's leverage is high (one bad handoff doc ruins the whole build). This is false economy.

Fix: pay for the orchestrator. Save on the workers.

### No escalation path

A cheap-tier worker that cannot escalate will guess. Guesses compound. By phase four the build is off course and nobody noticed because every phase individually "succeeded."

Fix: every brief has an escalation section. Every worker knows how to pause the build.

## The principle

Capability is not the question. Leverage is the question. A cheap model on a clear brief with an escalation path beats a premium model on a vague task every time, because the cheap model is doing narrower work with a wider safety net.

Write the brief like the next model is half as capable. Your workflow will survive model swaps, provider changes, and the thing that always happens eventually, which is a model you trusted getting quietly worse.
