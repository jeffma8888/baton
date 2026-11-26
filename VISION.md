# baton (codename) - vision

> Working codename; `baton` is not usable as a product name (R3 §E).
> Status: DRAFT v2 (revised after review, awaiting re-verification). Supersedes HYPOTHESIS v0.1 (in git history). Inputs: R1 APPROVED v2, R2 APPROVED v2, R3 APPROVED v4.

## The one-paragraph thesis

Teams run many coding agents in parallel against isolated copies of a repo, then review and merge, and every agent vendor ships a surface for its own agent (R1 §3a, §4). Conductor, which defined the category, is alive, proprietary and vendor-owned: it announced on 2026-11-04 that it was joining Meta and winding down, reversed course on 2026-11-12/13, and says it builds inside Meta (ownership and deal terms UNVERIFIED); it has no shipped self-hosted option, a GitHub-only cloud and an unsandboxed local mode (R1 §1; R3 §4). GitHub Agent HQ is the strongest cross-vendor incumbent: bundled into Copilot, with enterprise policies, audit logging and self-hosted runners, inside one code host's cloud (R1 §5; R3 §C). Free OSS conductors, several with servers or self-host, contest the team segment and partly occupy the self-hosted slot (R1 §3b, §5; R2 §5; R3 §C). The gap is narrower than "nothing exists": a self-hosted, cross-vendor control plane open in its core; declarative per-repo gates and audit in the open edition; a headless API comparable to Conductor's (R1 Analysis; R3 §C; both MED).

**baton is an open, self-hostable conductor platform that a company can onboard in an hour (hypothesis, LOW: scope undefined, open question 3) and customize as its own**: bring your agents, repos, sandboxes, review gates and branding (Enterprise); run it inside your own boundary for your teams and, later, your customers.

## What we learned in Phase 0

- The incumbent is still shipping; "stranded Conductor users" is not a wedge and carries a BLOCKING con (R1 §1, §2, Recommendations O2a).
- Agent HQ hosts Claude and Codex in public preview; not every vendor surface is single-vendor (R1 §5).
- Claude and Cursor offer customer-side execution; Cursor's agent loop stays in the vendor cloud (R1 §3a, §5).
- "Companies build from scratch" is only partly true: Stripe, Ramp and Spotify built in-house (R1 §3c, §4; R3 §4).
- Coder and Ona sell governed agent platforms, Coder's governance in paid tiers; Ona is closed to new customers (R1 §4; R2 §4; R3 §4).
- The local conductor is commoditised: 20+ OSS tools, a three-day rebuild, two monetisation failures (R1 §2, §3b, Analysis).
- Teams still pay: Conductor Teams and Paradev Team $60 per user; Devin Teams $80 a month flat for up to 100 users (R1 §5; R3 §3).
- Checkout separation is not execution isolation: worktrees share host network and credentials; containers share the kernel; microVMs are stronger, not proof of zero exfiltration (R2 §1).

## Who it is for

1. **Platform / DevEx teams** needing self-hosting, SSO, audit, budgets and per-repo policy across more than one agent vendor. [R1 Recommendations: real, MED] The payer: every comparable monetises here, but the named platform teams built rather than bought (R3 §A).
2. **Engineering teams** (10 to 500) who outgrew worktree scripts or one vendor's desktop and want shared, policy-gated fleets. [R1 Recommendations: real but contested, MED] The user and adoption funnel; free OSS tools and vendor bundles cap the price (R1 Recommendations O2b; R3 §A).
3. **B2B2B embedders** [R1 Recommendations: not evidenced, LOW], split per R3 §A: 3a, agent-product startups whose product is the orchestration UX, not a launch audience [R3 §A: MED, absence of evidence]; 3b, embedders for whom the conductor is not the differentiator, a later channel tested by direct conversation [R3 §A: LOW].

## What it is NOT

- Not another coding agent; baton orchestrates them.
- Not a general workflow engine (Temporal, Orkes class); it may use one as a substrate (R1 §3d, Analysis).
- Not an IDE.
- Not a dev-environment platform or sandbox provider; those are providers underneath (R3 §A, Recommendations).
- Not a hosted-first cloud (R3 §D).
- Not "a better Conductor UI": the UI is commoditised; invest in the policy, gate, audit and adapter layer (R1 Analysis; R3 §A, §C).

