# MWP Handoff Protocol

A protocol layer on top of the **Model Workspace Protocol (MWP)** for running long, multi-phase AI workflows across fresh, stateless worker sessions.

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

## What this is not

- A framework. There is no runtime to install. This is markdown plus a convention.
- A replacement for MWP. It builds on top of it. Read the canonical MWP repo first: https://github.com/RinDig/Model-Workspace-Protocol-MWP-
- A real-time multi-agent system. Phases run sequentially. Each phase is one worker. Each worker is one job.

## Why it works

Most "AI orchestration" is really one long conversation pretending to be a system. The conversation is the state. When it ends, the state ends.

This flips that. The handoff doc is the state. The conversation is disposable. Any worker, any model, any session can pick up the work cold, because the doc is always the source of truth.

That property is what makes the pattern survive at scale. You can run eight phases overnight across eight fresh sessions and wake up to a shipped branch, because no phase depends on anyone remembering anything.

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
- [Worked example](examples/worked-example.md) — a full eight-phase build, anonymized

## Status

This is the distilled version of a pattern that has shipped real production work. It is opinionated. It trades flexibility for predictability. That is the point.

## License

MIT. See [LICENSE](LICENSE).
