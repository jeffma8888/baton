# baton (codename) - vision

> Working codename; `baton` is not usable as a product name (R3 §E).
> Status: DRAFT v1 (awaiting review). Supersedes HYPOTHESIS v0.1 (git keeps v0.1). Inputs: R1 APPROVED v2, R3 APPROVED v4, R2 DRAFT v2; claims resting on R2 alone are marked (R2 v2, re-verification pending).

## The one-paragraph thesis

Teams increasingly run many coding agents in parallel against isolated copies of a repo, then review and merge their work, and every agent vendor ships a surface for its own agent (R1 §3a). The product that defined the conductor category, Conductor, is alive, proprietary and vendor-owned: it announced on 2026-11-04 that it was joining Meta and winding down, reversed the shutdown on 2026-11-12/13, and its team says it builds inside Meta (deal terms UNVERIFIED); it has no shipped self-hosted option, a GitHub-only cloud and an unsandboxed local mode (R1 §1; R3 §4). GitHub Agent HQ is the strongest cross-vendor incumbent: multi-vendor, bundled into Copilot, with enterprise policies, audit logging and self-hosted runners, inside one code host's cloud (R1 §5; R3 §C). Free OSS desktop tools contest the team segment (R1 §3b, §5). The gap, as R1 and R3 state it, is narrower than "nothing exists": a self-hosted control plane that is cross-vendor and open in its core; declarative per-repo gates and audit available in the open edition; and a headless API comparable to Conductor's (R1 Analysis; R3 §C).

**baton is an open, self-hostable conductor platform that a company can onboard in an hour (hypothesis, MED; open question 3) and customize as its own**: bring your agents, repos, sandboxes, review gates and branding; run it inside your own boundary for your teams and, if demand proves real, your customers.

## What we learned in Phase 0

- The incumbent is still shipping; "stranded Conductor users" is not a wedge and carries a BLOCKING con (R1 §1, §2, Recommendations O2a).
- Not every vendor surface is single-vendor: Agent HQ hosts Claude and Codex in public preview; Claude and Cursor offer customer-side execution, and Cursor's agent loop stays in the vendor cloud (R1 §3a, §5).
- "Companies build from scratch" is only partly true: Stripe, Ramp and Spotify built in-house; Coder and Ona sell governed agent platforms, Coder's governance in paid tiers (R1 §3c, §4; R3 §4).
- The local conductor is commoditised (20+ OSS tools, a three-day rebuild, two monetisation failures); teams still pay $50 to $80 per seat, and the most-cited losses were Cloud, API and multi-machine, not the UI (R1 §2, §3b, §5; R3 §3).
- Checkout separation is not execution isolation: worktrees share host network and credentials; containers share the kernel; microVMs are stronger, not proof of zero exfiltration (R2 §1; R2 v2, re-verification pending).

## Who it is for

R1 confidence in brackets.

1. **Platform / DevEx teams** needing self-hosting, SSO, audit, budgets and per-repo policy across more than one agent vendor. [R1 §5: real, MED] The payer: every comparable monetises here, but the named platform teams built rather than bought (R3 §A).
2. **Engineering teams** (10 to 500 engineers) who outgrew worktree scripts or one vendor's desktop and want shared, policy-gated fleets. [R1 §5: real but contested, MED] The user and adoption funnel; free OSS tools and vendor bundles cap the price (R1 Recommendations O2b; R3 §A).
3. **B2B2B embedders**, split per R3 §A: 3a, agent-product startups whose product is the orchestration UX, not a launch audience because the surface is their differentiator [MED, absence of evidence]; 3b, embedders for whom the conductor is not the differentiator, a later channel tested by direct conversation [R1 §5; R3 §A: LOW].

## What it is NOT

- Not another coding agent; baton orchestrates agents and does not compete with them.
- Not a general workflow engine (Temporal, Orkes class); it may use one as a substrate (R1 §3d).
- Not an IDE.
- Not a dev-environment platform or sandbox provider; those are providers underneath (R3 §A, Recommendations).
- Not a hosted-first cloud and not "a better Conductor UI"; the UI is commoditised, the investment is the policy, gate, audit and adapter layer (R3 §C, §D).

## Strategy

**Licence.** Apache-2.0 core plus a commercial, source-available `enterprise/` module. FSL, BSL with a competing-use grant, ELv2 and Sustainable Use are BLOCKING for the core because they forbid a customer offering the software to its own customers; SSPL is BLOCKING because it is not OSI-approved (R3 §B). Gates, budgets, basic audit and all adapters are core; SSO/SCIM, multi-org RBAC, audit export, HA, air-gap, branding and SLA are Enterprise (R3 §B). DCO, no CLA (R3 §F, Handoffs).

**Wedge and sequence.** OSS self-hosted team server with a thin CLI/desktop client first; hosted second; headless and white-label API last; hosted-first and closed-first are eliminated by BLOCKING items (R3 §D). Pricing: Enterprise per conducting user per year; hosted compute as sandbox-hours plus a platform fee; never meter self-hosted by usage (R3 §B, Recommendations).

**Architecture stance.** Build option B (team server, self-hosted runners, worktree and container executors, Postgres) inside option C's provider-neutral boundaries. A (local daemon, no shared server) and D (vendor-hosted control plane) are eliminated by BLOCKING cons; E (extend an existing OSS server or integrate Coder by API) survives and is spiked against B in HLD-000. First slice: two agents, two executors, at most two Git hosts (R2 options analysis and verdict; R2 v2, re-verification pending).

