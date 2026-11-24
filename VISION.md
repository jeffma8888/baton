# baton — vision (v0, pre-research draft)

> Working codename. Final name is a Phase-0 research deliverable (trademark + handle availability).
> Status: HYPOTHESIS. Everything below is to be tested by `docs/research/R1..R3`.

## The one-paragraph thesis

Teams increasingly write software by running **many coding agents in parallel** (Claude Code,
Codex, Cursor, OpenCode, Gemini CLI, ...) against isolated copies of a repo, then reviewing and
merging their work. The product category that made this usable, the **conductor**, was defined
by Conductor (YC S24; 100K+ builders) — and on 2026-11-04 Conductor announced it joined Meta,
shut its cloud on 2026-11-20, and winds down its local app in early 2027. Meanwhile every agent
vendor is building its *own* vendor-locked conductor, and companies that want one for their own
agents, repos, policies and customers have to build from scratch.

**baton is an open, self-hostable conductor platform that any company can onboard in an hour
and customize as their own**: bring your agents, your repos, your sandboxes, your review gates
and your branding; expose it to your teams or to your customers.

## Who it is for (hypotheses to validate)

1. **Startups building agent products** who need an orchestration surface for their customers
   but do not want to build and operate one (B2B2B: they are the customer, their users are the
   end users).
2. **Engineering teams** (10–500 engineers) who used Conductor or built ad-hoc worktree scripts
   and want a team-shared, auditable, policy-gated way to run agent fleets.
3. **Platform/DevEx teams** at larger companies who need self-hosting, SSO, audit, and
   per-repo policy for agent work.

## What it is NOT (initial non-goals)

- Not another coding agent. baton orchestrates agents; it does not compete with them.
- Not a general workflow engine (Temporal/Netflix-Conductor class). It may *use* one.
- Not an IDE.

## Design principles (to be challenged in HLD-000)

- **Agent-agnostic by contract**: any agent that can run in a sandbox and speak a defined
  protocol (ACP / MCP / CLI adapter) is a first-class citizen.
- **Isolation is the unit of safety**: one workspace = one isolated checkout (+ optional
  sandbox); nothing reaches a shared branch without passing a gate.
- **Policy as configuration**: review gates, budgets, allowed tools, and merge rules are
  declarative, per-org / per-repo, and auditable.
- **Self-host first, hosted later**: a single-binary / single-compose install is the product.
- **Customizable, not forkable**: branding, roles, gates and adapters are extension points,
  so customers upgrade with upstream instead of diverging.

## How this repo works

- `docs/research/` — primary-source research (Phase 0). Every claim carries a URL.
- `docs/design/` — HLDs then LLDs, in the format in `docs/design/TEMPLATE-HLD.md`.
  Every design doc lists ALL options, labels objections BLOCKING vs ADVISORY, and recommends.
- `docs/plans/` — implementation plans derived from approved designs (milestones, stories,
  verification).
- `docs/process/` — how the project is run (brief for agent teammates, review protocol).

Design is deliberately heavy up front: nothing is implemented until its HLD is approved and
its LLD + plan exist.
