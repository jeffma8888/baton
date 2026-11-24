# Review: R3-business-case.md v2

> Reviewer: Opus · review R3 (independent; a different model from the author). Date: 2026-11-23.
> Scope: source integrity (43 table rows covering R3's own citations, some grouping several registry URLs, plus 3 reviewer-added sources), reasoning, naming re-check, PR-FAQ, experiments/metrics, length, rule 5.
> Method: `web_fetch` and `curl` against the cited URLs, registry JSON APIs, GitHub REST API, RDAP/`whois`, and the USPTO trademark search backend (`https://tmsearch.uspto.gov`) plus TSDR (`https://tsdr.uspto.gov/statusview/sn<serial>`). R3 not edited; git not run.

## Verdict

**REVISE.**

The Facts base is strong: 35 of the 43 sampled citation rows fully support their claims, 5 partly do, 1 supports the facts but not the inference R3 draws, 1 mismatches and 1 could not be read. None is fabricated. The verdicts are mostly pointed the right way. Eight findings block approval:
- (a) The top naming candidate collides with a live product in the same category.
- (b) The press release makes a factual claim that R1 and the brief already corrected.
- (c) The CTO FAQ omits "why not Coder" and misframes "why not Agent HQ".
- (d) The GitHub Agent HQ facts are 13 months stale.
- (e, f) Two BLOCKING labels in the options analysis fail rule 4: one is mislabelled, one is applied inconsistently.
- (g) The "empty self-hosted server slot" claim contradicts R1.
- (h) No pre-build experiment can falsify the payer thesis.

All eight have bounded fixes. None requires new research beyond what this review already verified.

## Findings

### Naming

1. **[BLOCKING] §E, §5, Recommendations: `ripieno` collides with a live product in the same category.**
   - A GitHub repository named `ripieno`, created 2026-08-08, describes itself as "A self-hosted shared workspace where several people and several AI agents work on one codebase … Everyone brings their own: a Claude Code or Codex subscription, an API key …, or a local model via Ollama" (https://github.com/ijmh2/ripieno; metadata via https://api.github.com/repos/ijmh2/ripieno).
   - Its homepage reads "Ripieno — Your agents work better together … MIT open source · Preview software" (https://ripieno.vercel.app).
   - This is the same collision type R3 used against `baton` (`batonhq`). It probably also explains the ripieno.dev registration of 2026-07-15 (inference, `UNVERIFIED`).
   - R3 checked registry names and GitHub *handles* but not repository or product names.
   - Fix:
     - Demote or drop `ripieno`.
     - Add a GitHub repository search (`https://api.github.com/search/repositories?q=<name>+in:name`) and a product-name web check to the §5 method for every candidate.
     - Re-issue the HIGH naming recommendation with `cuelist`/`kapell` or new candidates.

### PR-FAQ

2. **[BLOCKING] §F press release, paragraph 2: a factual claim that R1 and the brief already corrected.**
   - "Each vendor shipped a conductor that works only with its own agent" contradicts R3's own §4 (GitHub Agent HQ is multi-vendor), R1 Analysis correction (2) and the brief ("GitHub Agent HQ is a cross-vendor orchestration surface"). GitHub's docs fetched today list Anthropic Claude and OpenAI Codex as third-party agents in public preview (https://docs.github.com/en/copilot/concepts/agents/about-third-party-agents).
   - In the same paragraph:
     - "was acquired by a large platform company" goes beyond the primary wording ("Conductor is joining Meta", https://conductor.build/changelog; "doing it here at Meta", R1 S26). The deal structure is `UNVERIFIED`.
     - "choosing between lock-in, … and home-grown worktree scripts" ignores Coder and the self-hostable open-source tools in R1 table 3e.
   - Fix: "Most vendors shipped a conductor for their own agent; the one cross-vendor surface lives inside a single code host's cloud." Replace "acquired" with "joined". Name the real gap: open core, self-hosted control plane, gates and audit in the open edition.

3. **[BLOCKING] §F FAQ: "Why not Coder?" is missing, and Q1 dodges Agent HQ.**
   - Coder is R3's own HIGH-rated overlap for the paying audience. Verified on https://coder.com/pricing:
     - Premium: "Audit prompts, restrict access, and manage costs so agents operate within your policies", "3rd-party AI agent compatibility (e.g., Claude, Codex)" and "Customizable branding".
     - AI Premium: "Run Coder Agents natively in your control plane, with no vendor SaaS or third-party agent wrapper".
   - A platform-team CTO asks this first.
   - Q1's answer, "If your fleet is one vendor's agent … you probably should", answers the Cursor and vendor-cloud questions, but not Agent HQ, which is multi-vendor.
   - Fix:
     - Split Q1. For Agent HQ: "If you are GitHub-only and accept a GitHub-hosted control plane, use it. [Name] is for non-GitHub or mixed hosts, a control plane inside your boundary, air-gap, and an open core."
     - Add a Coder question: "Coder is the workspace layer; [Name] is the task → gate → merge layer above it and runs on Coder as a provider." Be honest that AI Premium overlaps.
     - Stay inside five questions by merging Q4 into Q2 (both are about trust and relicensing).

### Competitive facts and reasoning

4. **[BLOCKING] §4 (Agent HQ bullet), §C, Open questions: the Agent HQ facts are stale.**
   - The Agent HQ post is undated in R3. It was published 2025-10-28 (`article:published_time` on https://github.blog/news-insights/company-news/welcome-home-agents/).
   - It already said "Try mission control today", yet R3's open question asks "when 'mission control' and the agent control plane ship".
   - §C rates "MED on timing — 'over the coming months'" from a quote that is 13 months old.
   - The current docs show third-party agents in public preview, enabled through organisation policies and an enterprise's "AI controls". The blog's control plane already promises "security policies, audit logging … control which agents are allowed". That is baton's gate/audit pitch.
   - Fix:
     - Date the bullet.
     - Add the docs fact.
     - Re-rate the timing as "shipping (public preview)".
     - Rewrite the open question as "will Agent HQ support non-GitHub hosts or a customer-hosted control plane?"

5. **[BLOCKING] §D option 3: a BLOCKING label that is neither an invariant nor physics (rule 4).**
   - "Cannot be first: a white-label edition is a packaging of an engine that does not yet exist" applies equally to option 1. Every option builds the engine first.
   - "Headless API first" is a coherent wedge. R1 §5 spells it out: "a headless API/SDK first; a tenant per end customer …".
   - The real cons (no demand evidence; multi-tenancy before the core) are ADVISORY and already listed.
   - Fix: relabel ADVISORY. The verdict becomes "options 2 and 5 are eliminated by BLOCKING items; 1, 3 and 4 survive; 1 is preferred on ADVISORY grounds." The recommendation does not change.

6. **[BLOCKING] §B licence options: the invariant is applied inconsistently (rule 4).**
   - Sustainable Use is BLOCKED because the VISION invariant lets a company offer baton "to its customers". Options 4 and 5 prohibit exactly that use but are only ADVISORY:
     - **FSL** permits any purpose "other than a Competing Use", defined as "making the Software available to others in a commercial product or service that … offers the same or substantially similar functionality as the Software" (https://fsl.software/FSL-1.1-ALv2.template.md).
     - **ELv2**: "You may not provide the software to third parties as a hosted or managed service …" (https://www.elastic.co/licensing/elastic-license).
   - §D option 5 also uses "open" as an invariant, while §B option 4 concedes FSL/BSL mean "the project cannot say 'open source'".
   - Fix, either way:
     - (a) Mark FSL/BSL with a competing-use grant and ELv2 `[BLOCKING]` on the same invariant. SSPL can stay ADVISORY, because it permits service offering subject to source disclosure.
     - (b) Downgrade Sustainable Use to ADVISORY, because VISION hedges "if the demand proves real".
     - State which. The Apache-2.0 recommendation survives either way.

7. **[BLOCKING] §C (desktop-tools bullet), §D verdict and option 1 pros, Recommendations (MED, Audience 2): the "empty server slot" claim contradicts R1.**
   - R3 states that the open-source tools "as desktop apps, offer no shared policy, audit or server-side control plane", that "the self-hosted server slot is empty", and "win on what none offers — shared, self-hosted, policy-gated".
   - R1 §3b/3e says otherwise:
     - Paseo: daemon on own machines, plus Paseo Hub "Teams and triggers".
     - Orca: machine plugins; self-host yes.
     - Emdash: "your own infrastructure", SSO/roles, spend limits.
     - Paradev: paid, not free ($0/$20/$60/$200), with Enterprise SAML/SCIM/audit.
   - Coder (verified above) is self-hosted, with audit and third-party agents.
   - R1's gap is narrower (MED): "a self-hosted control plane that is cross-vendor and open in its core; declarative per-repo gates and audit available in the open edition."
   - Fix:
     - Restate the differentiator as R1's gap and name the partial occupants.
     - Stop calling Paseo, Herdr and Paradev "free desktop" tools.
     - Re-justify the 1-over-4 tilt on that narrower basis.
     - Drop "proven payer" (§D option 1 pros). R1 rates the platform/DevEx audience "real — MED … the builders built rather than bought".

### Experiments and metrics

8. **[BLOCKING] §G and Recommendations (HIGH, "run the six pre-build experiments"): no experiment can falsify the payer thesis.**
   - Audience 1 (platform/DevEx) is the payer, and R1's main counter-evidence is build-versus-buy: Stripe, Ramp and Spotify built their own.
   - The six pre-build experiments sample Audience 2 (Conductor leavers/stayers, OSS issue trackers) or measure clicks.
   - The only platform-team test (5 design partners, 2 paid) runs *after* launch.
   - Fix: add a pre-build experiment with a kill threshold. For example: 10–15 discovery calls with platform/DevEx leads, half of them at organisations that already run an in-house agent harness, using a two-page spec. Pass = at least 3 sign a non-binding design-partner letter, and at least 2 name a budget line. Fail ⇒ re-scope to an OSS team server with no Enterprise plan, or stop.

9. **[ADVISORY] §G metric "Team adoption … (the Audience 2 → 3 signal)": a leftover from the old numbering.** Under VISION v0.1 numbering, Audience 3 is B2B2B. The intended signal is team → platform payer, i.e. "Audience 2 → 1". Fix the label.

10. **[ADVISORY] §G: metrics that cannot be measured or have no threshold.**
    - "≥ 40% of installs" and "50 instances" rely on an opt-in heartbeat, so the denominator is unknown. Define the denominator as opt-in instances, use downloads (container pulls, release downloads) as a reported proxy, and note the bias.
    - The fake-door test, issue-tracker mining and retention have no pass/fail thresholds. Add them.
    - The landing page needs a minimum sample size (e.g. ≥ 300 qualified visitors) before the "embed < 10%" rule fires.
    - Add a single day-90 kill/continue rule that combines activation, multi-adapter share and design-partner conversion.
    - "Each ≤ 1 week, ≤ a few hundred dollars" is optimistic for 20 recruited interviews. Budget incentives or allow two weeks.
    - "Name reservation" is an action, not an experiment. Do not count it among the six.

### B2B2B analysis

11. **[ADVISORY] §A Audience 3: the "surface is the product UX" argument is sound for the segment VISION names, but too narrow for B2B2B as a whole.** It holds for agent-product startups whose product *is* the orchestration UX. It misses embedders for whom the conductor is **not** the differentiator, i.e. where the Stripe/Auth0 condition does hold:
    - (a) code-host, forge and internal-developer-portal vendors adding an agent surface (the Roadie private-plugin model; Coder lists Backstage among its integrations);
    - (b) sandbox/compute vendors that want a reference surface to drive consumption (R1: E2B, Daytona, Modal and Vercel host Cursor workers; Vercel publishes a coding-agent template);
    - (c) model or inference vendors whose differentiator is the model (R3's own "court the vendors" point);
    - (d) IT services and consultancies delivering agent-driven development per client, with per-client tenancy and branding;
    - (e) internal platform teams serving many business units under their own brand.

    R3 also omits R1's weak headless-demand signals: Conductor API users built "mobile apps, UI clients, and internal tools" (R1 S15), and an HN user asked for "cloud webhooks + workspace/session API + isolated VMs" (R1 S24, 50370446). The "not a launch audience" verdict stands on ADVISORY grounds. But the landing-page CTA recruits from Conductor communities, so it under-samples (a)–(d) and the "embed < 10%" rule could kill Audience 3 on the wrong population. Fix: name these segments and test them with 5–10 direct conversations, with a pass threshold on letters of intent, not CTA share.

### Confidence labels

12. **[ADVISORY] Recommendations and §A: several confidence labels are higher than the evidence supports.**
    - "HIGH — Wedge" conflicts with §D's "MED that 1 beats 4".
    - "HIGH — Lead with Audience 2 as users and Audience 1 as payers" is built from MED parts (§A "MED on pull"; R1 MED for both audiences). Make it MED.
    - §A "HIGH: every cited winner fits" judges four winners that the author chose. Survivorship makes this MED.
    - "Time-to-first-value was minutes for one developer (HIGH: Clerk)": the Clerk pricing page supports the branding upsell, not "minutes". Mark `UNVERIFIED` or cite a quickstart.

### Source integrity

13. **[ADVISORY] Source integrity: minor mismatches (details in the table).**
    - §A quotes Coder as "secure environments for developers and their agents" and cites §4. The phrase is on neither page. The homepage reads "Self-hosted cloud development environments for your developers and their agents"; the pricing page reads "Choose your secure path for developers and their agents".
    - Elastic *announced* on 2024-08-29 that it "will be adding AGPL … in the coming weeks"; R3 says it "added".
    - Redis added AGPL "as an additional licensing option"; R3 says it "moved to AGPLv3".
    - Devin: "shared on-demand credits" is not on the current page, which says "Up to 100 users" rather than "members". The "pooled credits" category is unsupported.
    - The co-founder quote "CONDUCTOR IS BACK … we're doing it here at Meta" joins two separate posts (R1 S25, S26) with an ellipsis. Quote them separately.
    - Claude cloud-sessions GA covers Enterprise "premium seats or Chat + Claude Code seats" only.
    - npm `baton` was created in 2013 and last modified in 2022.
    - The name-reservation range "2026-07 to 2026-10" excludes the `tutti` crate (created 2026-03-15).
    - "Coder, Orkes and Roadie followed this order / all added hosting later" (§B, §D) is unsupported. Roadie has run "a SaaS version of Backstage since 2020", which is hosted from day one (https://roadie.io/pricing/). The Coder pricing page lists no hosted plan. Drop the claim or cite a source.

14. **[ADVISORY] Rule 1: "per R1" claims carry no URL.**
    - The trust quote, the co-founder posts, the OSS destinations, the Enterprise self-host status and the pricing tiers are cited as "per R1 S25-S27" or "per R1 §1". Copy the URLs inline: R1 S17, S19 (archived pricing), S23, S24, S25, S26, S29.
    - PR-FAQ Q2 ("data would be deleted in sixteen days") rests on R1 S23 (archived announcement: "data deletion begins"). The changelog R3 cites says only "shuts down on November 20". Cite S23.
    - The Anthropic Agent SDK policy reaches R3 only through Conductor's blog, which relays it. Label it as Conductor's account, or cite Anthropic's own notice.

15. **[ADVISORY] §C: the Meta-owned Conductor threat assessment is coherent with R1 but omits R1's sharpest differentiators.**
    - These R1 facts are exactly where baton is different, and R3 leaves them out:
      - Conductor Cloud is GitHub-only ("GitLab, Bitbucket … Not yet") and stores chat transcripts on Conductor servers (R1 S17).
      - In local mode, "agents run directly on your system without sandboxing" (R1 S8).
      - The API manages cloud workspaces only (R1 S20).
    - "Meta-owned" and "Meta-funded" are the brief's framing. The primary sources say "joining Meta" and "here at Meta". Label ownership and funding as inference.

16. **[ADVISORY] §F: overclaims in the FAQ answers.**
    - Q3, "Every adapter supports API-key and enterprise billing and self-hosted models": a vendor CLI adapter cannot make a proprietary agent use a self-hosted model. Q5's air-gapped mode with "models all run inside your boundary" works only with open harnesses and local models. Scope both claims.
    - Q4, "We will not relicense the core": add the mechanism that makes the promise credible, i.e. contributions under DCO with no CLA, so the core cannot be relicensed unilaterally. n8n's FAQ shows the opposite case: its CLA "gives n8n the ability to change its license without seeking additional permission" (https://docs.n8n.io/sustainable-use-license/).
    - "Remains proprietary with no self-hosted option" in a release dated 2027 is a prediction. Hedge it ("at the time of writing").
    - Mark the design-partner quote "[illustrative]".

17. **[ADVISORY] §B option 3: "incompatible with CNCF/LF-style neutral governance" overstates the case.** The Linux Foundation hosts GPL projects, the kernel among them. The constraint is CNCF-specific. Narrow the claim and cite the CNCF charter or IP policy.

18. **[ADVISORY] §D options 2 and 5: the BLOCKING items rest on VISION design principles that VISION itself marks "to be challenged in HLD-000".** Say that these eliminations are conditional on VISION v0.1, so that a later change of principle visibly reopens them.

### Naming (continued)

19. **[ADVISORY] §5 and §E: the `baton` trademark status can now be resolved, and `cuelist` is a generic term.**
    - The USPTO search backend returns 339 records for the word mark BATON. Live marks in software classes include:
      - BATON, Reg. 6418232, IC 042, "software for project management and collaboration" SaaS. Registered 2021-07-13, class status ACTIVE (https://tsdr.uspto.gov/statusview/sn90272389, verified).
      - BATON, Reg. 5388392 (Baton Systems, IC 009/042).
      - BATON, Reg. 4888336 (Interra Systems, IC 009).
      - BATON CORE, Reg. 7153516.
      - A pending BATON application (SN 99679424) in IC 009/042.
    - Replace "Trademark status `UNVERIFIED`" for `baton` with these. This strengthens the HIGH "do not ship as baton".
    - The same backend returns **0 records** for RIPIENO, CUELIST, "CUE LIST" and KAPELL (exact phrase), and 2 non-live software records for KAPELLMEISTER. This is US only. EUIPO and UKIPO remain unchecked.
    - "Cue list" is a term of art in show control. GitHub lists 17 `cuelist` repositories (QLab, ETC Eos, Companion tooling), which supports R3's weak-distinctiveness con.

### Length

20. **[ADVISORY] Length: about 5,717 words with URLs stripped (reviewer count), against a 5,000 budget.** Cuts that lose no verified fact:
    - **§4 Facts (620 w):** replace the Conductor homepage and pricing bullets with a pointer to R1 §1 plus the URLs (about −70). Merge the "Name reservations" bullet into §5 "Rejected on checks" (about −40).
    - **§A (567 w):** the Audience 2 paragraph restates §4. Cut it to three sentences (about −100).
    - **§B (547 w):** "Pricing shapes observed" repeats §3 and the pricing recommendation. Keep one line (about −60).
    - **§C (483 w):** cut the threat map to one line per threat plus its confidence, without re-quoting §4. Drop survival bullet 4, which duplicates PR-FAQ Q2 and the LOW recommendation (about −150).
    - **§D (379 w):** the option 1 pros restate §A, and the verdict sentence is repeated in Recommendations (about −50).
    - **Recommendations (351 w):** cut each parenthetical to the one-clause reason rule 3 requires (about −120).
    - **Handoffs (136 w):** drop the request that R1 catalogue licence and server mode. R1 table 3e already has both (about −50).
    - **Repetition:** the "nine-day shutdown" narrative appears in 7 lines. Keep it in §4 and one positioning bullet (about −60).

    Total about −700. That leaves room for the "why not Coder" FAQ and the payer experiment (findings 3 and 8).

### Handoffs

21. **[ADVISORY] Handoffs: the R1 handoff partly duplicates R1.** R1 table 3e already has licence and self-host status for T3 Code, Paseo, Orca, Herdr and Paradev. Keep only the asks R1 lacks: the name-reservation projects, Coder Agents depth, and issue-tracker mining.

### Rule 5

22. **[ADVISORY] Rule 5: the doc passes; tone is the only note.**
    - No real personal names, operator employer or internal URLs were found. The Series A customer list is correctly not reproduced.
    - For vendor-neutral tone, the competitive lever should rest on verifiable facts: proprietary, no self-host today, GitHub-only cloud. "Use the incumbent's … shutdown episode as the durable positioning lever" (Recommendations) and "got acquired, told me my data would be deleted" (Q2) read as a jab in a public repo. Keep Q2's honest answer; neutralise the framing.

## Source spot-check table

Fetched 2026-11-23 by the reviewer. S = supported; P = partly supported or overstated; F = failed or mismatched; N/A = could not be read.

| # | R3 § | URL | Result | Note |
|---|---|---|---|---|
| 1 | §2 | https://www.linuxfoundation.org/press/announcing-opentofu | S | 2023-09-20; 140+ orgs, 600+ individuals, ≥18 full-time developers for ≥5 years. |
| 2 | §2 | https://www.hashicorp.com/blog/hashicorp-adopts-business-source-license | N/A | HTTP 429 again, matching R3's disclosure. |
| 3 | §2 | https://www.elastic.co/blog/elasticsearch-is-open-source-again | P | 2024-08-29; "painful … it worked", fork, market confusion all present; AGPL "will be adding … in the coming weeks", not "added". |
| 4 | §2 | https://redis.io/blog/agplv3/ | P | SSPL March 2024, two hyperscaler forks, "not truly open source" present; AGPL "additional licensing option", not a move. |
| 5 | §2/§B | https://fsl.software/ + licence text | S | Converts after two years; "harmful free-riding". Licence defines "Competing Use" (finding 6). |
| 6 | §2/§B | https://docs.n8n.io/sustainable-use-license/ | S | Quotes verbatim; FAQ also confirms the CLA enables relicensing (finding 16). |
| 7 | §2 | https://about.gitlab.com/company/stewardship/ | S | "open core … almost all our revenue with subscriptions"; "source-available features". |
| 8 | §1 | https://github.com/Netflix/conductor | S | "Effective December 13, 2023 … internal Conductor fork". |
| 9 | §1 | https://github.com/conductor-oss/conductor | S | "durable execution platform for microservices, AI agents"; "maintained by Orkes"; MCP, human approval; Apache 2.0. |
| 10 | §1 | https://orkes.io/pricing | S | Playground not for production, individual; Enterprise in your cloud account, 99.99% SLA, TAM. |
| 11 | §1/§B | https://roadie.io/pricing/ | S / F | Facts supported (SaaS since 2020, contributing users, private plugins, SOC2 Type 2); contradicts "Roadie added hosting later". |
| 12 | §1/§4 | https://coder.com/pricing | S | Community runs Claude Code/Codex/Cursor; Premium "Annually per user", prompt audit, RBAC, OIDC sync, HA, SLA; AI Premium items as stated. Also lists "Customizable branding". No hosted plan listed. |
| 13 | §1 | https://raw.githubusercontent.com/coder/coder/main/LICENSE.enterprise | S | "may not move, change, disable, or circumvent the license key functionality". |
| 14 | §A | https://coder.com/ | F | Quoted phrase not present; actual: "Self-hosted cloud development environments for your developers and their agents." |
| 15 | §1 | https://github.com/sourcegraph/sourcegraph-public-snapshot | S | Private monorepo; last Apache commit; "primarily non-OSS-licensed files". |
| 16 | §1 | https://sourcegraph.com/pricing | S | Credits per user, org-wide pooling, single-tenant cloud / self-hosted, "Works with Claude Code, Cursor, Codex, Amp". |
| 17 | §1 | https://stripe.com/connect/pricing | S | Both models; 0.25% + 25¢ per payout; revenue share. |
| 18 | §1 | https://clerk.com/pricing | S | $0/$25/$300; $0.02 MRU over 50,000; "Remove Clerk branding" paid. |
| 19 | §1 | https://auth0.com/pricing | S | Up to 25,000 MAU free; Essentials/Professional; Private Deployment add-on ("AI Identity" not checked). |
| 20 | §3 | https://devin.ai/pricing | P | $80, up to 100 users, SAML/OIDC, 2 concurrent free-mode sessions, SCIM/RBAC, dedicated deployment present; "shared on-demand credits" not found. |
| 21 | §3 | https://vercel.com/docs/vercel-sandbox/pricing | S | $0.128/CPU-h, $0.0212/GB-h, $0.60/1M creations, 24 h, 10,000 concurrent. |
| 22 | §3 | https://docs.github.com/en/billing/concepts/product-billing/github-codespaces | S | $0.18/$0.36/$0.72/$1.44 per hour. |
| 23 | §3 | https://e2b.dev/pricing | S | Hobby $100 one-time, 1 h, 10 concurrent; Pro $150 + per-second, 24 h, 100 → 1,100. |
| 24 | §3 | https://www.daytona.io/pricing | S | $0.0504 vCPU-h, $0.0162 GiB-h, $0.000108 storage, $200 free. |
| 25 | §3 | https://temporal.io/pricing | S | $50/M actions, down to $25, $0.042/GB-h, "Greater of $500/mo or 10% of usage". |
| 26 | §3/§4 | https://www.anthropic.com/news/claude-code-on-the-web | S | GA 2026-09-23 (Enterprise only with premium or Chat + Claude Code seats); parallel, sandbox, git proxy, auto PR. |
| 27 | §3 | https://www.anthropic.com/pricing | S | Pro $17/month with annual subscription. |
| 28 | §4 | https://cursor.com/blog/2-0 | S | "many agents in parallel … git worktrees or remote machines"; multiple models improve results. |
| 29 | §4 | https://antigravity.google/ | S | "command center to manage multiple local agents in parallel"; background subagents; SDK; "Available at no charge". |
| 30 | §4 | https://conductor.build/ | S | "Run a team of coding agents in the cloud"; isolated microVMs; first-party agents "under the hood"; iOS/API. |
| 31 | §4 | https://conductor.build/blog/series-a | S | 2026-03-30, $22M, Spark and Matrix, 10x since January, "100% of their code", 12 named companies "dozens of engineers", "phone to your VPC". |
| 32 | §4 | https://conductor.build/blog/claude-subscription-update | S | May 13 announcement, June 15 "delayed indefinitely", "so that policy applied to us". This is Conductor's account of Anthropic's notice. |
| 33 | §4 | https://conductor.build/blog/sign-in-with-chatgpt | S | 2026-09-29, "partnering with OpenAI", bring ChatGPT plan to Codex. |
| 34 | §4 | https://conductor.build/changelog | S | 0.94.2 keeps the wind-down text and adds "Update: Conductor isn't shutting down. Conductor is back." and the banner removal; 0.99.1; Opus 5.5 / GPT-6.2 Sol defaults. |
| 35 | §4/§C | https://github.blog/news-insights/company-news/welcome-home-agents/ | P | Quotes accurate; publish date 2025-10-28 omitted; "Try mission control today" contradicts R3's open question (finding 4). |
| 36 | (added) | https://docs.github.com/en/copilot/concepts/agents/about-third-party-agents | n/a | Reviewer-added: Claude and Codex third-party agents "currently in public preview"; enterprise "AI controls". |
| 37 | §5 | https://github.com/ConductorOne/baton | S | Apache-2.0; "Identity governance for any SaaS, infrastructure, or tool". |
| 38 | §5 | https://github.com/openbaton | S | Organization "Open Baton", 43 public repos. |
| 39 | §5 | https://registry.npmjs.org/baton, https://pypi.org/pypi/baton/json, https://crates.io/api/v1/crates/baton | S | All 200 (taken). npm "An orchestration tool powered by conservatory", created 2013, modified 2022. |
| 40 | §4/§5 | https://registry.npmjs.org/batonhq | S | "Hand a unit of coding work from one AI agent to another", 2026-08-21. |
| 41 | §5 | npm / PyPI / crates.io for ripieno, cuelist, kapell | S | All nine 404 (free). |
| 42 | §5 | https://api.github.com/users/<name> | S | ripieno, cuelist, kapell users exist; `-dev` / `hq` variants 404. |
| 43 | §5 | https://rdap.org/domain/<d>, `whois` | S | ripieno.dev 2026-07-15, cuelist.dev 2026-04-23, kapell.dev 2026-05-25; ripieno.io and cuelist.io "Domain not found"; kapell.io 2022-08-11. |
| 44 | §4 | npm/crates metadata for divisi, attacca, tutti, yardmaster, paddock, anchorleg | P | Descriptions match R3; `tutti` crate is 2026-03-15, outside the stated range. |
| 45 | (added) | https://github.com/ijmh2/ripieno, https://ripieno.vercel.app | n/a | Reviewer-added: same-category product named Ripieno (finding 1). |
| 46 | (added) | https://tsdr.uspto.gov/statusview/sn90272389 | n/a | Reviewer-added: BATON Reg. 6418232, IC 042, ACTIVE (finding 19). |

Totals for R3's own citations (rows 1–35, 37–44; 43 rows): 35 S, 1 F (row 14), 1 S/F (row 11: facts supported, the inference drawn from it contradicted), 5 P, 1 N/A. No fabricated URL; no dead link apart from the disclosed 429.

## What is strong

- **The Facts base is unusually clean.** Nearly every number in §1–§3 (prices, rates, limits, dates) matched the live page to the digit. Failures were disclosed rather than papered over: the HashiCorp 429, the pricing redirect, the trademark gap marked `UNVERIFIED`.
- **The Conductor reversal is handled correctly and from primary sources.** R3 cites the changelog directly, never says "shut down", never pitches to "stranded" users, and downgrades Audience 2 confidence explicitly in the review log.
- **The B2B2B critique has a real principle.** "White-labelling is an upsell, not the entry product", and "the function must not be the embedder's differentiator", are testable conditions. The verdict ("later channel, Clerk-shaped") is right, even though the segment list is incomplete (finding 11).
- **The licence analysis is grounded in what actually happened.** HashiCorp/OpenTofu, Elastic and Redis are each backed by the originator's own words. The "what goes where" community/enterprise split is a concrete, reusable HLD constraint.
- **The experiments include genuine falsifiers:** "embed < 10%", "≥ 2 adapters < 20%", and "stayers cite 'it just works'". The written vendor-policy check is cheap and could change the design.
- **The FAQ answers are honest.** Q2 ("We cannot promise the company survives; we can promise the software does") and Q3 (do not build on subscription arbitrage) say what a skeptical CTO needs to hear.
- **The registry-level naming checks are correct and reproducible.** All nine candidate-registry results, the handles and the domain dates matched. Rejecting `baton` is right, and US trademark data now confirms it.
- **The rule 5 hygiene is good.** No personal names, the customer logo list is not reproduced, and the operator is not referenced.

## Re-verification of v3 (same reviewer)

> Date: 2026-11-23.
> - **Read:** v3 in full (286 lines, 44,648 bytes).
> - **Re-fetched:** 11 sources that are new or newly inlined in v3.
> - **Ran:** the `ritornello` checks the author could not: GitHub REST API, npm/PyPI/crates.io, RDAP/`whois`, and the USPTO tmsearch backend, with BATON as a control query.
> - R3 not edited; git not run.

### 1. Status of the 8 BLOCKING findings

All eight original BLOCKING findings are FIXED. ADVISORY findings 9–19, 21 and 22 are addressed. Finding 20 (length) is partly addressed (R-f). Line numbers (L) refer to v3.

| # | Finding | Status | Evidence in v3 | Residual |
|---|---|---|---|---|
| 1 | `ripieno` collides with a same-category product | FIXED | §5 L67 "**Rejected.**"; §E L135 "`ripieno` is dropped"; §5 method L64 adds the repository-name search | The new method was not fully applied to the replacement candidate. See N1. |
| 2 | Press-release para 2: "only its own agent"; "acquired" | FIXED | §F L148: "Most vendors shipped a conductor for their own agent; the one cross-vendor surface lives inside a single code host's cloud"; "joined a large platform company" | R-a |
| 3 | No "why not Coder"; Q1 dodged Agent HQ | FIXED | Q1 L158: "If you are GitHub-only and accept a GitHub-hosted control plane, use it". Q2 L159 is the Coder question and names the AI Premium overlap honestly. Old Q2 and Q4 are merged into Q3 L160, which adds DCO without a CLA. | — |
| 4 | Agent HQ facts stale | FIXED | §4 L58 "published 2025-10-28" plus the docs fact; §C L111 "timing: shipping, public preview"; Open questions L202 rewritten | — (docs quotes re-verified, row V1) |
| 5 | §D option 3 mislabelled BLOCKING | FIXED | §D L126: every con `[ADVISORY]`, "Survives"; verdict L130: "2 and 5 are eliminated by BLOCKING items …; 1, 3 and 4 survive" | — |
| 6 | §B invariant applied inconsistently | FIXED | L91 "Invariant applied uniformly"; L96 FSL/BSL `[BLOCKING]`; L97 ELv2 `[BLOCKING]`, SSPL `[ADVISORY]`; L98 Sustainable Use `[BLOCKING]`; verdict L100 | R-b ("open" is undefined) |
| 7 | "Empty server slot" contradicts R1 | FIXED | §C L114 names Orca, Paseo Hub, Emdash and Paradev and says they "partially occupy the self-hosted slot"; L118 quotes R1's narrower gap; §A L81 "not yet a proven payer"; §D L124 cons name the partial occupants | R-c (leftover "desktop tools") |
| 8 | No pre-build experiment falsifies the payer thesis | FIXED | §G L178 "Payer discovery calls (the kill test)": pass = ≥ 3 letters *and* ≥ 2 budget lines; fail ⇒ re-scope or stop. Recommendations L198. | — |

### 2. `ritornello`: checks run by the reviewer

**Summary:** the registries are free and USPTO shows no conflict, but a same-category product already uses the name (finding N1).

| Check | Result | Source |
|---|---|---|
| npm, PyPI, crates.io | All 404 (free). npm text search returns 0. Matches v3. | https://registry.npmjs.org/ritornello, https://pypi.org/pypi/ritornello/json, https://crates.io/api/v1/crates/ritornello, https://registry.npmjs.org/-/v1/search?text=ritornello |
| GitHub `ritornello` | Taken: a User account created 2017-08-01, 0 public repos. Not an organisation (`/orgs/ritornello` returns 404). | https://api.github.com/users/ritornello |
| GitHub `ritornello-dev` | **Taken**: an Organization created 2026-04-30, 0 public repos. v3 L70 says "not checked". | https://api.github.com/users/ritornello-dev |
| GitHub `ritornellohq`, `ritornello-hq`, `getritornello`, `ritornello-ai` | All 404 (free). | `https://api.github.com/users/<handle>` |
| GitHub repository-name search | 29 repositories, which matches v3's count. v3's "none in category" is **wrong**: one is a same-category product (next row). The rest are unrelated: a ChIP-seq peak caller (2016), a Rust internet-radio player, a Python map-generation library, a music-practice web app, a web store, a LilyPond snippet, and the `ritornello-labs` organisation (Anki/spaced-repetition tools). | https://api.github.com/search/repositories?q=ritornello+in:name |
| **Same-category product** | **"Ritornello", a macOS app; v0.1.0 released 2026-07-17 (DMG plus Sparkle appcast).** The Portuguese README says: "Canvas infinito de loop engineering — orquestre agentes de IA em terminais sobre um canvas: desenhe ciclos com avaliadores, condições de saída … e budgets … e mantenha o veredito final sempre humano". It ships a `rito` CLI through which "agentes conversam entre nós", and has "Pódio: custo por ciclo e veredito ship/block humano". In English: orchestrate AI agents in terminals on a canvas, with evaluators, exit conditions, budgets and a human ship/block verdict. That is agent orchestration with gates and budgets, this project's category. The repository description ("Releases oficiais do Ritornello — downloads e feed de atualizações") does not reveal this; only the README does. That is probably how the search missed it. | https://github.com/allansteve/ritornello-releases, https://github.com/allansteve/ritornello-releases/releases/tag/v0.1.0 |
| ritornello.dev | Registered 2026-07-23, which matches v3. The site reads "Ritornello … what returns, remains … Spaced-repetition decks and power-user tools: built around Anki", and the `ritornello-labs/www` repository is "ritornello.dev — organization homepage". Not same category, but it is an active software brand on the `.dev` domain. | https://rdap.org/domain/ritornello.dev, https://ritornello.dev, https://github.com/ritornello-labs/www |
| ritornello.io | `whois`: "Domain not found." Matches v3. | `whois ritornello.io` |
| ritornello.app, ritornello.ai | `.app` registered 2023-03-27. `.ai` returned RDAP 404 "Object not found"; not confirmed with `whois`. | https://rdap.org/domain/ritornello.app, https://rdap.org/domain/ritornello.ai |
| USPTO | Exact word mark RITORNELLO: **0 records**. Wildcard `*ritornell*` across word mark, pseudo-text, description and goods/services: 1 record, RITORNELLI, Reg. 3578625 (SN 77312220), IC 030, live, owned by a pasta maker; no software conflict. Control: BATON on the same endpoint returned 339 records, the same as in the first review, so the search works. US only. | `https://tmsearch.uspto.gov` (POST backend `/prod-stage-v1-0-0/tmsearch`); TSDR https://tsdr.uspto.gov/statusview/sn77312220 (not fetched) |
| EUIPO, UKIPO | `UNVERIFIED` (not checked). The word is Italian, so EU marks are plausible. | — |
| Web product-name check | `UNVERIFIED`: `web_search -s public` errored again this session. The README check above is the only product-name evidence. | — |

**Re-check of `kapell` (v3 L69).** The repository count of 89 matches. However, v3 names only one same-category `kapelle` (2026-08-19). The `kapelle` name search turns up two more agent-related repositories:
- https://github.com/0xkilgore/kapelle (2026-10-14): "Agent work under owner approval, with verifiable results".
- Its companion `kapelle-architecture`: "local-first agent operations".

Source: https://api.github.com/search/repositories?q=kapelle+in:name. See R-e.

### 3. New issues introduced by v3

**N1. [BLOCKING] §5 L70, §E L138, Recommendations L193: `ritornello` collides with a same-category product, yet v3 states "no same-category repository".**
- This is the same collision type as finding 1, introduced by the fix for finding 1.
- It feeds the HIGH naming recommendation ("take `cuelist`, `ritornello` or `kapell` to trademark search").
- The `-dev` handle is also taken, and the `.dev` domain hosts an active software brand.

Fix:
- Reject `ritornello` in §5 and §E with the same wording used for `ripieno`.
- Add one clause to the §5 method: "read the README of every repository-name hit whose description is empty, non-English or a release/landing repository".
- Recommendations: "take `cuelist` or `kapell`, or new candidates that pass the full §5 method, to trademark search".
- §E and Open questions L207: "the final two" become those two.

The following are ADVISORY and do not affect the verdict.

- **R-a. §F L148 and §A L82: two claims go beyond the sources.**
  - "Self-hosted alternatives exist, but their gates, audit and policy sit in paid tiers or do not exist." R1 table 3e marks Orca, Paseo Hub and Emdash pricing `UNVERIFIED`, so "paid tiers" is verified only for Coder and Paradev.
  - "As of this release remains proprietary with no self-hosted option" (L148) and "a permanent argument" (L82) both predict against Conductor's own roadmap: "Not yet, but we are building a self-hosted cloud offering … as part of an Enterprise plan" (verified today on https://www.conductor.build/docs/cloud/faq).
  - Fix: mark both as assumptions in a hypothetical 2027 release, and drop "permanent".
- **R-b. "Open" is used as an invariant without a definition.** §D L128 makes "violates 'open'" a BLOCKING con. §B, by contrast, labels "not OSI open source" ADVISORY (L96) and lets SSPL survive (L100), although Redis itself calls SSPL "not truly open source". VISION L19 says only "open". Define the term once (for example, "the core is under an OSI-approved licence") and apply it in both places. If the definition is OSI, SSPL becomes BLOCKING. The Apache-2.0 recommendation does not change.
- **R-c. §D: leftover wording from before the finding 7 fix.**
  - The verdict (L130) still says "what the desktop tools and Coder Community do not ship", but §C L114 now says these tools are "not only desktop apps". Say "the open-source conductors".
  - L124 says "Paseo Hub, Emdash and Paradev already sell" team features. Paseo Hub and Emdash pricing is `UNVERIFIED` in R1, so write "offer".
  - §C L114 lists Emdash as open source. R1 says only "Emdash Classic" is OSS, and it is in maintenance. Label it.
- **R-d. §B L95 (option 3): "incompatible with CNCF hosting, whose charter requires Apache-2.0 inbound and outbound".** Charter §11(c) lets the Governing Board approve an alternative licence "on an exception basis". The stronger constraint is §11(a)(ii): every project must "use only OSI-approved open source software licenses", which also excludes FSL, BSL, ELv2 and SSPL. Fix: write "requires Apache-2.0 unless the Governing Board grants an exception" (https://github.com/cncf/foundation/blob/main/charter.md).
- **R-e. §5 L69 and §E L139: `kapell` is weaker than stated.**
  - With `ritornello` out, `kapell` becomes rank 2, and it has three same-category near-homographs, not one. State that.
  - The `kapelle` fact has no URL (rule 1). Add one.
  - §E L137/L139 say "handles … free" for `cuelist` and `kapell`, but the bare GitHub user names are taken. Only the `-dev`/`hq` variants are free, as §5 says. Write "`-dev`/`hq` handles".
- **R-f. Length.** By the reviewer's count, v3 is about 5,125 prose words, excluding URLs, Sources and the Review log; about 5,388 including Sources. That is close to the 5,000 budget but still over. Dropping the `ritornello` bullet and rank (N1) and the §C "Cloud providers" bullet (L115) would close most of the gap.
- **R-g. Rule 3: confidence labels without the required one-clause reason.** §C L110 "(HIGH)", L115 "(MED)", L116 "(HIGH)"; §A L85 "(LOW.)"; and L82 "(MED on pull; HIGH on competitive pressure.)".
- **R-h. Status line L3 self-certifies "all 8 BLOCKING findings fixed".** Write "revised; pending re-verification", or update it after this review.
- **R-i. Rule 5 note for the N1 fix.** Cite the Ritornello app through its repository URL in Sources, as v3 already does for `ripieno`. Do not restate the account handle in prose, following v3's own practice of citing `x.com/i/status/…`.

### Spot-check of sources new or newly inlined in v3

S = supported; P = partly supported or overstated; F = failed.

| # | v3 L | URL | Result | Note |
|---|---|---|---|---|
| V1 | 58 | https://docs.github.com/en/copilot/concepts/agents/about-third-party-agents | S | "Third-party coding agents are currently in public preview"; Claude "can be powered by your existing Copilot subscription"; "your organization policies, or the "Agents" section of an enterprise's AI controls". |
| V2 | 36, 95 | https://github.com/cncf/foundation/blob/main/charter.md | P | §11(b)(ii)/(iii) quoted accurately; §11(c) exception clause omitted (R-d). |
| V3 | 25 | https://www.algolia.com/pricing | S | "10,000 requests/mo included then $0.60 per additional 1K requests". |
| V4 | 40 | https://cursor.com/pricing | S | "Teams $30+/user/month"; Enterprise "Custom". |
| V5 | 59 | https://openai.com/index/introducing-codex/ | S | "a cloud-based software engineering agent that can work on many tasks in parallel". |
| V6 | 17 | https://backstage.io/ | S | Publisher Spotify; "open source developer portal framework". |
| V7 | 18 | https://raw.githubusercontent.com/coder/coder/main/LICENSE | S | GNU Affero General Public License. |
| V8 | 85 | https://www.conductor.build/changelog/0.78.0-introducing-conductor-cloud | S | "mobile apps 👀, UI clients, and internal tools, like Slackbots". |
| V9 | 50 | https://www.conductor.build/docs/cloud/faq | S | GitLab/Bitbucket "Not yet"; cloud chat messages "stored on Conductor's servers"; self-hosted "Not yet, but we are building a self-hosted cloud offering". |
| V10 | 50 | https://www.conductor.build/docs/faq | S | "Agents run directly on your system without sandboxing." |
| V11 | 69 | https://api.github.com/search/repositories?q=kapell+in:name | S / P | Count of 89 matches; only one of three agent-related `kapelle` repositories is named (R-e). |
| V12 | 70 | https://api.github.com/search/repositories?q=ritornello+in:name | F | Count of 29 matches; "none in category" is false (N1). |

Totals: 9 S, 1 P, 1 S/P, 1 F. No fabricated URL.

### 4. Verdict (re-verification)

**REVISE.** The verdict is derived from BLOCKING items only:
- All 8 original BLOCKING findings are FIXED.
- 1 new BLOCKING finding (N1, `ritornello`) was introduced by the revision.

The N1 fix is confined to §5 L70, §E L136–L140, Recommendations L193 and one clause in the §5 method. Once those lines are fixed, the reviewer expects APPROVE. The lead can verify the fix on disk without another full review cycle.