## Strategy

**Invariants (R3 §B, §D).** Any company may offer baton to its own teams or to its customers; the core is under an OSI-approved licence; self-host first, hosted later. The licence and wedge eliminations below rest on these and reopen only if they change (R3 Handoffs).

**Licence** (R3 §B; Recommendations, MED). Apache-2.0 core plus a commercial, source-available `enterprise/` module. FSL, BSL with a competing-use grant, ELv2 and Sustainable Use fail the customer invariant and SSPL the open invariant, all BLOCKING for the core. Core: gates, budgets, basic audit, all adapters. Enterprise: SSO/SCIM, multi-org RBAC, audit export, HA, air-gap, branding, SLA. DCO, no CLA (R3 §F, Handoffs).

**Wedge and sequence** (R3 §D: viable HIGH; preferred MED). OSS self-hosted team server with a thin CLI/desktop client first; hosted second; headless and white-label API last; hosted-first and closed-first are eliminated by BLOCKING items. Pricing: Enterprise per conducting user per year; hosted compute as sandbox-hours plus a platform fee; never meter self-hosted by usage (R3 §B; Recommendations, MED).

**Architecture stance** (R2 options analysis verdict; advisory preference, MED). Build option B (team server, self-hosted runners, worktree and container executors, Postgres) inside option C's provider-neutral boundaries. A standalone and D (vendor-hosted control plane) first are eliminated by BLOCKING cons; E (extend an OSS server or integrate Coder by API) is spiked against B in HLD-000. First slice (R2 Three-month scope): two agents, two executors, at most two Git hosts.

**Naming status** (R3 §5, §E). Leading candidate `cuelist`; weak second `kapell`. A USPTO knock-out search returned 0 records for both; full USPTO, EUIPO and UKIPO clearance is pending; `cuelist` has weak distinctiveness ("cue list" is a show-control term of art); `.dev` or `.io` is open. Reserve the winner the day it clears.

## Design principles

Each v0.1 principle was challenged against R2; two were added; none dropped; all stay open to challenge in HLD-000.

- **Agent-agnostic by contract** (kept, restated). Adapters implement one normalized interface and declare per-agent capabilities; parity is never assumed; ACP, MCP and A2A are complementary formats, none a job specification; every adapter supports API-key and enterprise billing (R2 §2, Analysis; R3 §C).
- **One run, one checkout; isolation is a separate, labelled choice** (split from "isolation is the unit of safety"). Unsandboxed worktrees are `trusted-local`; a sandboxed backend serves lower-trust work and never mounts host checkouts or credentials (R2 §1, Analysis).
- **Gates are SHA-bound and fail closed** (added; ACP carries tool permission, not change approval). Approvals bind to the candidate SHA and invalidate on change; execution identities cannot self-approve or merge (R2 Analysis, Recommendations).
- **Policy as configuration, audit as evidence** (kept, extended). Gates, budgets, allowed tools and merge rules are declarative per org and repo; the audit log is append-only and reconstructable; both ship in the open edition (R2 Analysis; R3 §B).
- **Self-host first, hosted later** (kept, qualified). One product binary plus Postgres in one Compose file, not an infrastructure-free binary (R2 Analysis; R3 §D).
- **The open core runs the defining workflow end to end** (added; task -> run -> diff -> gate -> PR needs no paid module). Core includes at least two adapters, one local and one container backend, single-org roles and auditable events (R2 Analysis; R3 §B).
- **Customizable, not forkable** (kept, restated). Branding, roles, gates, adapters and providers are versioned extension points; untrusted plugins never run in the server (R2 §4, Analysis).

## How we will know

**The payer kill test** (R3 §G item 1): 10 to 15 calls with platform/DevEx leads, at least half at organisations running an in-house agent harness, against a two-page spec of the open/enterprise split. PASS: at least 3 non-binding design-partner letters AND at least 2 named budget lines. FAIL: re-scope to an OSS team server with no Enterprise plan, or stop.

**Before any HLD is approved**, all seven R3 §G experiments run (R3 Recommendations, HIGH): kill test, leaver/stayer interviews, segmented landing page, 3b conversations, fake-door pricing, issue-tracker mining, written vendor check.

