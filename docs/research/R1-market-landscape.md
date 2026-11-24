# R1 — Market landscape: conductor platforms

> Status: DRAFT v1 (complete, awaiting review). Author: Opus · R1 market landscape. Date: 2026-11-23.
> Research question: the market landscape for conductor platforms (orchestration surfaces for running many coding agents in parallel in isolated workspaces with review and merge flow).

## Facts

### 1. Conductor (conductor.build, YC S24)

**Identity and funding**
- YC profile: "Run a team of coding agents on your Mac"; founded 2024 by two co-founders; 8 employees; San Francisco. [S1]
- YC launch: the founders ran Claude Code in three cloned directories. The v1 pitch was "start a new Claude in an isolated copy of your codebase" with review/PR/merge, and "It's free—we use your existing Claude Code login." [S1]
- 2025-07-17: Show HN "Conductor, a Mac app that lets you run a bunch of Claude Codes at once" (228 points, 115 comments). [S2]
- 2025-07-18: earliest changelog entry found (0.0.16). [S3]
- 2026-03-30: $22M Series A (Spark, Matrix); claims 10x user growth since January 2026; ambition: "the platform for managing huge AI organizations, available everywhere from your phone to your VPC." [S4]
- 2026-04-02: acqui-hired the founder of "cmd" (AI in Xcode). [S5]
- Homepage (fetched 2026-11-23) claims "Trusted by 100k+ builders" and shows large-company logos. This is a vendor claim, not independently verified. [S6]

**Product (as of 2026-11-23)**
- Docs: "Conductor lets you run Claude Code, Codex, Cursor, and OpenCode in parallel. Each task gets its own workspace, branch, files, terminal, diff, and review path". [S7]
- Isolation model (local): one git worktree + branch per workspace on the user's Mac; "Agents run directly on your system without sandboxing." [S8][S9]
- Workflow doctrine: "The workspace is the unit of delegation. The branch and pull request are the unit of integration." It includes inline diff comments sent back to the agent, an agent "Review" action, a Checks tab (CI, deployments, comments), and PR, merge and archive. [S9]
- Agent support timeline: Claude Code at launch [S1]; Codex added 2025-10-31 (0.18.0) [S10]; Cursor Composer added 2026-06-08 (0.63.0) [S11]; OpenCode added 2026-06-23 (0.69.0) [S12].
- Uses the Claude Agent SDK to drive Claude Code; was exposed to an Anthropic third-party-subscription policy change (announced 2026-05-13, delayed indefinitely 2026-06-15). [S13]
- Bring-your-own subscriptions and API keys; cloud-hosted model platforms and custom providers supported (0.13.6). [S6][S3]
- Configuration: per-repo setup/run/archive scripts in `.conductor/settings.toml` (formerly `conductor.json`); per-repo injected system prompts. [S11][S8]
- Integrations: GitHub (incl. GitHub Enterprise, 0.22.4), Linear, GitHub Issues, Graphite stacks, Vercel deployments. [S3]
- 2026-07-23 (0.77.0): multiplayer cloud alpha + Conductor API alpha for Pro users. [S14]
- 2026-07-30 (0.78.0): "Introducing Conductor Cloud": isolated microVMs that keep working when the laptop closes; public API; users built "mobile apps, UI clients, and internal tools, like Slackbots" on it. [S15]
- 2026-09-09 (0.85.0): Routines (agents on a schedule or via GitHub Action; cloud only), private workspaces. [S16]
- 2026-10 (0.90.0/0.92.0): iOS app, then remote control of local Mac workspaces. [S3]
- 2026-11-16 (0.95.0): agents can create/manage other workspaces and chats (agent-as-orchestrator). [S3]
- 2026-11-18/20 (0.96/0.98): cost and token Usage page. [S3]

**Cloud sandboxes**
- Each cloud workspace = isolated microVM; 8 vCPU, 16 GB RAM, 32 GB ephemeral NVMe; runs on Vercel Sandbox; "built in close collaboration with Vercel." [S17][S18]
- Org-level "Cloud Computer" (shared repos, env vars, secrets, install script, versioned builds) — new workspaces start from the active build; up to 50 GitHub repos; GitHub only ("GitLab, Bitbucket ... Not yet"). [S18][S17]
- Cloud stores chat transcripts on Conductor servers (local mode does not). [S17]
- "Can I use my own cloud machines instead of Conductor's? Not yet, but we are building a self-hosted cloud offering ... as part of an Enterprise plan." [S17][S19]

**API**
- REST under `https://api.conductor.build/v0`, OpenAPI spec, bearer API key, "in beta"; requires Pro. Endpoints: projects, workspaces (create/rename/status/archive/unarchive/sleep), sessions (messages, status, cancel, attachments), read-only `POST /v0/sql` over transcripts, routines with webhook URLs; plus an MCP server. Manages cloud workspaces only. [S20]

