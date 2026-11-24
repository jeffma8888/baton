# baton — vision (v0, pre-research draft)

> Working codename. Final name is a Phase-0 research deliverable (trademark + handle availability).
> Status: HYPOTHESIS v0.1 (thesis corrected 2026-11-23 after R1). Tested by `docs/research/R1..R3`.

## The one-paragraph thesis

Teams increasingly write software by running **many coding agents in parallel** (Claude Code,
Codex, Cursor, OpenCode, Gemini CLI, ...) against isolated copies of a repo, then reviewing and
merging their work. The product category that made this usable, the **conductor**, was defined
by Conductor (YC S24; vendor-claimed 100K+ builders). On 2026-11-04 Conductor announced it was
joining Meta and shutting its cloud; on 2026-11-12/13 it reversed course and now continues as a
proprietary, Meta-owned product with no self-hosted option and visibly damaged user trust
(`docs/research/R1-market-landscape.md` §1-2). Users scattered to a dozen free OSS desktop tools;
every agent vendor ships its own orchestration surface, mostly single-vendor (GitHub Agent HQ is
the cross-vendor exception); and companies that need a conductor they can **run themselves,
with their own agents, repos, sandboxes, policy gates and audit** still build one from parts.

**baton is an open, self-hostable conductor platform that a company can onboard in an hour
and customize as their own**: bring your agents, your repos, your sandboxes, your review gates
and your branding; expose it to your teams, and, if the demand proves real, to your customers.

## Who it is for (hypotheses; R1 confidence in brackets)

1. **Platform / DevEx teams** at companies that need self-hosting, SSO, audit, budgets and
   per-repo policy for agent work, across more than one agent vendor. [R1: real, MED]
2. **Engineering teams** (10-500 engineers) who outgrew ad-hoc worktree scripts or a single
   vendor's desktop and want team-shared, policy-gated agent fleets. [R1: real but contested
   by strong free OSS desktop tools, MED]
3. **Startups building agent products** who would embed a conductor for THEIR customers
   (B2B2B). [R1: no primary evidence yet, LOW; kept as a later option, not the wedge]

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