**Leading indicators** after an OSS launch, on opt-in heartbeat instances with the opt-in rate stated (R3 §G): activation, at least 40% of instances complete a first agent run within 24 hours; multi-adapter, at least 20% of active instances use two or more adapters; team adoption, at least 50 instances with 3 or more users and one configured gate in week 4.

**Day-90 rule** (R3 §G): continue if activation is at least 40% AND multi-adapter share at least 20% AND at least 2 pilots are paid; one miss buys 30 days; two misses mean re-scope.

## Open questions

1. Will a Stripe-, Ramp- or Spotify-class platform team buy rather than build? Kill test (R3 §G item 1) into PRD-000.
2. Will Anthropic, OpenAI, Google and Cursor confirm in writing that third-party conductors may use subscription auth? R3 §G item 7; R2 Open questions.
3. Which two agents and two Git hosts do the first design partners need, and what does "onboard in an hour" include? PRD-000 (R2 Open questions).
4. What trust tier do pilot workloads need: hardened container or microVM by default? HLD-000 (R2 Open questions).
5. B or E: build runner plumbing, or integrate an OSS daemon or Coder by API? HLD-000 spike (R2 options analysis verdict).
6. Will Agent HQ add non-GitHub hosts or a customer-hosted control plane; when does Conductor self-host ship? R1 updates; PRD-000 (R3 Open questions).
7. Is there a 3b embedder willing to sign a letter of intent? R3 §G item 4; tenancy HLD latent until then.
8. Full trademark clearance for `cuelist` and `kapell`, and `.dev` or `.io`? Recorded against R3 §5, §E.

## How this repo works

`docs/research/` holds Phase 0 research, a URL per claim; `docs/design/` holds the PRD, HLDs and LLDs per `docs/design/TEMPLATE-HLD.md`; `docs/plans/` holds plans, none before HLD-000 is approved; `docs/process/` holds the brief, phase plan and review protocol. A different model reviews each deliverable; nothing is built before its HLD, LLD and plan exist.

## Changes from v0.1

- Thesis rewritten: alive incumbent, Agent HQ, OSS conductors, the gap as R1 and R3 state it.
- New sections: Phase 0 learnings; Strategy with invariants; How we will know; Open questions; Review log.
- Audience 3 split into 3a and 3b (R3 §A); "onboard in an hour" labelled a hypothesis.
- Isolation principle split; SHA-bound gates and end-to-end open core added; others restated against R2.

## Review log

| Tag | Change or reason | Where |
|---|---|---|
| B1 | Price unit per vendor as R3 §3 classes it: per user vs flat per team | Phase 0, pay bullet |
| B2 | Labels cited to R1 Recommendations or R3 §A, attributed; R1 audience-3 verdict added | Who it is for |
| B3 | Unconditional invariants paragraph added; thesis says "and, later, your customers" | Strategy; thesis |
| A1 | Stale R2 pending labels and status clause deleted; R2 APPROVED v2 | Header; Phase 0; Strategy; Principles |
| A2 | "ownership and deal terms UNVERIFIED" | Thesis |
| A3 | OSS conductors: servers, self-host, partial slot occupancy (R2 §5; R3 §C) | Thesis |
| A4 | Ona closed to new customers (R2 §4) | Phase 0 |
| A5 | Sub-claims re-cited (R1 §4, Analysis; R3 §A; R2 scope, Open questions); "not the UI" dropped | Thesis; NOT; Strategy; OQ 2 |
| A6 | One interface, declared capabilities, parity never assumed | Principle 1 |
| A7 | "at least two adapters"; "branding (Enterprise)" | Principle 6; thesis |
| A8 | Labels carried for gap, licence, wedge, pricing, architecture; hour claim LOW with reason | Thesis; Strategy |
| A9 | Seven pre-HLD experiments and day-90 rule added | How we will know |
| A10 | Naming status restated as R3 §5, §E has it | Strategy |
| A11 | v0.1 in git history; lead holds v0.1 and will confirm continuity; principles open to HLD-000 | Principles intro |
| A12 | Multi-fact bullets split; repo and changes sections shortened | Phase 0; NOT; last two sections |