**Naming status.** Leading candidate `cuelist`; weak second `kapell`. Unchecked: USPTO, EUIPO and UKIPO clearance, and whether `.io` suffices. Reserve the winner the day it clears (R3 §E, Open questions).

## Design principles

Each v0.1 principle was challenged against R2 and restated; two were added; none was dropped. Every R2 citation in this section is R2 v2, re-verification pending.

- **Agent-agnostic by contract** (kept, restated). Adapters normalize one capability set; ACP, MCP and A2A are complementary formats, none a job specification; every adapter supports API-key and enterprise billing (R2 §2, Analysis; R3 §C).
- **One run, one checkout; isolation is a separate, labelled choice** (was "isolation is the unit of safety"; split because checkout separation is not execution isolation). Unsandboxed worktrees are `trusted-local`; a sandboxed backend serves lower-trust work and never mounts host checkouts or credentials (R2 §1, Analysis).
- **Gates are SHA-bound and fail closed** (added; tool permission and change approval are different gates, and ACP transports only the former). Approvals bind to the candidate SHA and invalidate on change; execution identities cannot self-approve or merge (R2 Analysis, Recommendations).
- **Policy as configuration, audit as evidence** (kept, extended). Gates, budgets, allowed tools and merge rules are declarative per org and repo; the audit log is append-only and reconstructable, not a mutable activity feed; both ship in the open edition (R2 Analysis; R3 §B).
- **Self-host first, hosted later** (kept, made honest). The team server is one product binary plus Postgres in one Compose file, not an infrastructure-free binary (R2 Analysis; R3 §D).
- **The open core runs the defining workflow end to end** (added; an open conductor that needs a paid module for task -> run -> diff -> gate -> PR is not open). Core includes two adapters, one local and one container backend, single-org roles and auditable events (R2 Analysis; R3 §B).
- **Customizable, not forkable** (kept, restated). Branding, roles, gates, adapters and providers are versioned extension points; untrusted plugins never run in the server; nobody forks the UI to rebrand (R2 §4, Analysis).

## How we will know

**The payer kill test** (R3 §G item 1; runs before any HLD is approved, R3 Recommendations): 10 to 15 calls with platform/DevEx leads, at least half at organisations running an in-house agent harness, against a two-page spec of the community/enterprise split. PASS: at least 3 sign a non-binding design-partner letter AND at least 2 name a budget line. FAIL: re-scope to an OSS team server with no Enterprise plan, or stop.

**Leading indicators** after an OSS launch, on opt-in heartbeat instances with the opt-in rate stated (R3 §G): activation, at least 40% of instances complete a first successful agent run within 24 hours; agent-agnostic validation, at least 20% of active instances use two or more adapters; team adoption, at least 50 instances with 3 or more users and one configured gate in week 4.

## Open questions

1. Will a Stripe-, Ramp- or Spotify-class platform team buy rather than build? Kill-test report (R3 §G item 1), feeding PRD-000.
2. Will Anthropic, OpenAI, Google and Cursor confirm in writing that third-party conductors may use subscription auth? Written vendor check (R3 §G item 7); adapters assume API-key billing regardless.
3. Which two agents and two Git hosts do the first design partners need, and what does "onboard in an hour" include? PRD-000 (R2 Open questions).
4. What trust tier do pilot workloads need, so is the default a hardened container or a microVM? HLD-000 (R2 Open questions).
5. B or E: build the runner plumbing, or integrate an existing OSS daemon or Coder by API? Spike recorded in HLD-000 (R2 verdict).
6. Will Agent HQ support non-GitHub hosts or a customer-hosted control plane; when does Conductor self-host ship? R1 updates; PRD-000 positioning (R3 Open questions).
7. Is there a 3b embedder willing to sign a letter of intent? R3 §G item 4; the tenancy HLD stays latent until then.
8. Trademark clearance for `cuelist` and `kapell`? The maintainer's search, recorded against R3 §E.

## How this repo works

- `docs/research/` holds Phase 0 primary-source research; every claim carries a URL.
- `docs/design/` holds the PRD, then HLDs, then LLDs, per `docs/design/TEMPLATE-HLD.md`; all options listed, objections tagged BLOCKING or ADVISORY, verdicts from BLOCKING items alone.
- `docs/plans/` holds implementation plans; nothing is written there before HLD-000 is approved.
- `docs/process/` holds the brief, phase plan, review protocol and learnings log.

A different model reviews each deliverable; nothing is implemented until its HLD is approved and its LLD and plan exist.

## Changes from v0.1

- Thesis rewritten: incumbent alive, proprietary and vendor-owned; Agent HQ the strongest cross-vendor incumbent; OSS tools contest the team segment; the gap in R1 and R3's words.
- New sections: What we learned in Phase 0; Strategy; How we will know; Open questions; this list.
- Audience 3 split into 3a and 3b (R3 §A); payer and user roles assigned; "onboard in an hour" labelled a hypothesis.
- Non-goals extended; principles: isolation split into checkout and sandbox, SHA-bound gates and end-to-end open core added, the rest restated against R2; status moved to DRAFT v1.