**Team features**
- Multiplayer: org-shared workspaces and chats, presence, follow, co-prompting, reassign, share links, shared org API keys, team prompts, per-member token heatmap. [S21][S16][S3]

**Pricing (archived 2026-09-29)**
- Free $0 (local workspaces, BYO keys); Pro $50/mo (Cloud + "a bunch of cloud workspace hours", multiplayer up to 5 Pro users, API, mobile); Teams $60/mo/user (admin portal, central billing; invite-only); Enterprise custom (DPA, SAML SSO, SCIM, SLA). Claims SOC 2 Type II. [S19]

**Meta, shutdown, reversal**
- 2026-11-04 (0.93.0 + announcement): "Conductor is joining Meta." Original timeline: Nov 4 no new sign-ups; Nov 13 Cloud read-only with refunds; Nov 20 Cloud shuts down and data deletion begins; local app fixes "until early 2027"; "We'll have something new for you to use early next year." [S22][S23]
- 2026-11-12: co-founder on X: "CONDUCTOR IS BACK. We've decided to continue development of Conductor! Reports of our death have been greatly exaggerated (by us)." [S25] Asked whether they left Meta: "nope, we're doing it here at Meta!!" [S26] The founder's X bio reads "Conducting @meta". [S25]
- 2026-11-13 (0.94.2): "Conductor isn't shutting down. It's back, and we're building again"; the Cloud shutdown banner was removed. [S27][S3]
- The live announcement page now carries "Plans have changed! We'll still be developing Conductor." and the timeline is gone. [S28]
- 2026-11-16 to 2026-11-23: six further releases (0.95.0–0.99.0), including cloud-org features; i.e. development resumed within days. [S3]
- **Correction to the project's prior knowledge:** primary sources supersede the brief's "Cloud shut 2026-11-20 ... winds down early 2027": the shutdown was reversed on 2026-11-12/13. It is UNVERIFIED whether Cloud was ever read-only and whether sign-ups have reopened (the homepage links to sign-up again [S6]).

### 2. Where Conductor users are migrating, and what they miss

Sources are the HN threads on the announcement [S24] and the reversal [S29]. Numbers are comment ids (`https://news.ycombinator.com/item?id=<id>`); all statements are self-reported.

**Destinations named by users (not founders)**
- T3 Code: a team that bought Conductor Cloud the day before is "doing a refund + moving the org to t3.codes" (50370063); others report switching (50370254, 50371355, 50388360); "best one ... that isn't glued to a specific model provider" (50370357).
- Paseo: "Fully open-source ... I run an instance of Paseo in a Hetzner box" (50371439); switched "because of its full featured mobile app, local client server architecture, and open sourcedness. Conductor began to feel stale, pricey ... moving toward enterprise" (50376205).
- Orca: migrated "when I needed better multi-machine control, and remote control" (50369888); switched because Conductor "didn't support mobile app working with local instances and didn't have a good cli/mcp" (50372452); one user went back to Herdr over UI bugs (50371064).
- Herdr (50369863, 50371299, 50478312); Tern by Stencil (50369888, 50370438, 50370456); bb / getbb.app (50370903, 50370960); boxes.dev (50371148); Paradev, formerly Superset (50372547, 50372531).
- No tool at all: "plain Claude Code: one git worktree per session ... one coordinator session that assigns the work" (50373638); "I just use the official cli + desktop apps now. tmux + a bit of custom scripting" (50371914).
- Vendor desktops seen as substitutes: "Conductor is really no different from Claude Code Desktop now, aside from multiple harness and models" (50374171); "Doesn't Claude desktop do all of this and more?" (50371521).

**Founder-posted alternatives** (self-promotion, weak adoption evidence): Orca ("Conductor imports", "bring your own cloud"; 50372338), Paradev ("imports your Conductor worktrees"; 50371655), boxes.dev (50372313), Superconductor (50373171), hive.codes, podium.do, opencomputer.dev, aweb.ai, Juggler, Command Center, talyn.dev (50395930), and an observability vendor's "Studio" (50372045).
- Observers: "The number of comments ... saying 'founder of xyz here' ... is illustrative" (50377383); "clearly no moat in this" (50374681).

**Migration tooling shipped within a day**
- Paradev published "Migrate from Conductor": adopts Conductor worktrees in place, maps `conductor.json`/`.conductor/settings.toml` to `.paradev/config.json`, maps env vars, maps Cloud/iOS/API to its own equivalents. [S30]
- 2026-11-07 Show HN "Conductor sold to Meta. Rebuilding the app (3 days, $170) (MIT)" [S31]; the linked repo now returns HTTP 404 (checked 2026-11-23). A team reports it "forked + reverse engineered the whole project" in an afternoon (50374304).

