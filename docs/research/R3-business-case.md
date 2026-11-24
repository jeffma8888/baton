# R3 — Business case: is there a business in an open, self-hostable conductor, and how to position it

> Status: DRAFT v4 (revised after independent review and re-verification, `R3-business-case.review.md`; the Review log is the record). Author: Fable · R3 business case. Date: 2026-11-23.
> Research question: is there a business in an open, self-hostable, customizable conductor
> platform for fleets of coding agents, and how should it be positioned (audience, wedge,
> licence, pricing, name, launch, metrics)?
> Method: `web_search -s public` was down all session; every fact is from a direct fetch of the cited URL on 2026-11-23, carries R1's source URL inline, was verified by the reviewer (marked), or is marked `UNVERIFIED`. Rules: `docs/process/BRIEF.md`.

## Facts

### 1. Infrastructure-as-a-product precedents (B2B2B and open-core)

- Netflix stopped maintaining Conductor OSS on GitHub effective 2023-12-13, keeping an internal fork; the archived README points to community forks. https://github.com/Netflix/conductor
- `conductor-oss/conductor` is Apache-2.0, "actively maintained by Orkes and the community", repositioned as a "durable execution platform for microservices, AI agents, and adaptive workflow graphs" with LLM tasks, MCP tool calling and human approval steps. https://github.com/conductor-oss/conductor
- Orkes: free hosted "Developer Playground" (not for production) and custom-priced Enterprise, Orkes-hosted or customer-hosted ("all compute and data stay within your cloud account"), up to 99.99% SLA. https://orkes.io/pricing
- Roadie has run "a SaaS version of Backstage since 2020" (hosted from day one), bills on "contributing users" (people who write code tracked in the catalog; others free), supports private custom plugins via its CLI, SOC2 Type 2. https://roadie.io/pricing/
- Backstage is published by Spotify as an open platform. https://backstage.io/
- Coder's core is AGPL-3.0; `enterprise/` is under a proprietary licence that forbids circumventing licence-key functionality. https://raw.githubusercontent.com/coder/coder/main/LICENSE and https://raw.githubusercontent.com/coder/coder/main/LICENSE.enterprise
- Coder tiers: Community (free; "Run Claude Code, Codex, Cursor, and other AI coding agents inside ephemeral workspaces"), Premium ("annually per user": prompt audit, multi-org RBAC, OIDC sync, HA, SLA), AI Premium (custom: "Coder Agents", headless agents via API, bring-your-own model; §4). https://coder.com/pricing
- Sourcegraph moved its main repository private; `sourcegraph-public-snapshot` is "a publicly available copy ... just before the migration" and "contains primarily non-OSS-licensed files". https://github.com/sourcegraph/sourcegraph-public-snapshot
- Sourcegraph pricing: credits per user with org-wide pooling, single-tenant cloud or self-hosted; "works with Claude Code, Cursor, Codex, Amp, and more". https://sourcegraph.com/pricing
- Stripe Connect: "Stripe handles pricing for your users" (no platform fees; optional revenue share) or "You handle pricing for your users" (platform pays e.g. 0.25% + 25¢ per payout). https://stripe.com/connect/pricing
- Auth0: free to 25,000 monthly active users; Essentials/Professional; Enterprise with a "Private Deployment" add-on; markets "AI Identity" for agents and MCP servers. https://auth0.com/pricing
- Clerk: $0 / $25 / $300 per month plus $0.02 per monthly retained user above 50,000; "Remove Clerk branding" is paid. https://clerk.com/pricing
- Algolia: 10,000 requests/month included, then $0.60 per additional 1K requests. https://www.algolia.com/pricing

### 2. Open-core / self-hosted licensing: what happened to whom

