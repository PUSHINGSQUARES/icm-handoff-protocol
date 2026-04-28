# ICM Handoff Protocol

A protocol layer on top of the **Interpretable Context Methodology (ICM)** for running long, multi-phase AI workflows across fresh, stateless worker sessions.

> If your workflow dies the first time a session ends, you do not have a workflow. You have a preference.

This repo is for builders who have already hit the wall where a capable AI agent does great work for one session, then loses all context the moment the window closes. The first worker was brilliant. The second worker has no idea what just happened. You end up copy-pasting the last ten messages into a new chat to remember what you were doing.

The handoff protocol fixes that seam.

## What this is

A concrete, copy-pasteable pattern for:

1. **Splitting a large build into phases**, each completable by a fresh worker with no prior context.
2. **Writing briefs** that a stateless session can read once and execute without asking a clarifying question.
3. **A living handoff doc** that is both the contract and the memory. Every worker reads it on boot and updates it on exit.
4. **Chained dispatch**, where each phase launches the next. No human in the execution loop between phases. The human reviews at phase boundaries.
5. **Executor tiers**, where cheap models handle mechanical work and only escalate to premium models when judgment is required.
6. **Optional verification, provenance, and breakpoint patterns** ported from ICM's Section 6 research directions into concrete brief-template sections.

## What this is not

- A framework. There is no runtime to install. This is markdown plus a convention.
- A replacement for ICM. It builds on top. Read the canonical ICM repo first: https://github.com/RinDig/Interpreted-Context-Methdology
- A real-time multi-agent system. Phases run sequentially. Each phase is one worker. Each worker is one job.

## Why it works

Most "AI orchestration" is really one long conversation pretending to be a system. The conversation is the state. When it ends, the state ends.

This flips that. The handoff doc is the state. The conversation is disposable. Any worker, any model, any session can pick up the work cold, because the doc is always the source of truth.

That property is what makes the pattern survive at scale. You can run eight phases overnight across eight fresh sessions and wake up to a shipped branch, because no phase depends on anyone remembering anything.

## Relationship to ICM

ICM (Van Clief & McDermott, arXiv:2603.16021v2) defines the **spatial** structure of an AI workspace: folder hierarchy, five-layer context model, stage contracts, and the edit-surface property. Its canonical execution model is a single agent moving across stages within one session, with human review at stage boundaries.

This protocol extends ICM along the **temporal** axis: what happens when stages run in *different* sessions, possibly on different models, over hours or days, with no conversational memory carrying across. The handoff doc becomes the medium that carries state across that seam.

The two compose directly. An ICM workspace that uses this protocol gains session-independence for long builds. The handoff protocol without ICM is less useful, because it assumes the spatial scaffolding ICM provides (per-stage context, plain-text interface, editable intermediate outputs).

## Where this does not work

This protocol inherits ICM's limits and adds its own:

- **Real-time worker collaboration.** Workers do not talk to each other. The only channel between phases is the handoff doc. If your problem requires two agents in tight-loop dialogue, use an agent framework.
- **Concurrent phases.** Phases are sequential. Work inside a phase can parallelize (spawn sub-agents, split file edits), but phase N does not start until phase N-1's handoff is complete.
- **Automated mid-phase branching.** A phase that needs to pick between two paths based on its own mid-execution output should either split into two phases with a review gate between them, or escalate to the orchestrator. Trying to automate that branching moves this protocol toward being a framework.
- **Exploratory tasks with unpredictable phase shape.** Write the plan first, then use the protocol. The protocol does not help you discover the phases.

## Quickstart

1. Read [docs/protocol.md](docs/protocol.md) for the full spec.
2. Copy [templates/handoff-doc.md](templates/handoff-doc.md) into your workspace as `HANDOFF.md`.
3. Fill in the phases, the acceptance criteria, and the worker boot command.
4. Copy [templates/phase-brief.md](templates/phase-brief.md) per phase into `.briefs/` or wherever your dispatcher reads from.
5. Launch phase one. Review the update to the handoff doc when it lands. Launch phase two. Repeat.

A worked example lives in [examples/worked-example.md](examples/worked-example.md).

## Documentation

- [The protocol spec](docs/protocol.md) — the actual rules
- [Writing briefs](docs/writing-briefs.md) — how to write for a reader who has never met you
- [Executor tiers](docs/executor-tiers.md) — cheap model + escalation pattern
- [Verification, provenance, breakpoints](docs/verification-provenance-breakpoints.md) — optional patterns ported from ICM Section 6
- [Worked example](examples/worked-example.md) — a full eight-phase build, anonymized

## Status

This is the distilled version of a pattern that has shipped real production work. It is opinionated. It trades flexibility for predictability. That is the point.

## Citation and credit

ICM is the upstream this repo sits on. Credit goes to its authors:

> Van Clief, J., and McDermott, D. (2026). Interpretable Context Methodology: Folder Structure as Agent Architecture. arXiv:2603.16021v2. https://arxiv.org/abs/2603.16021

The concepts in [docs/verification-provenance-breakpoints.md](docs/verification-provenance-breakpoints.md) are direct adaptations of the future-directions section of that paper to the session-handoff problem.

## License

MIT. See [LICENSE](LICENSE).