**What users say they value or miss**
- Conductor Cloud specifically: "cloud webhooks + workspace/session API + isolated VMs ... Does anyone have suggestions for products that can match Conductor Cloud, not just Conductor Local?" (50370446).
- Worktree ergonomics: "took 90% of the friction out of working with and using worktrees" (50370861); "the correct way [isolated branches and PRs] becomes just as fast" (50372271).
- Polish: "beautiful clean UI, great UX" (50370579).
- After moving to T3 Code: misses easy worktree cleanup on archive and tabs per workspace; wants better code navigation (50477897).
- Requested or missing: open-sourcing the app (50369888, 50369909, 50374718); multi-machine / own hardware (50369888, 50371142, 50371299); multi-repo work (50377216).
- Trust damage after the reversal: "already broke our trust once" (50477646); "Too late, why would anyone want to go back?" (50477299).

**Coverage gaps**
- Reddit returned HTTP 403 and public web search was down all session, so Reddit, X threads (beyond the founder posts) and blog posts were NOT collected. No public Conductor issue tracker was found (UNVERIFIED).

### 3. Alternatives (as of 2026-11-23)

Stars/licence/dates: GitHub API, 2026-11-23 (popularity, not usage).

**3a. Agent-vendor-native**
- Claude Code runs sessions in isolated git worktrees (`--worktree`) [S32] and has experimental "agent teams" [S33].
- Claude Code cloud sessions run on vendor VMs (GitHub required) [S34] or on customer runners via "self-hosted environments" [S35].
- Claude Desktop: parallel sessions, diff review, PR/CI status, local/cloud/SSH targets; a session can message or archive other sessions [S36].
- Claude Ship (Team/Enterprise): Code Review, PR Steward, "Claude Approvals" policy file [S37]; routines [S38]; repo has no OSS licence [S39].
- Codex Cloud gives each task its own cloud workspace, works with GitHub or GitLab, and has published environments and network/secret controls [S40][S41].
- Codex app uses worktrees; mobile controls a connected computer [S42]; enterprise managed config [S43]; part of ChatGPT plans [S44]; CLI Apache-2.0 [S45].
- Cursor Cloud Agents: isolated VMs, "unlimited" parallel agents, multi-repo, four code hosts; started from Slack, Linear, API or iOS [S46].
- Cursor Self-Hosted Machines: the customer's worker runs the tools while Cursor runs the agent loop. Partner hosts include E2B, Daytona, Modal and Vercel [S47][S48].
- Gemini CLI: subagents [S49], experimental worktrees [S50], Apache-2.0 [S51]. Jules: an experimental async agent in a cloud VM [S52].
- GitHub Agent HQ (2025-10-28): "mission control" inside Copilot for agents from Anthropic, OpenAI, Google, Cognition and xAI, with a governance control plane [S53][S54].
- Devin: Teams $80/mo for up to 100 members, multiplayer, SSO; Enterprise adds SCIM/RBAC and VPC [S55]. In VPC mode the "brain" stays in the vendor cloud and only the Devbox runs in the customer environment [S56].
- Factory: bring-your-own inference, Team up to 10 seats, Enterprise on-prem options and audit logs [S57]. "Workstreams" run agents on local or vendor cloud computers [S58].
- Open harnesses that the independent tools wrap: OpenCode (MIT, 228,834★) [S59] and OpenHands (MIT, 94,180★) [S60].

**3b. Independent tools (fields in table 3e)**
- T3 Code: "open-source control plane for coding agents"; "Fork the whole thing"; claims 500,000 developers [S61][S62].
- Orca (YC): bots triggered from Slack, Linear and GitHub; mobile apps; "machine plugins" that run agents on your own cloud or sandboxes [S63][S64][S65].
- Paseo: desktop, mobile and remote-machine daemon; "Teams and triggers" via Paseo Hub; MCP, CLI and TS SDK; 624 community plugins. The LICENSE file is Apache-2.0, though the API reports NOASSERTION [S66][S67].
- Herdr: reaches your own machines over SSH or Tailscale; no account needed; 576 "mods" [S68][S69].
- Paradev (formerly Superset): a worktree per task; CLI, TS SDK, MCP; paid cloud workspaces [S70][S71].
- Emdash (YC): "Run ... on your own infrastructure or Emdash cloud"; ACP agents; SSO/roles, spend limits, shared secrets/MCP. The OSS "Emdash Classic" is in maintenance only [S72][S73].
- Vibe Kanban: the company shut down because the "vast majority are free users and we couldn't find a business model". The project is now community-maintained and local-only [S74][S75].
- Terragon (cloud, one container per agent) shut down; an "open-source snapshot" was published 2026-01-16 [S76].
- Crystal is "now Nimbalyst" (last push 2026-02-26) [S77]. Claude Squad is tmux plus a worktree per task [S78].
- Long tail: Termic ("Open-source Conductor.build alternative"), bb, Sculptor, Juggler, ouijit, Silo, malini (11–5,936★) [S79].
- boxes.dev (YC): "Full computers, not worktrees". Each agent gets a cloud Linux devbox; work starts from Slack, Linear, a schedule or the API; "Sleeping devboxes are free" [S80].
- Superconductor: per-second credits (100 credits per hour for 1 vCPU/2 GB; overage $25 per 10,000 credits); "never charged per seat"; bring your own plans or keys [S81].