- HashiCorp moved Terraform from MPL-2.0 to BSL 1.1 in 2023; on 2023-09-20 the Linux Foundation announced the OpenTofu fork with pledges from 140+ organisations, 600+ individuals and at least 18 full-time developers for five years. https://www.linuxfoundation.org/press/announcing-opentofu (HashiCorp's own post returned HTTP 429.)
- Elastic relicensed away from Apache in 2021 over a hyperscaler's competing offering and "the market confusion" it caused, expecting and getting a fork; on 2024-08-29 it announced it "will be adding" AGPL as an additional licence option, calling the three years "painful" but saying "it worked". https://www.elastic.co/blog/elasticsearch-is-open-source-again
- Elastic License v2: "You may not provide the software to third parties as a hosted or managed service". https://www.elastic.co/licensing/elastic-license (reviewer)
- Redis moved to SSPL in March 2024; two hyperscalers "now maintain their own fork"; Redis acknowledged SSPL "is not truly open source" and added AGPLv3 "as an additional licensing option". https://redis.io/blog/agplv3/
- The Functional Source License (FSL) converts to Apache-2.0 or MIT after two years, is "designed for SaaS companies" to prevent "harmful free-riding", and is not OSI open source (https://fsl.software/). Its grant excludes any "Competing Use": making the software available "in a commercial product or service that ... offers the same or substantially similar functionality as the Software". https://fsl.software/FSL-1.1-ALv2.template.md (reviewer)
- n8n's Sustainable Use License permits use "only for your own internal business purposes or for non-commercial or personal use" and redistribution "only if you do so free of charge for non-commercial purposes"; its FAQ says the CLA "gives n8n the ability to change its license without seeking additional permission". https://docs.n8n.io/sustainable-use-license/
- GitLab "has an open core business model and generate[s] almost all our revenue with subscriptions to paid tiers", with "source-available features". https://about.gitlab.com/company/stewardship/
- CNCF charter §11: inbound contributions "made under the Apache License, Version 2.0" and "All outbound code will be made available under the Apache License, Version 2.0". https://github.com/cncf/foundation/blob/main/charter.md

### 3. Pricing models observed for comparable dev-tool platforms (fetched 2026-11-23)

- Per seat: Coder Premium "annually per user" (https://coder.com/pricing); Cursor Teams "$30+/user/month", Enterprise custom (https://cursor.com/pricing); Sourcegraph credits per user (https://sourcegraph.com/pricing).
- Per team flat fee: Devin "Teams" $80/month for up to 100 users, SAML/OIDC SSO, 2 concurrent free-mode sessions per user; Enterprise adds SCIM/RBAC and a "dedicated deployment option". https://devin.ai/pricing
- Per sandbox-hour: Vercel Sandbox $0.128 per active-CPU hour, $0.0212 per GB-hour, $0.60 per 1M creations, 24-hour max session (https://vercel.com/docs/vercel-sandbox/pricing); GitHub Codespaces $0.18/h (2-core) to $1.44/h (16-core) (https://docs.github.com/en/billing/concepts/product-billing/github-codespaces).
- Per sandbox-second plus platform fee: E2B Hobby free with $100 one-time credit (1-hour sessions); Pro $150/month plus usage "billed by the second", 24-hour sessions. https://e2b.dev/pricing
- Per resource-hour: Daytona vCPU $0.0504/h, memory $0.0162/GiB-h, storage $0.000108/GiB-h, $200 free. https://www.daytona.io/pricing
- Per action: Temporal Cloud from $50 per million actions (self-service to $25), storage $0.042/GB-h, support "greater of $500/mo or 10% of usage". https://temporal.io/pricing
- Bundled: Claude Code cloud sessions GA 2026-09-23 for Pro, Max, Ultra, Team and Enterprise (premium or Chat + Claude Code seats) (https://www.anthropic.com/news/claude-code-on-the-web; Pro $17/month annual, https://www.anthropic.com/pricing); GitHub Agent HQ agents are "part of your paid GitHub Copilot subscription" (§4); Antigravity "available at no charge" (§4).

### 4. Competitive moves: Conductor's trajectory, agent vendors, platforms

- Conductor (conductor.build): "Run a team of coding agents in the cloud" on "isolated microVMs", running "the first-party Claude Code, Codex, Cursor, and OpenCode agents under the hood"; "Trusted by 100k+ builders" (vendor claim); desktop + iOS + API (https://conductor.build/). Per R1 §1: Cloud is GitHub-only ("GitLab, Bitbucket ... Not yet") and stores chat transcripts on Conductor servers; self-hosting "Not yet, but we are building a self-hosted cloud offering ... as part of an Enterprise plan" (https://www.conductor.build/docs/cloud/faq); locally, "Agents run directly on your system without sandboxing" (https://www.conductor.build/docs/faq); the API manages cloud workspaces only (https://docs.conductor.build/api); pricing $0 / $50 Pro / $60 Teams per user / Enterprise (archived 2026-09-29, http://web.archive.org/web/20260929175413/https://www.conductor.build/pricing; the live page redirects to the homepage).
- 2026-03-30: $22M Series A (Spark, Matrix); 10x user growth since January 2026; "YC startups are writing 100% of their code in Conductor"; a dozen large technology companies with "dozens of engineers using Conductor"; roadmap "from your phone to your VPC". https://conductor.build/blog/series-a
- 2026-05-13 / 2026-06-15, Conductor's account: Anthropic "announced subscription updates for Claude Agent SDK usage in third-party tools like Conductor", then "those changes are delayed indefinitely"; Conductor runs Claude Code through the Agent SDK, "so that policy applied to us". https://conductor.build/blog/claude-subscription-update (Anthropic's notice not fetched)
- 2026-09-29: "Sign in with ChatGPT", "partnering with OpenAI": users bring a ChatGPT plan to run Codex. https://conductor.build/blog/sign-in-with-chatgpt
- 2026-11-04: "Conductor is joining Meta"; "We'll be winding down Conductor over the next weeks and months": "Conductor Cloud goes read-only on November 13 and shuts down on November 20"; "Local Conductor keeps getting new models and critical bug fixes until early 2027" (preserved in the 0.94.2 changelog entry, https://conductor.build/changelog); the archived announcement adds that on November 20 "data deletion begins" (http://web.archive.org/web/20261106225120/https://www.conductor.build/announcement).
- 2026-11-12: co-founder post "CONDUCTOR IS BACK. We've decided to continue development of Conductor!" (https://x.com/i/status/2120973052024598620); asked whether the team had left Meta: "nope, we're doing it here at Meta!!" (https://x.com/i/status/2120975128158945374). 2026-11-13 (0.94.2): "Update: Conductor isn't shutting down. Conductor is back."; shutdown banner removed (https://www.conductor.build/changelog/0.94.2-conductor-is-back). Deal structure and ownership: `UNVERIFIED`; the primary wording is "joining Meta".
- To 2026-11-23: releases through 0.99.1 — "New chats now default to Opus 5.5 for Claude and GPT-6.2 Sol for Codex", auto-resume, a Usage page; cloud workspaces remain. https://conductor.build/changelog
- Aftermath (self-reported, Hacker News): "already broke our trust once" (https://news.ycombinator.com/item?id=50473888, comment 50477646); users reported moving to open-source conductors T3 Code (MIT), Orca (MIT), Paseo (Apache-2.0), Herdr (Apache-2.0), Paradev (MIT core, paid tiers) (https://news.ycombinator.com/item?id=50369503; per-tool URLs in R1 table 3e). Leavers versus stayers: `UNVERIFIED`.
- GitHub "Agent HQ", published 2025-10-28: agents "from Anthropic, OpenAI, Google, Cognition, xAI, and more ... as part of your paid GitHub Copilot subscription"; "mission control" to "assign, steer, and track the work of multiple agents"; "a dedicated control plane to govern AI access and agent behavior" with "security policies, audit logging" and control over "which agents are allowed"; compute via "GitHub Actions or self-hosted runners" (https://github.blog/news-insights/company-news/welcome-home-agents/). Docs, 2026-11-23: "Third-party coding agents are currently in public preview" — Anthropic Claude ("powered by your existing Copilot subscription") and OpenAI Codex — enabled via "your organization policies, or the 'Agents' section of an enterprise's AI controls" (https://docs.github.com/en/copilot/concepts/agents/about-third-party-agents).
- Vendor conductors: Cursor 2.0, "many agents in parallel ... powered by git worktrees or remote machines" (https://cursor.com/blog/2-0); Codex, "a cloud-based software engineering agent that can work on many tasks in parallel" (https://openai.com/index/introducing-codex/); Claude Code cloud sessions, "multiple tasks in parallel across different repositories from a single interface" (https://www.anthropic.com/news/claude-code-on-the-web); Antigravity, "your command center to manage multiple local agents in parallel" (https://antigravity.google/).
- Coder: "Self-hosted cloud development environments for your developers and their agents" (https://coder.com/). Premium: "Audit prompts, restrict access, and manage costs so agents operate within your policies", "3rd-party AI agent compatibility (e.g., Claude, Codex)", "Customizable branding"; AI Premium: "Run Coder Agents natively in your control plane, with no vendor SaaS or third-party agent wrapper" (https://coder.com/pricing; reviewer).

### 5. Naming availability checks (2026-11-23)

Method: npm / PyPI / crates.io; GitHub user and `-dev`/`hq` handles; GitHub repository-name search (`https://api.github.com/search/repositories?q=<name>+in:name`) for same-category products, reading the README of every hit whose description is empty, non-English or a release/landing repository; https://rdap.org/domain/<domain> and `whois`; USPTO via `https://tmsearch.uspto.gov` and TSDR (reviewer). EUIPO and UKIPO: not checked for any name.

- `baton`: GitHub `baton/baton` exists; `ConductorOne/baton` is an Apache-licensed identity-governance toolkit (https://github.com/ConductorOne/baton); `openbaton` is an NFV-orchestration organisation (https://github.com/openbaton); npm (2013), PyPI and crates.io `baton` taken; npm `batonhq` ("Hand a unit of coding work from one AI agent to another", 2026-08-21); baton.dev/.run/.io (2019), baton.sh (2025-03), getbaton.dev (2026-03) registered. USPTO (reviewer): live BATON word marks in software classes — Reg. 6418232, IC 042, "software for project management and collaboration" SaaS, registered 2021-07-13, ACTIVE (https://tsdr.uspto.gov/statusview/sn90272389); Reg. 5388392 (IC 009/042); Reg. 4888336 (IC 009); BATON CORE Reg. 7153516; pending SN 99679424 (IC 009/042).
- `ripieno`: registries free, but a **same-category product exists**: https://github.com/ijmh2/ripieno (2026-08-08, "A self-hosted shared workspace where several people and several AI agents work on one codebase"), homepage https://ripieno.vercel.app ("MIT open source"); ripieno.dev (2026-07-15) is probably the same project (`UNVERIFIED`). **Rejected.**
- `cuelist`: npm / PyPI / crates FREE; GitHub user `cuelist` taken (2017), `cuelist-dev`/`cuelisthq` free; cuelist.io unregistered; cuelist.dev registered 2026-04-23. Repo-name search: 17 repositories; READMEs read for the 8 with empty or non-English descriptions: all show-control or setlist tools (QLab, ETC Eos, grandMA3, QLC+, Chataigne, Companion, Max, MIDI, song catalogues), none agent-related; "cue list" is a show-control term of art. USPTO: 0 records for CUELIST and "CUE LIST" (reviewer).
- `kapell`: npm / PyPI / crates FREE; GitHub user `kapell` taken (2015), `kapell-dev`/`kapellhq` free; kapell.io taken (2022); kapell.dev registered 2026-05-25. Repo-name search: 89 repositories, including three same-category near-homographs named `kapelle`: https://github.com/EdwardLiuyc/kapelle (2026-08-19, "A terminal chatting workspace for a team of coding agents: each in its own git worktree", 1 star), https://github.com/0xkilgore/kapelle (2026-10-14, "Agent work under owner approval, with verifiable results") and its companion https://github.com/0xkilgore/kapelle-architecture ("local-first agent operations"; a `kapelle-desktop` app also exists). USPTO: 0 for KAPELL; 2 non-live records for KAPELLMEISTER (reviewer).
- `ritornello`: registries free and USPTO 0 (reviewer), but a **same-category product exists**: https://github.com/allansteve/ritornello-releases (macOS app "Ritornello", v0.1.0 released 2026-07-17; the Portuguese README describes orchestrating AI agents in terminals on a canvas with evaluators, exit conditions, budgets and a human ship/block verdict — the repository description, "official releases", does not reveal this, which is why the v3 search missed it). GitHub organisation `ritornello-dev` taken (2026-04-30); ritornello.dev (2026-07-23) hosts an unrelated Anki-tools brand; ritornello.io unregistered. **Rejected.**
- Rejected on checks: 2026 agent-orchestration packages `divisi` ("Delegate Codex work to frontier-model vendor CLIs"), `attacca` ("agent pool protocol"), `tutti` (crates, 2026-03-15, "Multi-agent orchestration CLI"), `yardmaster`, `paddock` ("Sandboxed Docker environment for running AI coding agents"), `anchorleg`; `fermata`, `downbeat`, `podium`, `concertmaster`, `kapellmeister`, `callboard`, `ostinato` (npm or crates taken); `partitur` (515 repo-name hits; a music-notation library); `stretto`, `anacrusis` (npm taken). https://registry.npmjs.org/<name>, https://crates.io/api/v1/crates/<name>

## Analysis

### A. Critique of the B2B2B (white-label) thesis

The §1 winners share four traits (MED: four author-chosen winners; survivorship applies): the function was not the customer's differentiator and carried compliance or operational burden; pricing scaled with the customer's success (per payout, retained user, request); white-labelling was an upsell, not the entry product ("Remove Clerk branding" is paid); and where open-core infrastructure lost ground, a platform owner bundled the function (Elastic/Redis vs. hyperscaler forks; Sourcegraph closing its source).

Applied to `VISION.md` v0.1:

- **Audience 1 — platform/DevEx teams (self-host, SSO, audit, per-repo policy).** Every comparable monetises here: Coder (prompt audit, RBAC, OIDC sync, HA, SLA in Premium), Devin (SCIM/RBAC, "dedicated deployment"), Orkes (customer-hosted Enterprise), Roadie (SOC2, private plugins), Conductor (self-hosting as an Enterprise plan still being built) (§1, §3, §4). Counter-evidence (R1): Stripe, Ramp and Spotify built their own harnesses (https://stripe.dev/blog/minions-stripes-one-shot-end-to-end-coding-agents, https://builders.ramp.com/post/why-we-built-our-background-agent, https://engineering.atspotify.com/2025/11/spotifys-background-coding-agent-part-1); Coder already sells self-hosted environments "for your developers and their agents" with third-party agent compatibility (§4). The right payer shape, not yet a proven payer. (MED on payer: comparables monetise here but the named platform teams built; MED on defensibility: Coder already sells to them.)
- **Audience 2 — engineering teams (10–500) who outgrew one vendor's desktop.** Demand is real (100k+ builders claimed, 10x growth, $22M Series A, $50–60 per-user tiers, §4) but has no deadline: Conductor did not go away, and November's leavers went to open-source tools. The episode leaves an argument for open and self-hosted while the incumbent remains proprietary with no shipped self-hosted option — it says it is building one (https://www.conductor.build/docs/cloud/faq). (MED on pull: vendor-claimed numbers; HIGH on competitive pressure: an active incumbent plus free open-source tools.)
- **Audience 3 — B2B2B embedders**, two sub-segments:
  - *3a, agent-product startups whose product is the orchestration UX.* The surface *is* their differentiator — the inverse of the Stripe/Auth0 condition; no fetched evidence of any conductor sold white-label. **Not a launch audience.** (MED: absence of evidence.)
  - *3b, embedders for whom the conductor is not the differentiator:* code-host, forge and developer-portal vendors adding an agent surface (the Roadie private-plugin model); sandbox vendors wanting a reference surface that drives consumption (E2B, Daytona, Vercel Sandbox); model vendors whose differentiator is the model; consultancies delivering per-client tenancy and branding; platform teams serving many business units under their own brand. Here the Stripe/Auth0 condition holds. Signals are weak: Conductor API users built "mobile apps, UI clients, and internal tools, like Slackbots" (https://www.conductor.build/changelog/0.78.0-introducing-conductor-cloud); one HN user asked for "cloud webhooks + workspace/session API + isolated VMs" (https://news.ycombinator.com/item?id=50369503, comment 50370446). A later channel, tested by direct conversations (§G). (LOW: two self-reported signals, no identified buyer.)

Net: Audience 2 is the user, Audience 1 the payer, Audience 3 a later channel. Position as the **neutral control plane above workspaces and agents** — task → isolated workspace → agent → gate → merge — with sandboxes and agents as pluggable providers; not "a better Conductor UI", not a dev-environment platform.

### B. Open-core / self-hosted playbook

Two invariants applied uniformly here and in §D. *Customers:* `VISION.md` says any company can offer baton "to its teams *or to its customers*"; a licence that forbids offering the software to customers is `[BLOCKING]`. *Open:* VISION says only "open"; this document defines it as "the core is under an OSI-approved licence", so a non-OSI core is `[BLOCKING]`. Both eliminations are conditional on VISION v0.1, whose principles are "to be challenged in HLD-000".

1. **Apache-2.0 core** (Conductor OSS, Backstage). Pros: maximal adoption; foundation-compatible; no procurement friction for Audience 1; patent grant. Cons: `[ADVISORY]` a hyperscaler or agent vendor can ship a hosted copy (the Elastic/Redis experience), mitigated because the value is *self*-hosting and neutrality; `[ADVISORY]` the moat must come from operations, enterprise modules and ecosystem.
2. **MIT.** `[ADVISORY]` no patent clause; no upside over Apache-2.0.
3. **AGPL-3.0 core + proprietary `enterprise/`** (Coder). Pros: deters closed hosted forks; proven in the closest comparable. Cons: `[ADVISORY]` some enterprise legal teams prohibit AGPL inbound, taxing Audience 1; `[ADVISORY]` network copyleft complicates the B2B2B channel; `[ADVISORY]` incompatible with CNCF hosting, whose charter requires Apache-2.0 inbound and outbound unless the Governing Board grants an exception (§11(c), https://github.com/cncf/foundation/blob/main/charter.md; §2).
4. **BSL 1.1 / FSL.** `[BLOCKING]` FSL's "Competing Use" exclusion — and a BSL with the usual competing-use grant — forbids a customer offering the software "in a commercial product or service that ... offers the same or substantially similar functionality" (§2), the use the invariant protects. `[BLOCKING]` not OSI-approved, so not "open"; `[ADVISORY]` HashiCorp's precedent (140+ organisations forked within weeks); `[ADVISORY]` Elastic and Redis both added AGPL back.
5. **Elastic License v2 / SSPL.** ELv2 `[BLOCKING]`: "You may not provide the software to third parties as a hosted or managed service" (§2). SSPL `[BLOCKING]`: not OSI-approved — Redis itself says it "is not truly open source" — so not "open"; `[ADVISORY]` it does permit service offering subject to source disclosure, and both originators retreated from it.
6. **Sustainable Use (n8n).** `[BLOCKING]` limits use to "internal business purposes" and forbids commercial redistribution (§2).

Verdict: FSL/BSL-with-competing-use, ELv2 and Sustainable Use are eliminated by the customer invariant, SSPL by the open invariant; Apache-2.0, MIT and AGPL survive. Recommended: **Apache-2.0 core** plus a **commercial, source-available `enterprise/` module** (Coder's packaging, GitLab's stewardship) — Apache's cons are ADVISORY and mitigated by self-host-first positioning; AGPL's land on the paying audience. (MED: judgment.)

What goes where. Community (Apache-2.0): server, CLI, web UI, isolation, all agent and sandbox adapters, code-host and tracker integrations, declarative per-repo gates and budgets, basic audit log, single-org, OIDC, opt-in telemetry — open gates and audit are R1's gap (§C), where Coder charges Premium. Enterprise (commercial, source-available): SSO/SCIM, multi-org RBAC, audit export, policy packs with approvals, HA, air-gap bundles, branding, cross-vendor cost accounting, SLA. Hosted cloud: only after the self-hosted product has paying customers — a VISION invariant, not a comparable's path (Roadie has been hosted since 2020; Orkes sells both; Coder lists no hosted plan).

Pricing (§3): Enterprise per "conducting user" per year (Roadie's "contributing users" definition spares reviewers); hosted compute as pass-through sandbox-hours plus a platform fee; never meter the self-hosted product by usage.

### C. Competitive risk and how an open agent-agnostic layer survives

Threat map (facts in §4):

- **Agent vendors' native conductors** (Anthropic, Codex, Cursor 2.0, Antigravity): free or bundled, single-vendor by construction; they commoditise single-agent parallel tasks. (HIGH: all four ship today, §4.)
- **GitHub Agent HQ**: multi-vendor, bundled into Copilot, shipping — mission control since 2025-10-28, third-party Claude and Codex agents in public preview, org- and enterprise-level policies, a control plane promising audit logging and agent allow-lists. That is this project's gate/audit pitch, delivered inside one code host's cloud. (HIGH on threat; timing: shipping, public preview.)
- **Coder**: self-hosted, AGPL open core, third-party agent compatibility, prompt audit and branding in Premium, its own agents in AI Premium; governance is paid. (HIGH; overlaps Audience 1.)
- **Conductor, developed at Meta** (ownership `UNVERIFIED`): active (0.94.2 → 0.99.1 in ten days), cross-vendor, platform-owner resources, could be made free. Verifiable differences: proprietary; no self-hosted option today; Cloud GitHub-only; transcripts on Conductor servers; local mode "without sandboxing"; API covers cloud workspaces only. (HIGH on threat: shipping weekly with platform-owner resources; MED on intent: ownership and pricing plans `UNVERIFIED`.)
- **Open-source conductors** (T3 Code, Orca, Paseo, Herdr, Emdash, Paradev; R1 table 3e): not only desktop apps — Orca's machine plugins run agents on your own cloud; Paseo is a daemon with "Teams and triggers" via Paseo Hub; Emdash (only "Emdash Classic" is open source, in maintenance) runs on "your own infrastructure" with SSO, roles and spend limits; Paradev sells $0/$20/$60/$200 tiers with Enterprise SAML/SCIM/audit. They partially occupy the self-hosted slot. (MED: catalogued by R1.)
- **Cloud providers**: a hosted copy of an Apache-licensed baton delivers neither neutrality nor on-prem. (MED: the Elastic/Redis precedent, §2, in a different market.)
- **Vendor policy risk**: the Agent SDK notice and the November reversal show a conductor's economics and existence are each one vendor decision away. (HIGH: both happened in 2026, §4.)

The gap, as R1 states it (MED): "a self-hosted control plane that is cross-vendor and open in its core; declarative per-repo gates and audit available in the open edition" — Coder puts governance in Premium, Paradev and Emdash in paid or claimed tiers — plus a headless API comparable to Conductor's. Narrower than "nothing exists".

Survival strategy: build exactly that gap — cross-vendor gates, budgets and audit in the open edition, on a server the customer runs, with non-GitHub hosts, air-gap and customer-chosen sandboxes; API-key and enterprise billing from day one; court vendors as partners (Conductor got "Sign in with ChatGPT" from OpenAI); position on verifiable facts, not on the incumbent's episode; treat the UI as commoditised and invest in the policy/gate/audit layer and adapters, where Audience 1 pays.

### D. Wedge options

1. **OSS self-hosted team server first → hosted → headless/white-label** (recommended). Pros: matches "self-host first"; builds R1's gap directly; the payer shape every comparable monetises (§A). Cons: `[ADVISORY]` slower revenue than hosted-first; `[ADVISORY]` weak telemetry (opt-in heartbeat); `[ADVISORY]` overlaps Coder on Audience 1 — integrate Coder as a sandbox provider; `[ADVISORY]` no migration wave, and Paseo Hub, Emdash and Paradev already offer team or enterprise features.
2. **Hosted cloud first, OSS later** (Conductor's path). `[BLOCKING]` violates "self-host first, hosted later" (conditional on VISION v0.1); `[ADVISORY]` pre-funds a sandbox fleet before revenue; `[ADVISORY]` reproduces the single point of failure Conductor's users watched.
3. **Headless API / SDK first** (Audience 3b; R1 §5: "a headless API/SDK first; a tenant per end customer"). `[ADVISORY]` demand evidence is weak (§A); `[ADVISORY]` multi-tenancy and branding before the core lifecycle is right; `[ADVISORY]` every option must build the engine first — sequencing, not impossibility. Survives.
4. **Local-first desktop app first (OSS), server later.** Pros: fastest time-to-wow; the surface November's leavers chose. Cons: `[ADVISORY]` cannot satisfy Audience 1; `[ADVISORY]` crowded by vendors, GitHub and six open-source projects; `[ADVISORY]` splits engineering from the server that must exist anyway. Viable as a *client* of option 1.
5. **Closed, enterprise-sales-first.** `[BLOCKING]` violates "open" as defined in §B (conditional on VISION v0.1); `[ADVISORY]` removes the one differentiator against bundled conductors.

Verdict: 2 and 5 are eliminated by BLOCKING items conditional on VISION v0.1; 1, 3 and 4 survive; 1 is preferred on ADVISORY grounds — open gates and audit on a self-hosted server are what the open-source conductors and Coder Community do not ship; 3 needs the same engine; 4 enters a crowded slot. Ship 1 with a thin desktop/CLI client from the first release. (HIGH that 1 is viable: it is the payer shape every comparable monetises; MED that 1 beats 3 or 4: judgment on sequencing.)

### E. Naming

- `baton` as a public name is **not recommended** (HIGH: §5): registries, obvious domains and handles taken; two prior Baton developer tools and a 2026 `batonhq` agent tool; live USPTO BATON word marks in software classes, including Reg. 6418232 (IC 042) and a pending IC 009/042 application. Codename only.
- `ripieno` and `ritornello` are dropped: a live same-category product uses each name (§5). Lesson, learned twice: registries and handles are not enough; every candidate needs a repository-name search with the READMEs of unclear hits read, plus a product-name check.
- Ranking (§5):
  1. **cuelist** — the ordered list of cues a stage manager calls; tasks called in order, gated. Registries, `-dev`/`hq` handles and `.io` free; `.dev` and the bare GitHub user taken; no same-category repository after README reads; USPTO 0. Con: `[ADVISORY]` "cue list" is a show-control term of art (17 QLab/Eos/grandMA repositories): weak distinctiveness.
  2. **kapell** — from *Kapellmeister*. Registries and `-dev`/`hq` handles free; USPTO 0. Cons: `[ADVISORY]` three same-category near-homograph `kapelle` repositories (2026-08 to 2026-10), one with a desktop app — a user searching "kapell" will find them; `[ADVISORY]` `.io`, `.dev` and the bare GitHub user all taken; `[ADVISORY]` German spelling, uncertain pronunciation. A weak second: usable, not clean.
- Before announcing: USPTO, EUIPO and UKIPO searches on `cuelist` and `kapell` (or any new candidate that passes the full §5 method); reserve the name on every registry, a GitHub org and a domain the same day — agent projects are land-grabbing the conducting namespace (§5).

### F. PR-FAQ (hypothetical, dated 2027-06-15)

**[Name] 1.0: the open conductor for fleets of coding agents, now generally available for self-hosting**

*2027-06-15* — Today the [Name] project released 1.0 of its open-source, self-hostable conductor: a control plane that lets an engineering organisation run many coding agents in parallel — Claude Code, Codex, Cursor CLI, OpenCode, Gemini CLI or its own — each in an isolated workspace, behind declarative review gates, with one audit trail, on infrastructure it controls.

Teams adopted parallel agents faster than their tooling. Most vendors shipped a conductor for their own agent; the one cross-vendor surface lives inside a single code host's cloud. The independent conductor that defined the category joined a large platform company in November 2026, announced and then reversed a shutdown within nine days, and remains proprietary; the self-hosted Enterprise option it said in 2026 it was building (https://www.conductor.build/docs/cloud/faq) is assumed, for this hypothetical release, not yet to have shipped. Self-hosted alternatives exist; where their gates, audit and policy are priced, they sit in paid tiers (Coder, Paradev), and elsewhere pricing is unpublished.

[Name] installs with one `docker compose up` or a single binary. Every task gets its own checkout and optional sandbox (worktrees, containers, Coder, Codespaces, E2B, Daytona); per-repo policy is configuration — which agents may run, which tools they may call, what they may spend, which approvals a change must pass before it reaches a shared branch. Agents, sandboxes, gates, roles and branding are extension points.

"We had four agents from three vendors and five ways to watch them," said a platform lead at an early design partner [illustrative]. "Now there is one queue, one set of gates and one log, and it runs in our VPC."

The core is Apache-2.0. A commercial Enterprise edition adds SSO/SCIM, multi-org RBAC, audit export, policy packs and SLA support; a hosted edition is in private beta. Get started at [docs URL].

**Five questions a skeptical CTO asks**

1. *Why not GitHub Agent HQ? It is multi-vendor and included in Copilot.* — If you are GitHub-only and accept a GitHub-hosted control plane, use it. [Name] is for non-GitHub or mixed code hosts, a control plane inside your own boundary, air-gap, and gates and audit as open code you can read and extend. We integrate with GitHub rather than replacing it.
2. *Why not Coder? It is self-hosted, open core, runs third-party agents and audits prompts.* — Coder is the workspace layer; [Name] is the task → gate → merge layer above it and runs on Coder as one sandbox provider. The honest overlap is Coder's AI Premium, which adds its own agents and governance; if you are already on Coder Premium and single-vendor, you may not need us. Our gates and audit are in the free edition; Coder's are paid.
3. *The last conductor announced it would delete my data in sixteen days, then changed its mind. Why should I trust that your core stays open?* — You should not have to trust us. The software is Apache-2.0 and lives on your machines; if the company behind [Name] disappears, your instance keeps working and the code can be forked under neutral governance, as OpenTofu was. Contributions arrive under a Developer Certificate of Origin with no contributor licence agreement, so no single party can relicense the core. The community-edition contract is published: the full lifecycle, all adapters and single-org policy stay open. We cannot promise the company survives; we can promise the software does.
4. *What stops a model vendor from cutting off third-party use of my subscriptions, as one announced and then paused in 2026?* — Nothing, so [Name] does not depend on it. Every adapter supports API-key and enterprise billing; subscription login is a convenience. Self-hosted models work through open-source harnesses that accept a model endpoint; a proprietary vendor CLI always calls its vendor.
5. *What leaves my network?* — By default, only the calls your agents already make to their model providers. Telemetry is opt-in and documented field by field. Air-gapped mode keeps control plane, sandboxes and models inside your boundary — with open harnesses and local models; vendor CLIs cannot run air-gapped.

### G. 90-day success metrics and pre-build validation

Metrics after an OSS launch (first 90 days). Denominator: opt-in heartbeat instances; report container pulls and release downloads as the install proxy and state the opt-in rate (heartbeat biases toward engaged users).

- Activation: ≥ 40% of opt-in instances complete a first successful agent run within 24 hours. Stars are not a metric.
- Team adoption: ≥ 50 instances with ≥ 3 distinct users and ≥ 1 configured gate in week 4 (the Audience 2 → 1 signal).
- Agent-agnostic validation: ≥ 20% of active instances use ≥ 2 agent adapters; below that the neutrality thesis is weak.
- Retention: week-8 active instances ≥ 40% of week-1; exit survey for churned instances.
- Ecosystem: ≥ 10 external pull requests merged; ≥ 2 third-party adapters; written replies from ≥ 2 vendor DevRel teams.
- Commercial: 5 platform-team design partners with a signed (even $0) pilot agreement and a stated procurement path; 2 paid pilots by day 90.
- Day-90 rule: continue if activation ≥ 40% *and* multi-adapter share ≥ 20% *and* ≥ 2 paid pilots; one miss buys 30 days; two misses mean re-scope.

Pre-build validation (each ≤ 2 weeks; interview incentives roughly $2–3K total):

1. **Payer discovery calls (the kill test).** 10–15 calls with platform/DevEx leads, at least half at organisations already running an in-house agent harness, against a two-page spec of §B's community/enterprise split. Pass: ≥ 3 sign a non-binding design-partner letter *and* ≥ 2 name a budget line. Fail: re-scope to an OSS team server with no Enterprise plan, or stop.
2. **Leaver/stayer interviews.** 20 Conductor users, half who left around the November episode and half who stayed, recruited from the two HN threads and the conductor communities. Leavers: which tool, why, would they pay for a self-hosted team server. Stayers: why the reversal restored trust, what would make them move. Both: ≥ 2 agents? Anyone asking for audit or policy? If stayers dominate and cite "it just works", §C's trust argument is weak.
3. **Segmented landing page** with three CTAs (team server / enterprise self-host / embed) and a waitlist. Rules fire only after ≥ 300 qualified visitors. If "embed" < 10% of sign-ups, deprioritise Audience 3a — not 3b, which this page does not reach.
4. **Audience 3b conversations.** 5–10 direct conversations with forge/dev-portal vendors, sandbox vendors, model vendors and consultancies. Pass: ≥ 2 letters of intent; CTA share is not the test.
5. **Fake-door pricing page**: per-conducting-user Enterprise and a hosted sandbox-hour price. Pass: ≥ 5% of qualified visitors click "talk to us".
6. **Issue-tracker mining** of the open-source conductors in R1 table 3e: count open requests for SSO, audit, policy, multi-agent and GitLab support. Signal: ≥ 20 distinct requests across the catalogue.
7. **Written vendor check**: ask Anthropic, OpenAI and Google developer relations in writing whether third-party conductors may use subscription auth; the answers set adapter design.

Action, not experiment: reserve the chosen name on all registries and domains in one day (§E).

## Recommendations

- **MED** — Lead with Audience 2 as users and Audience 1 as payers; defer 3a; test 3b by direct conversation. (Both are MED.)
- **MED** — Differentiate on R1's gap, not on the incumbent's episode; never pitch to "stranded" users. (No deadline; active incumbent.)
- **MED** — Wedge: OSS self-hosted team server with a thin CLI/desktop client, then hosted, then headless/white-label. (1 over 3 and 4 is ADVISORY judgment.)
- **HIGH** — Do not ship as `baton`; take `cuelist` or `kapell`, or a new candidate that passes the full §5 method including README reads, to trademark search and reserve the winner everywhere the same day. (§5 collisions and live USPTO marks.)
- **MED** — Licence: Apache-2.0 core plus a source-available commercial `enterprise/` module. (Beyond the BLOCKING eliminations, judgment.)
- **MED** — Position as the neutral control plane above sandboxes and agents; integrate Coder, Codespaces, E2B, Daytona and Vercel Sandbox as providers. (Coder's head start.)
- **MED** — Price Enterprise per conducting user per year and hosted compute as sandbox-hours plus a platform fee; never meter self-hosted by usage. (§3.)
- **LOW** — Plan neutral-foundation governance once a second vendor contributes. (Depends on participation that does not exist.)
- **HIGH** — Run the seven §G experiments before any HLD is approved. (The payer calls can falsify the thesis before code.)

## Open questions

- Will GitHub Agent HQ support non-GitHub code hosts or a customer-hosted control plane? If yes, the niche shrinks to air-gapped organisations.
- Conductor at Meta: when does the Enterprise self-hosted option ship; does it stay cross-vendor; does it become free?
- What would make a Stripe-, Ramp- or Spotify-class platform team buy rather than build? (Payer calls, §G.)
- Will any agent vendor commit in writing to third-party conductors using subscription auth?
- Will Coder treat this project as a partner (sandbox provider) or a competitor?
- Trademark clearance (USPTO, EUIPO, UKIPO) for `cuelist` and `kapell`; whether `.dev` is necessary or `.io` suffices.

## Handoffs to other docs

- **R1** — catalogue the 2026 agent-orchestrator projects found by name reservation (`divisi`, `attacca`, `tutti`, `yardmaster`, `batonhq`, `paddock`, `anchorleg`, `kapelle`, `ripieno`) and Coder Agents; watch Conductor's Enterprise self-host status.
- **R2** — adapters must support API-key and enterprise billing, not only subscription login (§C); abstract Coder, Codespaces, E2B, Daytona and Vercel Sandbox as providers; gates, audit and multi-org need extension boundaries from the first HLD.
- **HLD-000** — the §B community/enterprise split is a design constraint; the BLOCKING eliminations in §B and §D rest on VISION v0.1 invariants and reopen if those change; adopt DCO-without-CLA if PR-FAQ Q3 is to be true.

## Sources

Fetched 2026-11-23 by the author unless noted "reviewer" (verified by the independent reviewer the same day) or "R1" (URL from R1's source list).

- https://github.com/Netflix/conductor
- https://github.com/conductor-oss/conductor
- https://orkes.io/pricing
- https://roadie.io/pricing/
- https://backstage.io/
- https://coder.com/ — reviewer (homepage quote)
- https://coder.com/pricing — author and reviewer
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
- https://www.elastic.co/licensing/elastic-license — reviewer
- https://redis.io/blog/agplv3/
- https://fsl.software/ and https://fsl.software/FSL-1.1-ALv2.template.md — template clause: reviewer
- https://docs.n8n.io/sustainable-use-license/
- https://about.gitlab.com/company/stewardship/
- https://github.com/cncf/foundation/blob/main/charter.md
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
- https://github.blog/news-insights/company-news/welcome-home-agents/ — published 2025-10-28
- https://docs.github.com/en/copilot/concepts/agents/about-third-party-agents
- https://conductor.build/
- https://conductor.build/blog/series-a
- https://conductor.build/blog/claude-subscription-update — Conductor's account of Anthropic's notice
- https://conductor.build/blog/sign-in-with-chatgpt
- https://conductor.build/changelog and https://www.conductor.build/changelog/0.94.2-conductor-is-back
- https://www.conductor.build/docs/cloud/faq — R1 S17
- https://www.conductor.build/docs/faq — R1 S8
- https://docs.conductor.build/api — R1 S20
- https://www.conductor.build/changelog/0.78.0-introducing-conductor-cloud — R1 S15
- http://web.archive.org/web/20260929175413/https://www.conductor.build/pricing — R1 S19
- http://web.archive.org/web/20261106225120/https://www.conductor.build/announcement — R1 S23
- https://x.com/i/status/2120973052024598620 and https://x.com/i/status/2120975128158945374 — R1 S25, S26 (co-founder posts)
- https://news.ycombinator.com/item?id=50369503 and https://news.ycombinator.com/item?id=50473888 — R1 S24, S29 (HN threads; self-reported)
- https://stripe.dev/blog/minions-stripes-one-shot-end-to-end-coding-agents, https://builders.ramp.com/post/why-we-built-our-background-agent, https://engineering.atspotify.com/2025/11/spotifys-background-coding-agent-part-1 — R1 S98–S100
- Open-source conductor facts (T3 Code, Orca, Paseo, Herdr, Emdash, Paradev): `R1-market-landscape.md` table 3e, S61–S73
- https://registry.npmjs.org/<name>, https://pypi.org/pypi/<name>/json, https://crates.io/api/v1/crates/<name>, https://api.github.com/users/<name>, https://api.github.com/search/repositories?q=<name>+in:name, https://rdap.org/domain/<domain>, `whois` — naming checks (§5)
- https://github.com/ijmh2/ripieno and https://ripieno.vercel.app — reviewer; repository re-fetched by the author
- https://github.com/allansteve/ritornello-releases — reviewer (re-verification); README re-fetched by the author. https://api.github.com/users/ritornello-dev, https://ritornello.dev — reviewer
- https://github.com/EdwardLiuyc/kapelle, https://github.com/0xkilgore/kapelle, https://github.com/0xkilgore/kapelle-architecture — author; the latter two found by the reviewer
- https://tsdr.uspto.gov/statusview/sn90272389 and `https://tmsearch.uspto.gov` — reviewer (USPTO)
- https://github.com/ConductorOne/baton
- https://github.com/openbaton

## Review log

| Date | Reviewer | Verdict | Notes |
|---|---|---|---|
| 2026-11-23 | lead | REVISE | reconcile with Conductor reversal (R1 S25-S27) |
| 2026-11-23 | Fable · R3 | REVISED → v2 | Facts §4 rewritten from changelog primary; A/B/C/D/F/G, Recommendations, Open questions, Handoffs, Sources updated; audiences renumbered to VISION v0.1 |
| 2026-11-23 | Opus · review R3 (independent) | REVISE | 8 BLOCKING + 14 ADVISORY; 46-row source spot-check (35 S, 5 P, 1 F, 1 S/F, 1 N/A); see `R3-business-case.review.md` |
| 2026-11-23 | Fable · R3 | REVISED → v3 | BLOCKING fixed: (1) ripieno dropped, USPTO facts, repo-name search added, ritornello added; (2) PR para 2 Agent HQ multi-vendor, "joined"; (3) Agent HQ and Coder FAQs, Q2+Q4 merged; (4) Agent HQ dated 2025-10-28, docs fact, timing "shipping"; (5) D-3 relabelled ADVISORY; (6) FSL/ELv2 BLOCKING on the same invariant; (7) narrower R1 gap, partial occupants named; (8) payer discovery calls with kill threshold. ADVISORY: 3b sub-segment, confidence labels lowered, metric denominators/thresholds, Audience 2 → 1, Conductor facts in §C, FAQ scoping, CNCF, inline R1 URLs, five source corrections, ~700-word cut |
| 2026-11-23 | Opus · review R3 (re-verification) | REVISE | all 8 original BLOCKING FIXED; 1 new BLOCKING (N1: `ritornello` collides with a same-category macOS app); advisory R-a..R-i; 12-row spot-check (9 S, 1 P, 1 S/P, 1 F) |
| 2026-11-23 | Fable · R3 | REVISED → v4 | N1: `ritornello` rejected in §5, §E, Recommendations and Open questions; §5 method adds README reads for unclear hits, applied to `cuelist` (8 READMEs read, none agent-related) and `kapell` (three `kapelle` collisions with URLs); ranking now cuelist, kapell. Advisory: R-a (PR para 2 toned down, cloud FAQ cited; "permanent" dropped), R-b ("open" defined as OSI-approved; SSPL and FSL/BSL now BLOCKING on it; verdict unchanged), R-c (open-source conductors / offer / Emdash Classic label), R-d (CNCF §11(c) exception), R-e, R-g (one-clause reasons on every label), R-h (status line). R-f, R-i skipped |
| 2026-11-23 | lead (on-disk verification of v4, per reviewer's conditional) | APPROVE | N1 fix confirmed at §5 L70, §E L135, Recommendations L192, Open questions L206; ritornello appears only as rejected; licence invariants consistent; no stale phrases. Research doc frozen as v4 for Phase 1 input. |
