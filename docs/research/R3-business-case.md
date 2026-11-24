# R3 — Business case: is there a business in an open, self-hostable conductor, and how to position it

> Status: DRAFT v2 (revised after lead review; reconciled with Conductor's shutdown reversal, R1 S25-S27). Author: Fable · R3 business case. Date: 2026-11-23.
> Research question: is there a business in an open, self-hostable, customizable conductor
> platform for fleets of coding agents, and how should it be positioned (audience, wedge,
> licence, pricing, name, launch, metrics)?
> Method: `web_search -s public` was down all session; every fact is from a direct fetch of the
> cited URL on 2026-11-23, is cross-referenced to R1 as `per R1`, or is marked `UNVERIFIED`. Rules: `docs/process/BRIEF.md`.

## Facts

### 1. Infrastructure-as-a-product precedents (B2B2B and open-core)

- Netflix stopped maintaining Conductor OSS on GitHub effective 2023-12-13, keeping an internal fork; the archived README points users to community forks. https://github.com/Netflix/conductor
- The community fork `conductor-oss/conductor` is Apache-2.0, "actively maintained by Orkes and the community", and has repositioned from microservice orchestration to "durable execution platform for microservices, AI agents, and adaptive workflow graphs" with LLM tasks, MCP tool calling and human approval steps. https://github.com/conductor-oss/conductor
- Orkes monetises with a free "Developer Playground" (hosted, not for production, single user) and a custom-priced Enterprise plan offered either Orkes-hosted or customer-hosted ("all compute and data stay within your cloud account"), with up to 99.99% SLA and a TAM. https://orkes.io/pricing
- Roadie has sold hosted Backstage since 2020, bills on "contributing users" (people who write code tracked in the catalog; others log in free), supports private custom Backstage plugins published via its CLI, and is SOC2 Type 2. https://roadie.io/pricing/
- Backstage is published by Spotify as an open platform (site metadata lists Spotify as publisher). https://backstage.io/
- Coder's core is AGPL-3.0; the `enterprise/` directory is under a separate proprietary licence that forbids circumventing licence-key functionality. https://raw.githubusercontent.com/coder/coder/main/LICENSE and https://raw.githubusercontent.com/coder/coder/main/LICENSE.enterprise
- Coder's tiers: Community (free, open source; "Run Claude Code, Codex, Cursor, and other AI coding agents inside ephemeral workspaces"), Premium ("annually per user": prompt audit, multi-org RBAC, OIDC sync, HA, SLA support), AI Premium (custom: "Coder Agents" in the control plane, headless agents via API, network-isolated agent workspaces, bring-your-own model). https://coder.com/pricing
- Sourcegraph moved its main repository private; `sourcegraph-public-snapshot` is "a publicly available copy ... just before the migration", the last commit under Apache, and "contains primarily non-OSS-licensed files". https://github.com/sourcegraph/sourcegraph-public-snapshot
- Sourcegraph's current pricing is credit-based per user with org-wide pooling, offered as single-tenant cloud or self-hosted, and markets that it "works with Claude Code, Cursor, Codex, Amp, and more". https://sourcegraph.com/pricing
- Stripe Connect offers two models: "Stripe handles pricing for your users" (no platform fees; platform may earn a revenue share) or "You handle pricing for your users" (platform pays e.g. 0.25% + 25¢ per payout). https://stripe.com/connect/pricing
- Auth0: Free to 25,000 monthly active users; Essentials/Professional; Enterprise with a "Private Deployment" add-on; now markets "AI Identity" for agents and MCP servers. https://auth0.com/pricing
- Clerk charges a base fee of $0 / $25 / $300 per month plus $0.02 per monthly retained user above 50,000 per app; "Remove Clerk branding" is a paid-tier feature. https://clerk.com/pricing
- Algolia prices by usage: 10,000 requests/month included, then $0.60 per additional 1K requests. https://www.algolia.com/pricing

### 2. Open-core / self-hosted licensing: what happened to whom

- HashiCorp moved Terraform from MPL-2.0 to BSL 1.1 in 2023; on 2023-09-20 the Linux Foundation announced the OpenTofu fork with pledges from 140+ organisations and 600+ individuals and at least 18 full-time developers for five years. https://www.linuxfoundation.org/press/announcing-opentofu (HashiCorp's post, https://www.hashicorp.com/blog/hashicorp-adopts-business-source-license, returned HTTP 429.)
- Elastic relicensed away from Apache in 2021 over a hyperscaler's competing offering and "the market confusion" it caused, expecting and getting a fork; on 2024-08-29 it added AGPL back as a licence option, calling the three years "painful" but saying "it worked". https://www.elastic.co/blog/elasticsearch-is-open-source-again
- Redis moved to SSPL in March 2024; two hyperscalers "now maintain their own fork"; Redis acknowledged SSPL "is not truly open source" and subsequently moved to AGPLv3. https://redis.io/blog/agplv3/
- The Functional Source License (FSL) is a "Fair Source" licence that converts to Apache-2.0 or MIT after two years and is "designed for SaaS companies" to prevent "harmful free-riding"; it is not OSI open source. https://fsl.software/
- n8n's Sustainable Use License permits use "only for your own internal business purposes or for non-commercial or personal use" and redistribution "only if you do so free of charge for non-commercial purposes". https://docs.n8n.io/sustainable-use-license/
- GitLab states it "has an open core business model and generate[s] almost all our revenue with subscriptions to paid tiers", balancing open-source code with "source-available features". https://about.gitlab.com/company/stewardship/

### 3. Pricing models observed for comparable dev-tool platforms (all fetched 2026-11-23)

- Per seat: Coder Premium is "annually per user" (https://coder.com/pricing); Cursor lists Teams at "$30+/user/month", Enterprise custom (https://cursor.com/pricing); Sourcegraph bundles credits per user (https://sourcegraph.com/pricing).
- Per team with pooled credits: Devin "Teams" is $80/month with shared on-demand credits for up to 100 members, SAML/OIDC SSO, and 2 concurrent free-mode sessions per user; Enterprise adds SCIM/RBAC and a "dedicated deployment option". https://devin.ai/pricing
- Per sandbox-hour: Vercel Sandbox bills $0.128 per active-CPU hour, $0.0212 per GB-hour of provisioned memory, $0.60 per 1M sandbox creations, 24-hour max session and 10,000 concurrent sandboxes on Pro/Enterprise. https://vercel.com/docs/vercel-sandbox/pricing
- Per sandbox-hour: GitHub Codespaces compute is $0.18/h (2-core), $0.36/h (4-core), $0.72/h (8-core), $1.44/h (16-core). https://docs.github.com/en/billing/concepts/product-billing/github-codespaces
- Per sandbox-second plus platform fee: E2B Hobby is free with $100 one-time credit (1-hour sessions, 10 concurrent); Pro is $150/month plus usage "billed by the second", 24-hour sessions, 100 concurrent expandable to 1,100. https://e2b.dev/pricing
- Per resource-hour: Daytona bills vCPU at $0.0504/h, memory at $0.0162/GiB-h, storage at $0.000108/GiB-h, with $200 free compute. https://www.daytona.io/pricing
- Per action: Temporal Cloud starts at $50 per million actions (self-service down to $25), storage $0.042/GB-h active, and support is "greater of $500/mo or 10% of usage". https://temporal.io/pricing
- Bundled: Anthropic's Claude Code cloud sessions are GA (2026-09-23) for Pro, Max, Ultra, Team and Enterprise seats (https://www.anthropic.com/news/claude-code-on-the-web; Pro is $17/month annual, https://www.anthropic.com/pricing); GitHub Agent HQ agents are "part of your paid GitHub Copilot subscription" (§4); Google Antigravity is "available at no charge" (§4).

### 4. Competitive moves: Conductor's trajectory, agent vendors, platforms

- Conductor (conductor.build) describes itself as "Run a team of coding agents in the cloud": multiplayer shared workspaces, "Conductor Cloud runs on isolated microVMs", "runs the first-party Claude Code, Codex, Cursor, and OpenCode agents under the hood", desktop + iOS + API. https://conductor.build/
- 2026-03-30: Conductor raised a $22M Series A from Spark and Matrix, reported 10x user growth since January 2026, said "YC startups are writing 100% of their code in Conductor", listed a dozen large technology companies each with "dozens of engineers using Conductor", and stated the roadmap as "the platform for managing huge AI organizations, available everywhere from your phone to your VPC". https://conductor.build/blog/series-a
- 2026-05-13: Anthropic "announced subscription updates for Claude Agent SDK usage in third-party tools like Conductor"; 2026-06-15: Anthropic "announced that those changes are delayed indefinitely". Conductor runs Claude Code through the Agent SDK, "so that policy applied to us". https://conductor.build/blog/claude-subscription-update
- 2026-09-29: Conductor shipped "Sign in with ChatGPT", "partnering with OpenAI", so users bring their ChatGPT plan to run Codex in Conductor. https://conductor.build/blog/sign-in-with-chatgpt
- Conductor pricing: $0 / $50 / $60 per user per month plus custom Enterprise (`per R1 §1`; https://conductor.build/pricing redirected to the homepage in this session).
- 2026-11-04: Conductor announced it was "joining Meta" and "winding down Conductor over the next weeks and months": "Conductor Cloud goes read-only on November 13 and shuts down on November 20"; "Local Conductor keeps getting new models and critical bug fixes until early 2027". The text is preserved in the 0.94.2 changelog entry. https://conductor.build/changelog
- 2026-11-12/13: reversal, nine days later. The same entry now reads "Update: Conductor isn't shutting down. Conductor is back." and "Removed the Conductor Cloud shutdown banner from cloud workspaces"; a co-founder's post of 2026-11-12: "CONDUCTOR IS BACK ... we're doing it here at Meta" (`per R1 S25-S27`). https://conductor.build/changelog
- To 2026-11-23: releases continued through 0.99.1 — "New chats now default to Opus 5.5 for Claude and GPT-6.2 Sol for Codex", auto-resume after usage limits, a Usage page, artifacts, themes; cloud workspaces remain. Conductor is an actively developed, Meta-owned, proprietary cloud + desktop product with no self-hosted option; self-hosting is "being built" for Enterprise only (`per R1`). https://conductor.build/changelog
- Aftermath (`per R1 S25-S27`, user reports): trust damage ("already broke our trust once", Hacker News); some users moved to free open-source desktop conductors (T3 Code, Paseo, Orca, Herdr, Paradev). Leavers versus stayers: `UNVERIFIED`.
- GitHub "Agent HQ" (announced at GitHub Universe; fetched 2026-11-23): "coding agents from Anthropic, OpenAI, Google, Cognition, xAI, and more will become available directly within GitHub as part of your paid GitHub Copilot subscription"; "mission control" to "assign, steer, and track the work of multiple agents"; "a dedicated control plane to govern AI access and agent behavior"; branch controls for agent-created code; agent identity; compute via "GitHub Actions or self-hosted runners". https://github.blog/news-insights/company-news/welcome-home-agents/
- Cursor 2.0 ships "a new interface for working with many agents in parallel ... powered by git worktrees or remote machines", runs multiple models on the same problem, and adds review and a native browser tool. https://cursor.com/blog/2-0
- OpenAI Codex (2025-05) is "a cloud-based software engineering agent that can work on many tasks in parallel", each in an isolated sandbox preloaded with the repo. https://openai.com/index/introducing-codex/
- Anthropic's cloud sessions run "multiple tasks in parallel across different repositories from a single interface", each in an isolated sandbox with network/filesystem restrictions and a git proxy, with automatic PR creation. https://www.anthropic.com/news/claude-code-on-the-web
- Google Antigravity is an "agentic development platform" whose App is "your command center to manage multiple local agents in parallel", plus a CLI with "background subagents", an IDE and an SDK. https://antigravity.google/
- Name reservations by 2026 agent-orchestration projects (registry metadata, 2026-07 to 2026-10): `divisi` ("Delegate Codex work to frontier-model vendor CLIs"), `attacca` ("agent pool protocol"), `tutti` ("Multi-agent orchestration CLI"), `yardmaster` ("coordinates agents across machines"), `batonhq` ("Hand a unit of coding work from one AI agent to another"), `paddock` ("Sandboxed Docker environment for running AI coding agents"), `anchorleg` ("Rotate coding-agent subscriptions"). https://registry.npmjs.org/<name>, https://crates.io/api/v1/crates/<name>

### 5. Naming availability checks (all 2026-11-23; HTTP status or registry JSON)

- `baton`: GitHub user and repo `baton/baton` exist (https://github.com/baton/baton); `ConductorOne/baton` is an Apache-licensed identity-governance toolkit (https://github.com/ConductorOne/baton); `openbaton` is an existing NFV-orchestration organisation (https://github.com/openbaton); npm `baton` taken ("An orchestration tool", 2022; https://registry.npmjs.org/baton); PyPI `baton` taken (https://pypi.org/pypi/baton/json); crates.io `baton` taken (https://crates.io/api/v1/crates/baton); npm `batonhq` taken by an agent hand-off tool (2026-08).
- `baton` domains: baton.dev registered 2019-03-26 (exp. 2027-03-26); baton.run registered 2019-05-21; baton.io created 2019-05-07; baton.sh created 2025-03-18; getbaton.dev registered 2026-03-03. Source: https://rdap.org/domain/<domain> and `whois`.
- `baton` trademarks: USPTO/EUIPO search not performed (script-driven UIs; `web_search` down). ConductorOne's Baton and Open Baton are verified in-market uses; registration status `UNVERIFIED`.
- Alternatives (npm / PyPI / crates.io / GitHub handle / domains):
  - `ripieno`: npm FREE, PyPI FREE (404), crates FREE; GitHub user exists, `ripieno-dev` and `ripienohq` free; ripieno.io not registered (whois "Domain not found"); ripieno.dev registered 2026-07-15.
  - `cuelist`: npm FREE, PyPI FREE, crates FREE; GitHub user exists, `cuelist-dev`/`cuelisthq` free; cuelist.io not registered; cuelist.dev registered 2026-04-23.
  - `kapell`: npm FREE, PyPI FREE, crates FREE; GitHub user exists, `kapell-dev`/`kapellhq` free; kapell.io created 2022-08-11 (taken); kapell.dev registered 2026-05-25.
  - Rejected on checks: `divisi`, `tutti`, `attacca`, `fermata`, `downbeat`, `yardmaster`, `podium`, `concertmaster`, `kapellmeister`, `anchorleg`, `callboard`, `paddock` (all have a conflicting npm or crates package, several by 2026 agent-orchestration projects); `ostinato` is free on npm/PyPI but taken on crates.

## Analysis

### A. Critique of the B2B2B (white-label) thesis

What distinguished the infrastructure-as-a-product winners in §1:

- The function was *not* the customer's differentiator and carried compliance or operational burden (payments, identity, search); buyers were glad to outsource it. (HIGH: every cited winner fits.)
- Pricing scaled with the customer's success — per payout, per retained user, per request — so revenue grew without renegotiation. (HIGH: Stripe, Clerk, Algolia pricing pages.)
- Time-to-first-value was minutes for one developer, and white-labelling was an *upsell* ("Remove Clerk branding" is paid), not the entry product. (HIGH: Clerk.)
- Where open-core infrastructure *lost* or retreated, the attacker was the platform owner bundling the function (Elastic/Redis vs. hyperscaler forks; Sourcegraph closing its source once code-AI became the product) or the function turned out to be the customer's core UX. (MED: causal attribution is interpretation.)

Applying this to the three audiences in `VISION.md` v0.1 (numbered as there):

- **Audience 1 — platform/DevEx teams (self-host, SSO, audit, per-repo policy).** Where the money is in every comparable: Coder gates audit, RBAC, OIDC sync, HA and SLA behind Premium; Devin gates SCIM/RBAC and "dedicated deployment"; Orkes sells customer-hosted Enterprise; Roadie sells SOC2 and private plugins (§1, §3). Conductor itself gates self-hosting behind an Enterprise tier that is still "being built" (§4), which confirms the payer shape and leaves the slot open for now. Risk: Coder already sells "secure environments for developers and their agents" and GitHub promises "a dedicated control plane to govern AI access and agent behavior" (§4). (HIGH on payer shape; MED on defensibility.)
- **Audience 2 — engineering teams (10–500) who outgrew one vendor's desktop.** Demand is real but has no deadline. Pull: 100K+ builders (vendor claim), 10x growth in 2026, a $22M Series A, dozens of engineers at large companies, and $50–60 per-user tiers that teams pay (§4). Push is weaker than v1 assumed: Conductor did not go away, so no cohort *must* move, and those who left in November went to free open-source desktop tools (§4), which now contest this audience on price. What the episode created is a permanent *reason to prefer open and self-hosted* — a nine-day demonstration that a proprietary conductor's existence is a vendor decision, and the incumbent remains proprietary with no self-hosted option. Verdict: real demand, contested by free desktop tools and a well-resourced incumbent; the open self-hosted team server is the differentiated offer, not a rescue. (MED on pull; HIGH on competitive pressure.)
- **Audience 3 — startups building agent products (B2B2B).** For an agent-product startup the "run many agents, review, merge" surface *is* the product UX — the inverse of the Stripe/Auth0 condition. Conductor, the best-funded instance of the category, never sold a white-label edition; its roadmap was "your phone to your VPC" (§4), i.e. self-hosting, not embedding. No fetched evidence of a startup white-labelling a third-party conductor (R1 agrees: LOW). Verdict: **not a launch audience** (MED: absence of evidence). It can become real later as a headless API plus embeddable review/merge components priced per end-customer workspace (Clerk shape), once a core exists (see D).

Net: the thesis is right that there is a buyer, wrong about which buyer leads. Audience 2 is the user, Audience 1 the payer, Audience 3 a later channel. Position the product as the **neutral control plane above workspaces and agents** — the task → isolated workspace → agent → gate → merge lifecycle, with sandboxes (Coder, Codespaces, E2B, Daytona, Vercel Sandbox, worktrees) and agents as pluggable providers — not as "a better Conductor UI" and not as a dev-environment platform.

### B. Open-core / self-hosted playbook

Licence options. All are viable in the sense of "legally usable"; cons are labelled per the brief.

1. **Apache-2.0 for the whole core** (Conductor OSS, Backstage pattern).
   - Pros: maximal adoption; foundation-compatible; zero legal friction for Audience 1 (platform-team) procurement; explicit patent grant; consistent with the pitch "open so it cannot be shut down — or nearly shut down" (§4).
   - Cons: `[ADVISORY]` a hyperscaler or agent vendor can ship a hosted baton (the Elastic/Redis experience) — mitigated because baton's value is *self*-hosting and neutrality; `[ADVISORY]` the moat must come from operations, enterprise modules and ecosystem, not licence.
2. **MIT**. Same as (1) without the patent clause. `[ADVISORY]` weaker patent protection for contributors and users; no upside over Apache-2.0.
3. **AGPL-3.0 core + proprietary `enterprise/` directory** (Coder pattern).
   - Pros: deters closed hosted forks; proven in the closest comparable (Coder).
   - Cons: `[ADVISORY]` some enterprise legal teams prohibit AGPL inbound, taxing exactly Audience 1 (platform teams); `[ADVISORY]` network copyleft complicates the later B2B2B channel (embedders must publish modifications or buy a commercial licence); `[ADVISORY]` incompatible with CNCF/LF-style neutral governance.
4. **BSL 1.1 / FSL (fair source, converts to OSS after 2–4 years)**.
   - Pros: blocks a competing hosted service during the window; Sentry-style sustainability.
   - Cons: `[ADVISORY]` not OSI open source — the project cannot say "open source", undercutting the one message that distinguishes it from vendor conductors; `[ADVISORY]` HashiCorp's precedent (140+ organisations forked within weeks, §2) shows the community cost; `[ADVISORY]` Elastic and Redis both added AGPL back (§2).
5. **Elastic License v2 / SSPL**. `[ADVISORY]` all of (4) plus Redis's own admission that SSPL "is not truly open source"; both originators have retreated.
6. **Sustainable Use (n8n-style)**. `[BLOCKING]` forbids commercial redistribution and limits use to "internal business purposes"; the `VISION.md` invariant is that any company can offer baton "to its teams *or to its customers*", which this licence prohibits without a separate commercial contract. Eliminated.

Verdict: only option 6 is eliminated by a BLOCKING item. Among 1–5 the choice is ADVISORY and overrulable; this doc recommends **Apache-2.0 core** with a **commercial, source-available `enterprise/` module** (Coder's packaging, GitLab's stewardship) because Apache's ADVISORY cons are mitigated by self-host-first positioning while those of 3–5 land on the paying audience. (MED: judgment, not physics.)

What goes where (pattern from Coder, GitLab, Roadie, Orkes, Devin pricing pages):

- Community (Apache-2.0): single-binary/compose server, CLI, web UI, worktree and container isolation, all agent and sandbox adapters, GitHub/GitLab/Linear integrations, declarative gates and budgets, single-org, OIDC login, basic audit log, opt-in telemetry.
- Enterprise (commercial, source-available): SSO/SCIM sync, multi-org RBAC, audit export, policy packs with approvals, HA, air-gap bundles, branding/theming (Clerk precedent), cross-vendor cost accounting, SLA support.
- Hosted cloud: *after* the self-hosted product has paying customers (Coder, Orkes, Roadie all added hosting later; hosted-first is the single point of failure Conductor's users watched nearly fail between 2026-11-04 and 11-13). Price it as pass-through sandbox-hours (§3 rates give the floor) plus a platform fee.

Pricing shapes observed and the recommended mix (§3):

- Per seat (Coder, Cursor Teams, Sourcegraph) is the dominant enterprise shape. Recommend Enterprise per "conducting user" per year (Roadie's "contributing users" definition spares reviewers).
- Per sandbox-hour (Vercel, Codespaces, E2B, Daytona) fits hosted compute only; never meter the self-hosted product by usage.
- Per action (Temporal) and platform fee / revenue share (Stripe, Clerk) fit the later B2B2B channel.

### C. Competitive risk and how an open agent-agnostic layer survives

Threat map (all from §4):

- **Agent vendors' native conductors** (Anthropic cloud sessions, Codex, Cursor 2.0, Antigravity): free or bundled, vendor-locked by construction. They commoditise the *single-agent* parallel-tasks experience but do not serve mixed fleets, non-vendor sandboxes or self-hosted policy. (HIGH)
- **GitHub Agent HQ**: the existential threat — multi-vendor, bundled into Copilot, "mission control", control plane, agent identity, Actions or self-hosted runners. Agent-agnostic *and* distributed. (HIGH on threat; MED on timing — "over the coming months".)
- **Coder**: already self-hosted, already AGPL open core, already selling "AI Premium" with agent control plane and network-isolated agent workspaces. Overlaps with Audience 1. (HIGH)
- **Meta-owned Conductor**: the direct competitor. Active (0.94.2 → 0.99.1 in the ten days to 2026-11-23, §4), cross-vendor, cloud microVMs plus desktop, mobile and API, per-user pricing, now with a platform owner's resources. Its gaps are this project's thesis: proprietary, no self-hosted option (Enterprise self-host "being built"), trust damaged by a nine-day announced shutdown. Distribution risk: Meta can make it free. (HIGH on threat; MED on intent — primary signals are the changelog and one co-founder post.)
- **Free open-source desktop conductors** (T3 Code, Paseo, Orca, Herdr, Paradev; `per R1`): absorbed November's leavers; contest Audience 2 on price and openness but, as desktop apps, offer no shared policy, audit or server-side control plane. (MED: catalogued by R1, not reviewed here.)
- **Cloud providers**: each sells its own agent and sandbox primitives; hosting a copy of an Apache-licensed baton is possible but delivers neither neutrality nor on-prem. (MED)
- **Vendor policy risk**: Anthropic's 2026-05-13 announcement on Agent SDK use in third-party tools (delayed indefinitely 2026-06-15) shows a conductor relying on consumer subscriptions can lose its economics by vendor decision; the November shutdown-and-reversal shows a proprietary conductor's existence is likewise one decision away. (HIGH: both documented by Conductor itself.)

Survival strategy for an open layer:

- Serve what bundled conductors structurally cannot: mixed fleets (Cursor itself reports that racing multiple models improves results), non-GitHub code hosts, air-gapped deployments, customer-chosen sandboxes, and gates/budgets that span vendors.
- Make vendor-policy risk a feature: support API-key and enterprise billing paths and self-hosted models from day one; never depend on consumer-plan arbitrage.
- Court the vendors as partners, not enemies: Conductor got "Sign in with ChatGPT" from OpenAI (§4); a neutral surface that increases a vendor's usage is in every second-place vendor's interest.
- Offer neutral governance (a foundation) if and when a second vendor wants to contribute; OpenTofu shows how fast a neutral home attracts commitments when trust in a single vendor is in question (§2).
- Make the November episode the positioning: "the conductor your organisation can keep running whatever its vendor decides". Against a proprietary incumbent whose users have seen a shutdown notice, open + self-hosted is a durable message, not a rescue, and stays true even if Meta ships Enterprise self-hosting (proprietary, tier-gated).
- Accept that the UI layer will be commoditised; invest moat in the policy/gate/audit layer and the adapter ecosystem, which is where Audience 1 pays.

### D. Wedge options

1. **OSS self-hosted team server first → hosted cloud second → white-label/headless last** (recommended).
   - Pros: matches `VISION.md` "self-host first"; offers Audience 2 the one thing neither the proprietary incumbent nor the free desktop tools offer (a shared, self-hosted, policy-gated server) and converts to the proven payer (Audience 1); Coder, Orkes and Roadie followed this order.
   - Cons: `[ADVISORY]` slower revenue than hosted-first; `[ADVISORY]` weak usage telemetry (mitigate with opt-in heartbeat); `[ADVISORY]` overlaps Coder on Audience 1 — mitigate by integrating Coder as a sandbox provider; `[ADVISORY]` no forced migration wave to ride — adoption must be earned against an active incumbent and free desktop tools.
2. **Hosted cloud first, OSS later** (Conductor's path).
   - Cons: `[BLOCKING]` violates the `VISION.md` invariant "self-host first, hosted later"; `[ADVISORY]` requires pre-funding a sandbox fleet before revenue (compute floor in §3); `[ADVISORY]` reproduces the single point of failure Conductor's users watched nearly fail in November 2026.
3. **White-label / headless API first** (Audience 3).
   - Cons: `[BLOCKING]` cannot be first: a white-label edition is a packaging of an engine that does not yet exist, so the ordering is logically impossible; `[ADVISORY]` no fetched evidence of demand; `[ADVISORY]` requires multi-tenancy and branding before the core lifecycle is right.
4. **Local-first desktop app first (OSS), server later** (Conductor's original 2025 path).
   - Pros: fastest time-to-wow for individuals; the surface Conductor's leavers chose.
   - Cons: `[ADVISORY]` desktop-only cannot satisfy Audience 1 (SSO, audit, shared policy), deferring monetisation; `[ADVISORY]` it is the surface vendors (Cursor 2.0, Antigravity), GitHub and now five free open-source projects (§4) already offer, so a sixth desktop app has no differentiator; `[ADVISORY]` splits engineering from the server that must exist anyway. Viable as a *client* of option 1.
5. **Closed, enterprise-sales-first** (sell self-hosted licences to platform teams, open later).
   - Cons: `[BLOCKING]` violates the `VISION.md` invariant "open"; `[ADVISORY]` removes the one differentiator against bundled conductors.

Verdict: options 2, 3 and 5 are eliminated by BLOCKING items alone. Options 1 and 4 both survive; preferring 1 over 4 is ADVISORY and overrulable, and the case for 1 strengthened after the reversal because the desktop slot is now crowded with free OSS entrants while the self-hosted server slot is empty. Recommended: option 1 with a thin desktop/CLI client (option 4's strength) from the first release. (HIGH that 1 is viable; MED that 1 beats 4.)

### E. Naming

- `baton` as a public name is **not recommended** (HIGH: §5). Every registry name, every obvious domain and the GitHub handle are taken; ConductorOne ships a developer tool called Baton; Open Baton is a prior orchestration project; `batonhq` (coding-agent hand-off) appeared in 2026-08. Trademark status is `UNVERIFIED`, but the in-market collisions alone would force a scoped name everywhere. Keep `baton` as the internal codename only.
- The conducting namespace is being land-grabbed by agent projects now (§4), and even obscure terms' `.dev` domains were registered between April and July 2026. Reserve the chosen name on npm, PyPI, crates.io, a GitHub org and a domain on the same day.
- Three alternatives that passed the registry checks (§5):
  1. **ripieno** — the full-ensemble section that plays *together* behind the soloists; metaphor: the fleet behind the human. npm/PyPI/crates free, `ripieno-dev`/`ripienohq` free on GitHub, ripieno.io available. Con: `[ADVISORY]` pronunciation unfamiliar in English; `.dev` already taken (2026-07).
  2. **cuelist** — the ordered list of cues a stage manager calls; metaphor: tasks called in order, gated. npm/PyPI/crates free, `cuelist-dev`/`cuelisthq` free, cuelist.io available. Con: `[ADVISORY]` generic enough that trademark distinctiveness may be weak; `.dev` taken (2026-04).
  3. **kapell** — from *Kapellmeister*, the leader of an ensemble. npm/PyPI/crates free, `kapell-dev`/`kapellhq` free. Con: `[ADVISORY]` kapell.io taken (2022) and kapell.dev taken (2026-05); only `.sh`/`.run`/`.app` would remain (not checked).
- Recommendation: run USPTO (TESS/TSDR), EUIPO and UKIPO searches on the final two candidates before announcing; this doc could not. (Trademark status: `UNVERIFIED` for all names.)

### F. PR-FAQ (hypothetical, dated 2027-06-15)

**[Name] 1.0: the open conductor for fleets of coding agents, now generally available for self-hosting**

*2027-06-15* — Today the [Name] project released 1.0 of its open-source, self-hostable conductor: a control plane that lets an engineering organisation run many coding agents in parallel — Claude Code, Codex, Cursor CLI, OpenCode, Gemini CLI or its own — each in an isolated workspace, behind declarative review gates, with one audit trail, on infrastructure it controls.

Teams adopted parallel agents faster than their tooling. Each vendor shipped a conductor that works only with its own agent. The independent conductor that defined the category was acquired by a large platform company in November 2026, told its users it would shut down, reversed within nine days, and remains proprietary with no self-hosted option. Platform teams were left choosing between lock-in, a control plane whose future is decided elsewhere, and home-grown worktree scripts with no policy, audit or shared view.

[Name] installs with one `docker compose up` or a single binary. Every task gets its own checkout and optional sandbox (worktrees, containers, or Coder, Codespaces, E2B, Daytona); teams see what every agent is doing; per-repo policy is configuration: which agents may run, which tools they may call, what they may spend, which checks and approvals a change must pass before it reaches a shared branch. Agents, sandboxes, gates, roles and branding are extension points, so organisations customise without forking.

"We had four agents from three vendors and five different ways to watch them," said a platform lead at an early design partner. "Now there is one queue, one set of gates and one log, and it runs in our VPC."

The core is Apache-2.0. A commercial Enterprise edition adds SSO/SCIM, multi-org RBAC, audit export, policy packs and SLA support; a hosted edition for teams that do not want to run infrastructure is in private beta. Get started at [docs URL].

**Five questions a skeptical CTO asks**

1. *Why not just use GitHub Agent HQ, Cursor, or my vendor's cloud sessions — they are free with what I already pay?* — If your fleet is one vendor's agent and your code is on GitHub, you probably should. [Name] is for organisations running more than one agent, code host or sandbox, needing gates and budgets that span vendors, or required to keep the control plane on their own infrastructure. We integrate with those surfaces rather than replacing them.
2. *The last conductor got acquired, told me my data would be deleted in sixteen days, then changed its mind. Why should I trust you more?* — You should not have to trust us. The thing you run is Apache-2.0 and lives on your machines. If the company behind [Name] disappears, your instance keeps working and the code can be forked under neutral governance — the OpenTofu path. We cannot promise the company survives; we can promise the software does.
3. *What stops Anthropic or OpenAI from cutting off third-party use of my subscriptions, as was announced and then paused in 2026?* — Nothing, which is why [Name] does not depend on it. Every adapter supports API-key and enterprise billing and self-hosted models; subscription login is a convenience, not the economic basis.
4. *Which features will move from free to paid next year?* — The community edition contract is published: the full task → workspace → agent → gate → merge lifecycle, all adapters, and single-org policy stay Apache-2.0. Paid features are organisational: identity sync, multi-org, audit export, HA, branding, support. We will not relicense the core (see question 2 for why that would be self-defeating).
5. *What leaves my network?* — By default, nothing except the calls your agents already make to their model providers. Telemetry is opt-in and documented field by field. In air-gapped mode the control plane, sandboxes and models all run inside your boundary.

### G. 90-day success metrics and pre-build validation

Measure after an OSS launch (first 90 days):

- Activation: installs that complete a first successful agent run within 24 hours (opt-in heartbeat); target ≥ 40% of installs. Stars are not a metric.
- Team adoption: instances with ≥ 3 distinct users and ≥ 1 configured gate in week 4 (the Audience 2 → 3 signal); target 50 instances.
- Agent-agnostic validation: share of active instances using ≥ 2 agent adapters; if < 20%, the neutrality thesis is weaker than assumed.
- Retention: week-4 and week-8 active instances as a fraction of week-1; ask churned users why (exit survey in the UI).
- Ecosystem: external pull requests merged, third-party adapters published, and vendor DevRel contacts who respond in writing about third-party use policy.
- Commercial: 5 platform-team design partners with a signed (even $0) pilot agreement and a stated procurement path; 2 with a paid pilot by day 90.

Cheapest validation *before* building (each ≤ 1 week, ≤ a few hundred dollars):

- Segmented landing page with three CTAs (team server / enterprise self-host / embed in my product) and a waitlist; measure sign-ups per segment and which CTA wins. If "embed" is < 10% of sign-ups, drop Audience 3 from the roadmap.
- Twenty interviews with Conductor users, half who LEFT during or after the November episode and half who STAYED, recruited from the Hacker News thread, Conductor's Discord and the T3 Code / Paseo / Orca / Herdr / Paradev communities. Leavers: which tool, why, what they gave up, would they pay for a self-hosted team server. Stayers: why the reversal restored enough trust, what Meta ownership changes, what would make them move. Both: do they run ≥ 2 agents; has anyone in their organisation asked for audit or policy. If stayers dominate and cite "it just works", the trust argument in §C is weaker than assumed.
- Fake-door pricing test: a pricing page with per-conducting-user Enterprise and a hosted sandbox-hour price; measure click-through to "talk to us".
- Issue-tracker mining of the open-source conductors R1 catalogues: count requests for SSO, audit, multi-agent, GitLab support, policy — this is a free demand signal for the Enterprise module.
- Written vendor check: ask Anthropic, OpenAI and Google developer-relations in writing whether third-party conductors may use subscription auth; the answers set the adapter and pricing design.
- Name reservation: reserve the chosen name on all registries and domains in one day (§E).

## Recommendations

- **HIGH** — Lead with Audience 2 (engineering teams) as users and Audience 1 (platform/DevEx teams) as payers; defer Audience 3 (B2B2B white-label) until a core exists and a landing-page test shows demand. (Comparables monetise organisational controls; no evidence of white-label conductor demand.)
- **MED** — Treat Audience 2 demand as real but contested, not a migration wave: no deadline, an active Meta-funded incumbent, and free OSS desktop tools that absorbed November's leavers. Win on what none offers — shared, self-hosted, policy-gated — and use the incumbent's proprietary status and nine-day shutdown episode as the durable positioning lever ("the conductor your organisation can keep running whatever its vendor decides"). Never pitch to "stranded" users. (Downgraded from HIGH in v1; evidence is the changelog itself.)
- **HIGH** — Wedge: OSS self-hosted team server with a thin CLI/desktop client (option D-1 plus D-4's client), then hosted, then headless/white-label. (Options 2, 3, 5 are eliminated by BLOCKING items; 1 vs 4 is judgment, tilted further toward 1 now that the desktop slot has five free OSS entrants.)
- **HIGH** — Do not ship under the name `baton`; reserve one of `ripieno`, `cuelist`, `kapell` (after a trademark search) on all registries the same day. (All checks in §5.)
- **MED** — Licence: Apache-2.0 core plus a source-available commercial `enterprise/` module; do not use BSL/FSL/SSPL and never Sustainable-Use. (Only Sustainable-Use is BLOCKING; the rest is ADVISORY judgment informed by the HashiCorp/Elastic/Redis record.)
- **MED** — Position as the neutral control plane *above* sandboxes and agents; integrate Coder, Codespaces, E2B, Daytona and Vercel Sandbox as providers rather than competing on dev-environment infrastructure. (Coder's head start is large; the gate/audit/multi-vendor layer is open.)
- **MED** — Price Enterprise per conducting user per year and hosted compute as sandbox-hours plus a platform fee; never meter self-hosted by usage. (§3.)
- **LOW** — Plan for neutral-foundation governance once a second vendor contributes; do not start there. (Benefit depends on vendor participation that does not yet exist.)
- **HIGH** — Run the six pre-build experiments in §G before any HLD is approved; the landing-page segmentation, the leaver/stayer interviews and the written vendor-policy check are the three that can change the design.

## Open questions

- Meta-owned Conductor: will it ship Enterprise self-hosting and when; will it stay cross-vendor or tilt to Meta's models; will Meta make it free? Each reshapes Audience 2; none removes the open/self-hosted gap.
- How large was November's exodus relative to Conductor's base, and where did leavers go? The §G interviews decide whether the trust argument sells or merely resonates.
- GitHub Agent HQ: when "mission control" and the agent control plane ship, will they support non-GitHub code hosts or self-hosted control planes? If yes, baton's niche shrinks to air-gapped and mixed-host organisations — but so does Conductor's.
- Will any agent vendor commit in writing to allowing third-party conductors to use subscription auth, or is API-key billing the only durable path?
- Would Coder treat baton as a partner (sandbox provider) or as a competitor, and does its AGPL licence constrain an Apache-2.0 integration?
- Trademark clearance for the three alternative names; whether a `.dev` domain is necessary or `.io`/`.sh` suffices.

## Handoffs to other docs

- **R1** — Meta timeline reconciled with S25-S27 (this doc cites the changelog directly). Please catalogue: the free OSS desktop conductors (T3 Code, Paseo, Orca, Herdr, Paradev) with licence and whether any has a server mode; the 2026 agent-orchestrator projects found by name reservation (`divisi`, `attacca`, `tutti`, `yardmaster`, `batonhq`, `paddock`, `anchorleg`); Coder Agents. Watch Conductor's Enterprise self-host status. Mine their issue trackers per §G.
- **R2** — adapters must support API-key, enterprise billing and self-hosted models, not only subscription login (§C); sandbox providers to abstract: Coder, Codespaces, E2B, Daytona, Vercel Sandbox (§3 units are what metering must record); gates/audit/multi-org are the paid surface and need clean extension boundaries from the first HLD.
- **HLD-000** — the community/enterprise split in §B is a design constraint (what must be an extension point); white-label branding is an upsell, not a core requirement.

## Sources

Fetched 2026-11-23 unless noted. Primary unless labelled.

- https://github.com/Netflix/conductor
- https://github.com/conductor-oss/conductor
- https://orkes.io/pricing
- https://roadie.io/pricing/
- https://backstage.io/
- https://coder.com/pricing
- https://github.com/coder/coder
- https://raw.githubusercontent.com/coder/coder/main/LICENSE
- https://raw.githubusercontent.com/coder/coder/main/LICENSE.enterprise
- https://github.com/sourcegraph/sourcegraph-public-snapshot
- https://sourcegraph.com/pricing
- https://stripe.com/connect/pricing
- https://auth0.com/pricing
- https://clerk.com/pricing
- https://www.algolia.com/pricing
- https://www.linuxfoundation.org/press/announcing-opentofu
- https://www.hashicorp.com/blog/hashicorp-adopts-business-source-license — HTTP 429 in session; not read
- https://www.elastic.co/blog/elasticsearch-is-open-source-again
- https://redis.io/blog/agplv3/
- https://fsl.software/
- https://docs.n8n.io/sustainable-use-license/
- https://about.gitlab.com/company/stewardship/
- https://cursor.com/pricing
- https://cursor.com/blog/2-0
- https://devin.ai/pricing
- https://vercel.com/docs/vercel-sandbox/pricing
- https://docs.github.com/en/billing/concepts/product-billing/github-codespaces
- https://e2b.dev/pricing
- https://www.daytona.io/pricing
- https://temporal.io/pricing
- https://www.anthropic.com/news/claude-code-on-the-web
- https://www.anthropic.com/pricing
- https://openai.com/index/introducing-codex/
- https://antigravity.google/
- https://github.blog/news-insights/company-news/welcome-home-agents/
- https://conductor.build/
- https://conductor.build/blog/series-a
- https://conductor.build/blog/claude-subscription-update
- https://conductor.build/blog/sign-in-with-chatgpt
- https://conductor.build/changelog — 0.94.2 entry (wind-down text and "Update: Conductor isn't shutting down. Conductor is back."), releases through 0.99.1
- https://conductor.build/pricing — redirected to homepage; pricing tiers taken `per R1 §1`
- Co-founder post of 2026-11-12, Hacker News trust quote, OSS desktop tools, Enterprise self-host status: `per R1 S25-S27` (R1 holds the URLs)
- https://registry.npmjs.org/<name>, https://pypi.org/pypi/<name>/json, https://crates.io/api/v1/crates/<name>, https://github.com/<name>, https://rdap.org/domain/<domain>, `whois` — naming checks (§5)
- https://github.com/ConductorOne/baton
- https://github.com/openbaton

## Review log

| Date | Reviewer | Verdict | Notes |
|---|---|---|---|
| 2026-11-23 | lead | REVISE | reconcile with Conductor reversal (R1 S25-S27) |
| 2026-11-23 | Fable · R3 | REVISED → v2 | Facts §4 rewritten from changelog primary; A/B/C/D/F/G, Recommendations, Open questions, Handoffs, Sources updated; audiences renumbered to VISION v0.1 |