**3c. Self-hosted platforms and building blocks**
- Coder: self-hosted Coder Agents (chat + API, its own agent) and Tasks. Community edition is free under AGPL; Premium is priced per user and adds AI governance [S82][S83][S84][S85].
- Ona: "Enterprise software factory" of governed background agents (Ona, Codex and others) [S86].
- Building blocks: container-use (container + branch per agent) [S87]; agentapi (archived) [S88]; open-swe [S89]; Vercel coding-agent-template [S90]; E2B [S91]; Daytona (repo archived) [S92].

**3d. Workflow engines that market "agent orchestration"**
- Netflix/conductor was archived in Dec 2023 [S93]. conductor-oss/conductor calls itself an "event driven agentic workflow engine" (Apache-2.0, 32,495★) [S94].
- Orkes: "Agentic Workflow Platform" with durable, deterministic execution, RBAC and audit [S95]. Temporal: "orchestrator for AI applications" built on Durable Execution (MIT) [S96][S97].
- HN commenters confused Conductor with Netflix Conductor/Orkes (50371514, 50375033) [S24].

**3e. Comparison table**

| Product | Isolation model | Agents | Team features | Self-host | Licence | Pricing | Maturity |
|---|---|---|---|---|---|---|---|
| Conductor [S8][S17][S19] | Local worktree, no sandbox; cloud microVM | Claude Code, Codex, Cursor, OpenCode | Multiplayer, org cloud computer, API | No ("being built" for Enterprise) | Proprietary | $0 / $50 / $60 per user / Enterprise | Series A; shutdown reversed |
| Claude Code + Desktop [S32]–[S37] | Worktree; vendor VM; self-hosted runner | Claude only | Code Review, approvals policy, agent teams (exp.) | Execution only | Proprietary | Claude plans (tiers UNVERIFIED) | GA |
| Codex [S40]–[S45] | Worktree (app); per-task cloud workspace | Codex only | Enterprise managed config | CLI only | CLI Apache-2.0 | ChatGPT plans | GA |
| Cursor Cloud Agents [S46]–[S48] | Cloud VM; self-hosted worker | Cursor agent, multi-model | Team/Enterprise | Worker only; loop stays in vendor cloud | Proprietary | Team/Enterprise | GA |
| GitHub Agent HQ [S53] | GitHub-hosted (detail UNVERIFIED) | Copilot + 5 vendors' agents | Org governance control plane | No | Proprietary | Copilot subscription | Announced 2025-10 |
| Devin [S55][S56] | Vendor VM; Devbox in customer VPC | Devin | ≤100 members, multiplayer, SCIM/RBAC | Partial | Proprietary | Teams $80/mo | GA |
| Factory [S57][S58] | Local or vendor cloud computer | Droids, BYO inference | ≤10 seats Team; audit, sub-orgs | Enterprise on-prem options | Proprietary | Tiered | GA |
| T3 Code [S61] | Local worktrees (user report 50477897 [S29]) | Claude Code, Codex, Antigravity, OpenCode, Cursor, Grok | None found | Yes | MIT | Free, BYO subscription | 75k★, since 2026-02 |
| Orca [S63]–[S65] | Worktree per agent; machine plugins | 10+ agent CLIs | Slack/Linear/GitHub bots, mobile | Yes | MIT | UNVERIFIED | 135k★, since 2026-03, YC |
| Paseo [S66][S67] | Daemon on own machines | 35+ providers | Paseo Hub teams + triggers | Yes | Apache-2.0 | Hub pricing UNVERIFIED | 25k★, since 2025-10 |
| Herdr [S68][S69] | Own machines via SSH/Tailscale | Agent CLIs | None found | Yes | Apache-2.0 | Free | 53k★, since 2026-03 |
| Paradev [S70][S71] | Worktree per task; cloud workspace | Multiple CLIs | Team cloud 150 h/seat; Enterprise SAML/SCIM/audit/SOC 2 | Local yes; cloud UNVERIFIED | MIT | $0 / $20 / $60 / $200 per user / Enterprise | 17k★, since 2025-10 |
| Emdash [S72][S73] | Local; own infra or vendor cloud | ACP agents | SSO, roles, spend limits, shared secrets | Claimed | Apache-2.0 (Classic) | UNVERIFIED | 6k★, YC |
| Vibe Kanban [S74][S75] | Local worktrees | Multiple | Removed | Local only | Apache-2.0 | Free | Company shut down |
| Claude Squad [S78] | tmux + worktree | Claude Code, Codex, OpenCode, Amp | None | Yes | AGPL-3.0 | Free | 9k★ |
| Terragon [S76] | Cloud container per agent | Claude Code, Codex, Amp, Gemini | n/a | Snapshot only | Apache-2.0 | n/a | Shut down 2026-01 |
| boxes.dev [S80] | Cloud Linux VM per agent | Claude Code, Codex | Slack/Linear/API/schedule triggers | No | Proprietary | UNVERIFIED | YC |
| Superconductor [S81] | Metered cloud computer | Claude, ChatGPT, Grok plans + 8 key types | 4 / 40 / unlimited members included | No | Proprietary | Credits, no per-seat fee | UNVERIFIED |
| Coder [S82]–[S85] | Self-hosted workspace | Own agent + CLIs (list UNVERIFIED) | AI governance (Premium) | Yes | AGPL-3.0 Community | Free / per-user Premium | 17k★ |
| Ona [S86] | Governed background agents | Ona, Codex, others | Enterprise governance | UNVERIFIED | Proprietary | UNVERIFIED | UNVERIFIED |
| Orkes / Temporal [S95][S96] | n/a (workflow tasks) | Any via code | RBAC, audit | Yes | Apache / MIT | Cloud + OSS | Mature; different category |

