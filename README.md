# baton

An open, self-hostable **conductor platform**: run fleets of coding agents (Claude Code, Codex,
Cursor, OpenCode, ...) in isolated workspaces, with team visibility, policy gates and review
flow — and let any company onboard it and make it their own.

**Status: design phase.** No runnable code yet, on purpose. Read in this order:

1. [`VISION.md`](VISION.md) — thesis, audiences, non-goals
2. [`docs/research/`](docs/research/) — market, technical and business research (primary sources)
3. [`docs/design/`](docs/design/) — high- and low-level design docs (options → trade-offs → recommendation)
4. [`docs/plans/`](docs/plans/) — implementation plans derived from approved designs

Every design decision is visible here before it becomes code.

## Why now

Conductor, the product that defined the category, announced on 2026-11-04 that it was joining
Meta and shutting down; on 2026-11-12/13 it reversed and continues as a proprietary Meta-owned
product with no self-hosted option. Its users scattered to free single-user desktop tools; agent
vendors each ship their own, mostly single-vendor, surface. What does not exist is an **open,
agent-agnostic, self-hostable conductor with policy, audit and team features** that a company
can adopt and make its own. See `docs/research/R1-market-landscape.md`.

## License

TBD (decided in `docs/research/R3-business-case.md`).