### 4. Who is building their own conductor

**Agent vendors (each ships its own surface; see 3a)**
- Every major agent vendor ships its own surface (3a): Anthropic [S35][S36], OpenAI [S40][S42], Cursor [S46][S47], Google [S50][S52], Cognition [S55], Factory [S58].
- GitHub's Agent HQ is the one cross-vendor native surface: it hosts other vendors' agents under GitHub governance [S53].
- The Conductor team joined Meta and says it is building Conductor "here at Meta" [S22][S26].

**Dev-tool and platform companies**
- Coder sells self-hosted Coder Agents/Tasks with AI governance in its Premium tier [S82][S83][S84].
- Ona sells governed background agents to enterprises. Its "Background Agents Summit" lists talks from Stripe, Uber, Harvey, Cloudflare and Genentech [S86].
- Linear makes agents workspace users, and third-party agents can be distributed to other Linear orgs [S101].

**Companies that built in-house**
- Stripe "Minions": homegrown unattended one-shot agents that start from Slack and end in a PR; over 1,000 PRs merged per week. Stated reason for building: a huge Ruby/Sorbet codebase and homegrown libraries [S98].
- Ramp: "Why We Built Our Own Background Agent" (Inspect). Now offered to outside teams via early access (page meta description only; body not retrieved) [S99].
- Spotify "Honk": a background coding agent wired into Fleet Management, with 1,500+ merged PRs [S100].
- HN founder claims (weak evidence): talyn.dev "used internally at posthog" (50395930); a large observability vendor's internal "Studio" (50372045) [S24].
- Conductor's API was used to build "mobile apps, UI clients, and internal tools, like Slackbots" [S15].

**Vertical startups exposing orchestration to their own customers (B2B2B)**
- No primary evidence found of an agent-product or vertical startup embedding a third-party conductor for its own end users.
- Closest signals are building blocks, not purchases: Vercel's coding-agent template [S90], open-swe [S89], Linear's agent platform [S101], and sandbox partners that host Cursor workers [S47].

### 5. Customer segments (the three VISION.md audiences)

**Audience 1: startups building agent products (B2B2B)**
- For: a user asked for a product matching "cloud webhooks + workspace/session API + isolated VMs" (50370446) [S24]; Conductor API users built UI clients [S15]; sandbox infrastructure is a market of its own (E2B, Daytona, Modal, Vercel host Cursor workers) [S47].
- Against: no buyer found; sandbox vendors (E2B, Vercel Sandbox) and agent SDKs already serve builders [S91][S90]; Daytona's main repo is archived [S92] (meaning UNVERIFIED).
- Onboard-and-customize would mean: a headless API/SDK first; a tenant per end customer with isolated secrets and quotas; white-label UI; per-tenant metering; pluggable sandbox providers.

**Audience 2: engineering teams of 10–500**
- For: team plans existed or exist at $50–80 (Conductor Teams $60/user [S19], Paradev Team $60/user [S71], Devin Teams $80/mo [S55]). One team had bought Conductor Cloud for an org (50370063) [S24]. The most-cited loss was team or cloud features: Cloud, API, multi-machine (50370446, 50369888) [S24].
- Against: strong free OSS options (T3 Code, Orca, Herdr, Paseo, 25k–135k★); vendor desktops seen as substitutes (50374171) [S24]; Vibe Kanban's owner "couldn't find a business model" [S75]; Terragon shut down [S76]; "clearly no moat" (50374681) [S24]; and the "stranded Conductor users" wedge closed when Conductor reversed course [S27].
- Onboard-and-customize would mean: import `.conductor/settings.toml`/`conductor.json` and existing worktrees, as Paradev did [S30]; shared setup scripts, env and secrets; team prompts; SSO; review gates; first parallel PR within the hour VISION promises.

**Audience 3: platform/DevEx teams at larger companies**
- For: Stripe, Ramp and Spotify built their own [S98][S99][S100]. Vendors are adding customer-side execution (Claude self-hosted environments [S35], Cursor Self-Hosted Machines [S47], Devin VPC [S56], Factory on-prem [S57]). Conductor had planned a self-hosted cloud for Enterprise [S17]. Coder and Ona already sell governed agent platforms [S84][S86].
- Against: the in-house builders cite codebase-specific needs [S98]; incumbents (Coder, Ona, GitHub Agent HQ) already sell to this buyer [S84][S86][S53]; vendor self-hosted modes keep the agent loop or "brain" in the vendor cloud [S47][S56], so a neutral control plane may need vendor cooperation.
- Onboard-and-customize would mean: deploy into the customer's VPC or Kubernetes; plug into GitHub Enterprise/GitLab, CI, IdP and secrets; policy-as-code gates and audit export; several vendors' agents side by side; fleet-wide batch changes (the Spotify Fleet Management pattern [S100]).

## Analysis

- **The local conductor is commoditised.** Confidence HIGH, because there are 20+ OSS tools (§3b), a public rebuild in 3 days [S31], a dozen founders pitching in one HN thread, and two shutdowns (Vibe Kanban's company, citing free users; Terragon) [S75][S76].
- **Vendors are converging on the same feature set.** Confidence HIGH, because Claude, Codex, Cursor, Devin and Factory all ship parallel sessions, isolated workspaces, review/PR flow and cloud execution (§3a). Users call Conductor "no different from Claude Code Desktop" [S24].
- **VISION's premise needs two corrections.** Confidence HIGH, because these are primary sources. (1) Conductor did not shut down; it reversed course 2026-11-12/13 [S25][S27]. (2) Not every vendor surface is vendor-locked: GitHub Agent HQ is cross-vendor [S53], and Claude and Cursor offer customer-side execution [S35][S47].
- **"Companies have to build from scratch" is only partly true.** Confidence MED, because Coder, Ona and OSS tools exist [S84][S86][S63], yet Stripe, Ramp and Spotify still built their own [S98][S99][S100]. Off-the-shelf fit at scale looks poor (inference).
- **Gaps that remain uncovered** (MED, inferred from features observed across §3):
  - a self-hosted control plane that is cross-vendor and open in its core;
  - declarative per-repo gates and audit available in the open edition (Coder puts governance in Premium [S84]);
  - a headless API comparable to Conductor's [S20].
- **Workflow engines are a different category.** Confidence HIGH, because Orkes and Temporal orchestrate durable task graphs [S95][S96]. They have no per-agent checkout, diff review or merge surface, so they are a possible substrate (matches the VISION non-goal).
- **"Conductor" is a contested name.** Confidence HIGH: Netflix/Orkes Conductor [S93][S94], Superconductor [S81] and HN confusion [S24].

## Recommendations

**Options: which audience baton should design for first**

- **O1: Audience 1 (B2B2B agent-product startups) first.**
  - Pros: API-first fits VISION; little direct competition in an embeddable conductor.
  - Cons: [ADVISORY] no demand evidence found (§4, §5); [ADVISORY] needs multi-tenant, white-label and metering work before any value; [ADVISORY] sandbox vendors are adjacent and could move up the stack.
- **O2a: Audience 2, positioned as the home for "Conductor refugees".**
  - Pros: a named migration moment; import tooling exists as precedent [S30].
  - Cons: [BLOCKING] the premise is false, because Conductor reversed its shutdown and is shipping again [S27][S3]; [ADVISORY] early movers report having already moved to T3 Code, Paseo, Orca [S24].
- **O2b: Audience 2, positioned on team governance (shared, auditable, policy-gated fleets).**
  - Pros: demonstrated willingness to pay about $50–80 per seat [S19][S71][S55]; OSS install as the funnel.
  - Cons: [ADVISORY] crowded with free tools and vendor bundles; [ADVISORY] documented monetisation failures [S75][S76].
- **O3: Audience 3 (platform/DevEx, self-host + SSO + audit + per-repo policy) first.**
  - Pros: the strongest build-own evidence [S98]–[S100]; vendors are adding customer-side execution [S35][S47][S56]; Conductor itself planned this [S17]; it matches "self-host first".
  - Cons: [ADVISORY] Coder, Ona and GitHub Agent HQ incumbents; [ADVISORY] vendor loops stay vendor-hosted [S47][S56]; [ADVISORY] long enterprise cycles.
- **O4: All three audiences at once.**
  - Pros: maximal market.
  - Cons: [ADVISORY] unfocused, and O1's multi-tenancy needs conflict with O3's single-org hardening for an early team.
- **O5: Do not build; adopt or contribute to an existing OSS conductor** (Orca, Paseo, Herdr, Coder Community).
  - Pros: zero build cost.
  - Cons: [ADVISORY] none found that documents open-edition declarative gates plus audit plus a cross-vendor self-hosted control plane (absence of evidence, so ADVISORY not BLOCKING).

**Verdict (from BLOCKING items alone):** only O2a is eliminated. O1, O2b, O3, O4 and O5 carry no BLOCKING con and remain viable. The preference below rests on ADVISORY items only.

**Advisory preference (overrulable):** make O3's needs the design centre (self-hosted control plane, cross-vendor adapters, policy-as-config, audit). Ship it as a single-binary OSS install so that O2b teams are the adoption funnel. Treat O1 as a hypothesis to test with interviews before building multi-tenant or white-label features.

**Per-audience verdicts**
- Audience 3 platform/DevEx: **real** — MED, because there is strong build-own and vendor-investment evidence, but the builders built rather than bought.
- Audience 2 engineering teams: **real but contested** — MED, because people pay per seat, but free OSS and vendor bundles cap the price and the refugee wedge is gone.
- Audience 1 B2B2B startups: **not evidenced** — LOW, because no primary evidence was found and search tooling was degraded, so absence is weak.

## Open questions

- Did Conductor Cloud actually go read-only, or delete any data, before the reversal? Have sign-ups and paid tiers reopened, and on what terms under Meta?
- What are real usage numbers (downloads, MAU) for T3 Code, Orca, Herdr and Paseo? Stars are not usage.
- Will Anthropic, OpenAI and Cursor let third-party control planes drive their self-hosted runners, and on what subscription or ToS terms? Conductor already hit one such policy [S13].
- Is there any B2B2B buyer? This needs interviews, because no public evidence was found.
- What are the prices for Coder Premium AI governance, Ona, Orca, Paseo Hub, Emdash and boxes.dev? These were not retrieved.
- Reddit and X sentiment beyond the founder posts was not collected (tooling).

## Handoffs to other docs

- **VISION.md (maintainer):** correct the thesis paragraph, because Conductor reversed its shutdown [S27]. Soften "every agent vendor ... vendor-locked" given GitHub Agent HQ [S53] and "have to build from scratch" given Coder, Ona and OSS tools [S84][S86].
- **BRIEF.md (maintainer):** the "What we already know" block is superseded by the reversal [S27]. It also contains founders' real names, which conflicts with its own rule 5.
- **R2 technical landscape:**
  - isolation models (worktree vs container vs microVM vs full devbox) [S8][S87][S17][S80];
  - customer-side execution patterns where the loop stays in the vendor cloud (Claude runners [S35], Cursor workers [S47], Devin Devbox [S56]);
  - ACP adoption (Emdash) [S73];
  - Conductor's API and SQL-over-transcripts design [S20];
  - workflow engines as a substrate [S95][S96].
- **R3 business case:**
  - monetisation failures [S75][S76];
  - price anchors: $50–80 per seat [S19][S71][S55] and credit-based, no-seat pricing [S81];
  - licence mix: MIT/Apache dominate, AGPL at Coder and Claude Squad [S85][S78];
  - the "Conductor" name collision [S93][S81];
  - Meta-owned Conductor as a competitor with damaged trust [S24].

## Sources

- [S1] YC company profile — https://www.ycombinator.com/companies/conductor
- [S2] Show HN launch thread, 2025-07-17 — https://news.ycombinator.com/item?id=44594584
- [S3] https://www.conductor.build/changelog , https://www.conductor.build/changelog/rss.xml
- [S4] https://www.conductor.build/blog/series-a
- [S5] https://www.conductor.build/blog/cmd-joins-conductor
- [S6] https://www.conductor.build
- [S7] https://docs.conductor.build
- [S8] https://www.conductor.build/docs/faq
- [S9] https://docs.conductor.build/workflow
- [S10] https://www.conductor.build/changelog/0.18.0-codex
- [S11] https://www.conductor.build/changelog/0.63.0-cursor-support-dispatcher
- [S12] https://www.conductor.build/changelog/0.69.0-opencode
- [S13] https://www.conductor.build/blog/claude-subscription-update
- [S14] https://www.conductor.build/changelog/0.77.0-early-access-multiplayer-api-background-tasks-performance
- [S15] https://www.conductor.build/changelog/0.78.0-introducing-conductor-cloud
- [S16] https://www.conductor.build/changelog/0.85.0-sections-routines-and-a-new-model-picker
- [S17] https://www.conductor.build/docs/cloud/faq
- [S18] https://www.conductor.build/docs/cloud/cloud-computer
- [S19] Conductor pricing page, archived 2026-09-29 — http://web.archive.org/web/20260929175413/https://www.conductor.build/pricing
- [S20] https://docs.conductor.build/api
- [S21] https://www.conductor.build/docs/cloud/collaboration
- [S22] https://www.conductor.build/changelog/0.93.0-conductor-is-joining-meta
- [S23] Original announcement, archived 2026-11-06 — http://web.archive.org/web/20261106225120/https://www.conductor.build/announcement
- [S24] HN thread "Conductor is joining Meta", 2026-11-04 — https://news.ycombinator.com/item?id=50369503
- [S25] Co-founder post "CONDUCTOR IS BACK", 2026-11-12 (read via api.fxtwitter.com mirror of the same post id) — https://x.com/i/status/2120973052024598620 (id-only form of the post URL)
- [S26] Co-founder reply "we're doing it here at Meta", 2026-11-12 — https://x.com/i/status/2120975128158945374 (id-only form)
- [S27] https://www.conductor.build/changelog/0.94.2-conductor-is-back
- [S28] https://www.conductor.build/announcement
- [S29] HN thread "Announcement: Conductor Is Back", 2026-11-12 — https://news.ycombinator.com/item?id=50473888
- [S30] https://docs.paradev.com/migrate-from-conductor
- [S31] Show HN rebuild, 2026-11-07 — https://news.ycombinator.com/item?id=50407925
- [S32] https://code.claude.com/docs/en/worktrees
- [S33] https://code.claude.com/docs/en/agent-teams
- [S34] https://code.claude.com/docs/en/claude-code-on-the-web
- [S35] https://code.claude.com/docs/en/self-hosted-environments
- [S36] https://code.claude.com/docs/en/desktop
- [S37] https://code.claude.com/docs/en/claude-ship
- [S38] https://code.claude.com/docs/en/common-workflows
- [S39] https://github.com/anthropics/claude-code
- [S40] https://developers.openai.com/codex/cloud
- [S41] https://developers.openai.com/codex/cloud/environments
- [S42] https://developers.openai.com/codex/app/worktrees
- [S43] https://developers.openai.com/codex/enterprise
- [S44] https://developers.openai.com/codex/pricing
- [S45] https://github.com/openai/codex
- [S46] https://cursor.com/docs/cloud-agent
- [S47] https://cursor.com/docs/cloud-agent/self-hosted
- [S48] https://cursor.com/pricing
- [S49] https://geminicli.com/docs/core/subagents
- [S50] https://geminicli.com/docs/cli/git-worktrees
- [S51] https://github.com/google-gemini/gemini-cli
- [S52] https://jules.google/docs
- [S53] https://github.blog/news-insights/company-news/welcome-home-agents/
- [S54] https://docs.github.com/en/copilot/concepts/agents/coding-agent/about-coding-agent
- [S55] https://devin.ai/pricing
- [S56] https://docs.devin.ai/enterprise/vpc/overview
- [S57] https://factory.ai/pricing
- [S58] https://docs.factory.ai/web/getting-started/overview
- [S59] https://github.com/anomalyco/opencode
- [S60] https://github.com/OpenHands/OpenHands
- [S61] https://github.com/pingdotgg/t3code
- [S62] https://t3.codes
- [S63] https://github.com/stablyai/orca
- [S64] https://www.onorca.dev
- [S65] https://www.onorca.dev/plugins?kind=machines
- [S66] https://github.com/getpaseo/paseo
- [S67] https://paseo.sh
- [S68] https://github.com/herdrdev/herdr
- [S69] https://herdr.dev
- [S70] https://github.com/ParadevAI/Paradev
- [S71] https://paradev.com/pricing
- [S72] https://github.com/generalaction/emdash
- [S73] https://emdash.sh
- [S74] https://github.com/BloopAI/vibe-kanban
- [S75] Vibe Kanban shutdown post, archived — http://web.archive.org/web/20261027005909/https://www.vibekanban.com/blog/shutdown
- [S76] https://github.com/terragon-labs/terragon-oss
- [S77] https://github.com/stravu/crystal
- [S78] https://github.com/smtg-ai/claude-squad
- [S79] Long tail — https://github.com/simion/termic , https://github.com/get-bb/bb , https://github.com/imbue-ai/sculptor , https://github.com/juggler-ai/juggler , https://github.com/ouijit/ouijit , https://github.com/silo-code/silo , https://github.com/bestboyhq/malini
- [S80] https://boxes.dev
- [S81] https://superconductor.com/pricing
- [S82] https://coder.com/docs/ai-coder
- [S83] https://coder.com/docs/ai-coder/tasks
- [S84] https://coder.com/pricing
- [S85] https://github.com/coder/coder
- [S86] https://ona.com
- [S87] https://github.com/dagger/container-use
- [S88] https://github.com/coder/agentapi
- [S89] https://github.com/langchain-ai/open-swe
- [S90] https://github.com/vercel-labs/coding-agent-template
- [S91] https://github.com/e2b-dev/E2B
- [S92] https://github.com/daytonaio/daytona
- [S93] https://github.com/Netflix/conductor
- [S94] https://github.com/conductor-oss/conductor
- [S95] https://orkes.io
- [S96] https://temporal.io/solutions/ai
- [S97] https://github.com/temporalio/temporal
- [S98] https://stripe.dev/blog/minions-stripes-one-shot-end-to-end-coding-agents
- [S99] https://builders.ramp.com/post/why-we-built-our-background-agent
- [S100] https://engineering.atspotify.com/2025/11/spotifys-background-coding-agent-part-1
- [S101] https://linear.app/developers/agents
