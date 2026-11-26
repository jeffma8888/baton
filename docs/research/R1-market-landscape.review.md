# Review: R1-market-landscape.md v1 (DRAFT, dated 2026-11-23)

> Reviewer: Fable · review R1 (independent; a different model from the author). Date: 2026-11-25.
> Scope: source integrity (49 of the 101 sources fetched, covering §1–§5 and the table; plus GitHub REST API for 16 repository rows and 1 reviewer-added source), independent re-verification of the Conductor shutdown reversal from primary sources, completeness against the R1 task, reasoning (rule 4), consistency with R3 v4 (APPROVED), rule 5 publishability.
> Method: `web_fetch` against the cited URLs. HN item pages were read raw and comment ids matched by regex. X posts were read via the api.fxtwitter.com mirror of the same post id (the id-only URL form R1 uses). Wayback snapshots as cited. GitHub REST API for stars / licence / created / archived. R1 was not edited; git was not run.

## Verdict

**REVISE.**

The Facts base is strong: of 49 sampled source rows, 42 fully support their claim, 5 partly do, 1 mismatches (Devin VPC, [S56]) and 1 could not be read (Ramp, [S99], which R1 already discloses). None is fabricated. The Conductor reversal, which VISION.md was rewritten on, re-verifies from primary sources exactly as R1 states it (Finding 4 is a paraphrase slip, not a factual one). The rule-4 structure is correct: the only elimination (O2a) rests on a BLOCKING con with HIGH confidence and primary sources, and the verdict sentence says so. Three findings block approval:

- (a) The GitHub Agent HQ row and prose are stale and contradict R3 v4 §C, which was approved with the newer facts (third-party Claude/Codex agents in public preview; execution on Actions runners including self-hosted; enterprise AI controls).
- (b) The Devin "Devbox in customer VPC" claim is not what the cited page says, and it is reused three times as evidence for a pattern.
- (c) The three audiences are numbered in the opposite order from VISION.md v0.1 and R3 v4, so "O1: Audience 1 first" reads as the opposite recommendation across the three documents.

All three have bounded fixes (one new source, a handful of cell and label edits). Nine ADVISORY findings follow, mostly date-stamping and resolving two UNVERIFIED labels whose answers are on the cited pages.

## Findings

### Source integrity and currency

**1. [BLOCKING] GitHub Agent HQ is described as "announced", with isolation and self-hosting UNVERIFIED or "No"; current GitHub docs say otherwise, and R3 v4 already uses the newer facts.**
§3a L102; table 3e L139; §4 L161; §5 L194; Recommendations L201, L225.
Evidence (fetched 2026-11-25):
- `https://docs.github.com/en/copilot/concepts/agents/coding-agent/about-third-party-agents` (not cited by R1): Claude and Codex are available as third-party agents in Copilot, "currently in public preview", enabled by organization policies or the enterprise "AI controls" settings.
- [S54] (cited): the coding agent runs in one of three environments: GitHub-hosted cloud sandbox, GitHub Actions (self-hosted runners are supported), or "Remote" (a customer-defined environment).
So the row "GitHub-hosted (detail UNVERIFIED) | Copilot + 5 vendors' agents | Org governance control plane | No | Proprietary | Copilot subscription | Announced 2025-10" understates maturity, and "Self-host: No" is wrong for execution. R3 v4 §C (approved) states the public-preview and enterprise-AI-controls facts; R1 is the upstream document and must not lag it.
Fix: add the third-party-agents docs page as a source; row 3e → isolation "Cloud sandbox / Actions runner (incl. self-hosted) / remote custom environment"; agents "Copilot + Claude, Codex (public preview); Google, Cognition, xAI announced"; self-host "Execution only (runner); control plane is GitHub"; maturity "Shipping; third-party agents in public preview (2026-11)". Reword L102 and L161 to "hosts Claude and Codex in public preview" and date-stamp. In L201 and §5 L194, Agent HQ becomes a stronger, not weaker, incumbent for the platform/DevEx audience; say so.

**2. [BLOCKING] Devin: "only the Devbox runs in the customer environment" / "Devbox in customer VPC" / Self-host "Partial" is not supported by [S56].**
§3a L103; table 3e L140; §5 L194; Recommendations L225; Handoffs L257.
[S56] (fetched) describes "Customer Dedicated Deployment": a single-tenant deployment that is hosted in Cognition's cloud and connected to the customer's network via AWS PrivateLink or an IPSec tunnel; the "brain" "always resides within Cognition's Cloud". Nothing on the page places the Devbox in the customer's VPC or environment. [S55] "Deploy in your VPC" is marketing shorthand for this dedicated option.
Consequence: the pattern sentence "vendor self-hosted modes keep the agent loop or 'brain' in the vendor cloud [S47][S56]" is still true, but Devin is not an example of *customer-side execution* and should leave the lists at L194, L225 ("vendor loops stay vendor-hosted" survives on [S47] alone) and L257 ("Devin Devbox" must go; Claude runners [S35] and Cursor workers [S47] stand).
Fix: L103 → "Enterprise offers a Cognition-hosted single-tenant deployment connected to the customer network via PrivateLink/IPSec; brain and Devbox both stay in Cognition's cloud [S56]." Row: isolation "Vendor VM; single-tenant dedicated deployment (vendor-hosted)"; self-host "No".

**3. [BLOCKING] Audience numbering is inverted relative to VISION.md v0.1 and R3 v4.**
§5 L182, L187, L192; Recommendations L214 ("O1: Audience 1 (B2B2B ...) first"), L217, L220, L223 ("O3: Audience 3 (platform/DevEx ...) first"); Summary L238–L240.
R1: 1 = B2B2B, 2 = teams, 3 = platform/DevEx. VISION.md v0.1 (rewritten on R1) and R3 v4 §A: 1 = platform/DevEx, 2 = teams, 3 = B2B2B. The substance agrees (platform/DevEx real MED; teams real-but-contested MED; B2B2B not evidenced LOW, matching R3 v4's A1/A2/A3 verdicts), only the labels are crossed, which is exactly the kind of drift the R3 review already caught once ("leftover Audience 1" in R3 v3). A reader of R3 who opens R1 and sees "O1: Audience 1 first" will read it as platform/DevEx first.
Fix: renumber §5 and O1/O3 to the VISION order (five label edits), or add one line under §5: "Numbering below is R1's; VISION v0.1 and R3 v4 use 1 = platform/DevEx, 2 = teams, 3 = B2B2B."

**4. [ADVISORY] §1 L52: the question that "nope, we're doing it here at Meta!!" answers is misparaphrased.**
[S26] is a reply to post id 2120974877700083950, which asks "Did acquisition fall through?", not "whether they left Meta". The reply still supports "still at Meta", so the conclusion stands. Also [S25] is quoted without its leading "Announcement:" and its last sentence "Regularly scheduled updates will resume next week!"; mark the trim with an ellipsis or quote in full (the dropped sentence actually strengthens L55).
Fix: "Asked whether the acquisition had fallen through: ..."

**5. [ADVISORY] §1 is undated and two days of releases already change three claims.**
L55; L26; L82; L190. [S3] RSS on 2026-11-25 lists 0.100.0 (2026-11-24) and 0.101.0 "Multi-repo workspaces" (2026-11-25) after 0.99.0.
- L55 "six further releases (0.95.0–0.99.0)" was correct on 2026-11-23; it needs "as of 2026-11-23".
- L82 lists "multi-repo work (50377216)" as requested/missing; it shipped in 0.101.0.
- 0.101.0 notes: "Conductor now stores settings in the app instead of `.conductor/settings.toml` files", so L26 (configuration model) and L190 (import `.conductor/settings.toml`) describe a superseded format; Paradev's migration mapping [S30] also targets the old files.
Fix: date-stamp §1 ("facts as of 2026-11-23") and add a short "post-draft changes" note with the two releases; VISION's onboarding promise ("import existing config") should not assume a file-based settings format.

**6. [ADVISORY] Table 3e L151 boxes.dev: "Pricing UNVERIFIED" and Self-host "No", but the cited page has both.**
[S80] (fetched) shows Trial $0 (20 box-hours), Starter $19/user/mo, Pro $99, Max $249, Teams custom with SSO/SCIM/RBAC/audit/spend controls and "BYO cloud / VPC"; "Sleeping devboxes are free". UNVERIFIED is the wrong label when the cited source answers the question; and the Teams tier makes Self-host "Partial", not "No".
Fix: fill the row; add SSO/SCIM/RBAC/audit to team features; delete boxes.dev from Open question L248.

**7. [ADVISORY] Table 3e L153 Coder: "Own agent + CLIs (list UNVERIFIED)" is resolvable from [S84], which also lists "Customizable branding".**
[S84] Community: "Run Claude Code, Codex, Cursor"; Premium AI: "3rd-party AI agent compatibility (e.g., Claude, Codex)", headless agents via API, audit prompts, RBAC, OIDC sync. Premium also lists "Customizable branding", which is the only incumbent evidence in R1's source set for the white-label/branding principle in VISION and for R3 v4's embedder sub-segment, and R1 does not mention it.
Fix: resolve the label to "Coder Agents + Claude Code, Codex, Cursor"; add "customizable branding" to the row or to §4.

**8. [ADVISORY] Two vendor-doc paraphrases are slightly stronger than their sources.**
- §3a L94 [S34]: "GitHub required" — the page lists GitHub and refers GitLab/Bitbucket to a "Platform restrictions" page (restricted, not categorically excluded). Fix: "GitHub (GitLab/Bitbucket restricted)".
- §3a L100 [S47]: "Partner hosts include E2B, Daytona, Modal and Vercel" is true; the page now lists ten (AWS Lambda, Cloudflare, Namespace, Modal, Daytona, E2B, Vercel, Tensorlake, Superserve, Render). "Include" covers it; add "as of" since the list is growing and is itself evidence for the embedder segment (Finding 9).

**9. [ADVISORY] B2B2B: R1 treats the segment as one; R3 v4 (approved) splits it into 3a and 3b, and the 3b evidence is already inside R1 but not connected.**
§4 L176–L178; §5 L182–L186 (R1's "Audience 1"). R3 v4 §A: 3a "the surface is the product" (not a launch audience); 3b "embedders for whom the conductor is not the differentiator" (sandbox vendors, forges, consultancies, model vendors; later channel, LOW). R1's "no primary evidence" verdict is quoted correctly by R3 v4 and VISION ("[R1: no primary evidence yet, LOW]"), so there is no contradiction; but R1's own sources already describe the 3b channel: ten sandbox vendors hosting Cursor workers [S47], Linear's agents "for distribution to other organizations" [S101], Coder's branding [S84]. Without a pointer the two documents look like they disagree on whether anything was found.
Fix: one sentence in §5 B2B2B: "R3 v4 splits this audience into 3a/3b; the embedder sub-segment (3b) is evidenced indirectly by [S47][S101][S84] and remains LOW."

**10. [ADVISORY] Rule 5 / handoffs: one stale item and one inference presented as fact.**
- L254: "It [BRIEF.md] also contains founders' real names" — the current BRIEF.md contains no personal names (fixed since R1 was drafted). Mark done or remove.
- L266: "Meta-owned Conductor" — ownership is an inference from "Conductor is joining Meta" [S22] and "we're doing it here at Meta" [S26]; no primary source states that an acquisition closed or on what terms, and the author's own [S26] question was "Did acquisition fall through?". Write "Conductor, now developed at Meta per [S22][S26] (deal terms UNVERIFIED)". The R3 review made the same point and R3 v4 adopted it.
R1 itself is clean on rule 5: X posts cited by id only; no HN usernames; the Series A customer list [S4] is not reproduced; no founder names.

**11. [ADVISORY] Early changelog claims cite the paginated index, which a reader cannot check in one hop.**
L14 (0.0.16, 2025-07-18), L25 (0.13.6 custom providers), L27 (0.22.4 GitHub Enterprise) cite [S3] = `/changelog` + RSS. The HTML index is 22 pages and the RSS carries only recent entries; the reviewer could not confirm these three from page 1 (not disproved either).
Fix: cite the per-release URLs (`/changelog/0.0.16-...`) as R1 already does for 0.93.0 and 0.94.2.

**12. [ADVISORY] Numbers drift; harmless if dated.**
Paseo plugins 624 → 652 [S67]; Orca 135k★ → 138.7k★; t3code 79.8k★; herdr 52.9k★; Paradev 16.7k★; Emdash 6.3k★; Vibe Kanban 28.8k★; Claude Squad 8.9k★; Coder 17.5k★ (GitHub API, 2026-11-25). All consistent with R1's figures within a week's growth. Add "stars as of 2026-11-23" once above the table.

### Completeness against the R1 task (no further findings)

Checked and present: Conductor features / pricing (archived page, exact tiers) / API (v0, beta, Pro) / cloud (specs, region, no self-host yet) / timeline incl. reversal (§1); migration destinations with HN comment ids and the Paradev migration page, plus what users miss and what they requested (§2); alternatives table with isolation / agents / team features / self-host / licence / pricing / maturity for every tool named in the task (Claude Code agent teams and self-hosted environments, Codex cloud, Cursor cloud agents and Self-Hosted Machines, Gemini worktrees and Jules, Agent HQ, Devin, Factory, Coder, Ona, T3 Code, Paseo, Orca, Herdr, Paradev, Vibe Kanban, Crystal "now Nimbalyst", Claude Squad, Terragon, Emdash, Superconductor, boxes.dev) (§3); who builds their own with primary engineering-blog sources for Stripe, Spotify, Ramp and Linear's agent platform (§3b); three audiences with for/against evidence (§5). Gaps are Findings 1, 6, 7 above. Not independently fetched by this reviewer: [S36][S37] Claude Desktop, [S42]–[S45] Codex app/enterprise, [S49] Gemini subagents, [S87]–[S97] building blocks, [S79] long tail; these are unverified here, not disputed.

### Reasoning (rule 4)

- Recommendations from BLOCKING alone: yes. L233 states it; the only BLOCKING con (O2a, refugee wedge gone) rests on [S27][S3] with HIGH confidence; all other cons are ADVISORY, including the O5 "do not build" con, which correctly treats absence of evidence as ADVISORY.
- Confidence labels: every HIGH/MED/LOW in §4, §5 and the Summary carries a one-clause reason (L201 "because these are primary sources"; L238–L240 each with "because ..."). Good.
- UNVERIFIED: used liberally and correctly in the table, except the two cells where the cited page has the answer (Findings 6, 7) and the Agent HQ cell where a public docs page does (Finding 1).
- Vendor claims as fact: "100k+ builders" and T3's "500,000" are labelled as vendor claims; stars are labelled "popularity, not usage". The one vendor claim misread is Devin's VPC (Finding 2).

## Source spot-check table

S = supported; P = partly / wording slightly stronger than source; F = mismatch; N = page not readable; A = reviewer-added source.

| # | Source | Claim in R1 | Result | Note |
|---|---|---|---|---|
| 1 | [S1] YC profile | "Run a team of coding agents on your Mac"; cloned dirs; free, uses Claude Code login; Linear | S | Founding year / headcount not in fetched text; not confirmed, not contradicted |
| 2 | [S2] Show HN launch | 228 points / 115 comments, 2025-07-17 | S | |
| 3 | [S3] changelog + RSS | releases 0.94.2–0.99.0, 2026-11-13..23 | S | Correct as of 2026-11-23; RSS now adds 0.100.0 (11-24), 0.101.0 multi-repo (11-25). Early entries (0.0.16, 0.13.6, 0.22.4) on later pages, not re-verified (Finding 11) |
| 4 | [S4] Series A post | 2026-03-30, $22M, Spark + Matrix, 10x since January, "phone to your VPC" | S | Page names 13 customer companies; R1 correctly does not reproduce the list |
| 5 | [S6] homepage | "Trusted by 100k+ builders" (vendor claim) | S | |
| 6 | [S8] FAQ | "Agents run directly on your system without sandboxing" | S | FAQ still says "Right now we don't [charge]" — stale vendor text, R1 does not rely on it |
| 7 | [S17] Cloud FAQ | 8 vCPU / 16 GB / 32 GB NVMe; Vercel sandbox us-east-1; GitLab/Bitbucket "Not yet"; transcripts on Conductor servers; self-hosted cloud "building ... Enterprise" | S | |
| 8 | [S19] pricing (Wayback 2026-09-29) | $0 / Pro $50 / Teams $60 per user invite-only / Enterprise DPA, SAML SSO, SCIM, SLA; SOC 2 Type II | S | Exact |
| 9 | [S20] API docs | v0, OpenAPI, bearer, beta, Pro required; routines/webhooks; `POST /v0/sql`; MCP server | S | |
| 10 | [S22] 0.93.0 | "Conductor is joining Meta"; Cloud read-only 11-13, off 11-20; local fixes to early 2027 | S | Page now carries appended "Update: Conductor isn't shutting down" |
| 11 | [S23] announcement (Wayback 2026-11-06) | Nov 4 no sign-ups; Nov 13 read-only + refunds; Nov 20 "We begin deleting Conductor Cloud user data" | S | |
| 12 | [S24] HN "joining Meta" (50369503) | 50370063, 50371439, 50370446, 50374681, 50374171, 50373638, 50395930 quoted | S | All seven verbatim; 222 points / 228 comments |
| 13 | [S25] X post 2120973052024598620 | "CONDUCTOR IS BACK ... exaggerated (by us)"; bio "Conducting @meta"; 2026-11-12 | S | Quote trims "Announcement:" and final sentence (Finding 4) |
| 14 | [S26] X post 2120975128158945374 | "nope, we're doing it here at Meta!!" | P | Reply verbatim; the question was "Did acquisition fall through?", not "whether they left Meta" (Finding 4) |
| 15 | [S27] 0.94.2 | "Conductor isn't shutting down. It's back, and we're building again." 2026-11-13 | S | RSS pubDate confirms |
| 16 | [S28] live announcement | timeline removed; "Plans have changed! We'll still be developing Conductor." | S | |
| 17 | [S29] HN "Conductor Is Back" (50473888) | 50477646 "already broke our trust once"; 50477299 "Too late"; 50477897 T3 Code gaps | S | 36 points / 21 comments |
| 18 | [S30] Paradev migrate | adopts worktrees in place; maps `conductor.json`/`.conductor/settings.toml` → `.paradev/config.json`; env vars; Cloud/iOS/API equivalents | S | Now targets a superseded Conductor format (Finding 5) |
| 19 | [S31] Show HN rebuild | 2026-11-07, "3 days, $170", MIT | S | |
| 20 | [S33] agent teams | experimental, `CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1` | S | |
| 21 | [S34] Claude Code on the web | vendor VMs, "GitHub required" | P | GitLab/Bitbucket referred to "Platform restrictions", not excluded (Finding 8) |
| 22 | [S35] self-hosted environments | runners on customer infra, Anthropic control plane, public beta Team/Enterprise, GitHub/GHES | S | |
| 23 | [S40] Codex cloud | own workspace per task; GitHub or GitLab; published environments; network/secrets | S | |
| 24 | [S46] Cursor cloud agents | isolated VMs; unlimited parallel; multi-repo; 4 code hosts; Slack/Linear/API/iOS | S | |
| 25 | [S47] Cursor self-hosted | Cursor runs loop/inference/planning, worker runs edits/commands; partners E2B, Daytona, Modal, Vercel | S | Partner list now ten hosts (Finding 8) |
| 26 | [S50] Gemini CLI worktrees | experimental | S | |
| 27 | [S52] Jules | experimental coding agent, GitHub | S | |
| 28 | [S53] Agent HQ blog | 2025-10-28; Anthropic/OpenAI/Google/Cognition/xAI; mission control; control plane | S | Blog says "over the coming months"; R1 stops here (Finding 1) |
| 29 | [S54] about-coding-agent | cited for governance control plane | P | Page also lists three execution environments incl. Actions self-hosted runners and "Remote"; R1 table says Self-host "No" (Finding 1) |
| 30 | [S55] Devin pricing | Teams $80/mo ≤100 members, multiplayer, SSO; Enterprise SCIM, RBAC, VPC | S | "Deploy in your VPC" is the marketing line for the dedicated deployment |
| 31 | [S56] Devin VPC overview | "only the Devbox runs in the customer environment" | **F** | Page: Cognition-hosted single-tenant deployment via PrivateLink/IPSec; brain "always resides within Cognition's Cloud"; Devbox not placed in customer environment (Finding 2) |
| 32 | [S57] Factory pricing | BYO inference; Team up to 10 seats; Enterprise on-prem options, audit logging, sub-orgs | S | |
| 33 | [S59][S60][S61][S63][S66][S68][S70][S72][S74][S76][S77][S78] + coder, conductor-oss, daytona, agentapi | stars, licences, created dates, "Crystal is now Nimbalyst", Terragon created 2026-01-16, daytona/agentapi archived | S | GitHub REST API 2026-11-25; all within a week's drift (Finding 12); paseo licence NOASSERTION via API as R1 notes |
| 34 | [S62] t3.codes | "open-source control plane for coding agents"; "Fork the whole thing"; "500,000 devs" (vendor claim); agents list; MIT | S | |
| 35 | [S64] onorca.dev | Slack/Linear/GitHub bots; mobile; open source; "your machines" | S | |
| 36 | [S65] Orca plugins?kind=machines | machine plugins | S | 512 plugins listed |
| 37 | [S67] paseo.sh | desktop/mobile/remote daemon; Paseo Hub "Teams and triggers"; MCP/CLI/TS SDK; 624 plugins; "+35 more" providers | S | 652 plugins on 2026-11-25; maintained by one person, sponsorship + Hub funded |
| 38 | [S69] herdr.dev | SSH/Tailscale; "No account"; Apache 2.0; free | S | 576 mods |
| 39 | [S71] Paradev pricing | $0 / $20 / $60 / $200 / Enterprise (SAML SSO, SCIM, audit, SOC 2); Team 150 h/seat | S | |
| 40 | [S73] emdash.sh | own infra or Emdash cloud; ACP; SSO/roles/spend limits | S | Emdash Classic in maintenance, new work on 2.0 |
| 41 | [S75] Vibe Kanban shutdown (Wayback) | "couldn't find a business model" and shutdown quotes | S | Exact |
| 42 | [S76] terragon-oss | snapshot 2026-01-16; Claude Code/Codex/Amp/Gemini; container isolation; Apache-2.0 | S | 262★ |
| 43 | [S80] boxes.dev | "Full computers, not worktrees"; Slack/Linear/schedule/API; "Sleeping devboxes are free"; pricing UNVERIFIED; self-host No | P | Page has $0 / $19 / $99 / $249 / Teams custom with SSO/SCIM/RBAC/audit/spend controls and "BYO cloud / VPC" (Finding 6) |
| 44 | [S81] Superconductor pricing | 100 credits/hr 1 vCPU/2 GB; $25 per 10,000 overage; 4/40/unlimited members; "never charged per seat"; BYO plans/keys | S | |
| 45 | [S84] Coder pricing | Community free; Premium per-user, audit prompts, RBAC, OIDC; AI Premium Coder Agents, headless API; agent list UNVERIFIED | P | Page names Claude Code/Codex/Cursor and "3rd-party AI agent compatibility (e.g., Claude, Codex)"; also "Customizable branding" (Finding 7) |
| 46 | [S86] ona.com | "The Enterprise software factory"; Ona/Codex/other agents governed; Stripe, Uber, Harvey, Cloudflare, Genentech talks | S | |
| 47 | [S98] Stripe Minions | 1,000+ PRs/week; Slack → PR; reason Ruby/Sorbet + homegrown libraries | S | Page also says devboxes chosen because worktrees "wouldn't scale at Stripe" — useful for §3b |
| 48 | [S99] Ramp Inspect | built own background agent; early access to outside teams | N | Body is JS-rendered; meta description matches R1's disclosure; R1 already flags non-retrieval |
| 49 | [S100] Spotify Honk | 1,500+ merged PRs; Fleet Management; internal CLI pluggable across agents | S | |
| 50 | [S101] Linear agents | agents behave like users; "for internal use ... or for distribution to other organizations" | S | |
| 51 | docs.github.com/.../about-third-party-agents | (not cited) Claude and Codex third-party agents in public preview; org policies / enterprise AI controls | A | Basis of Finding 1 |

Totals: 50 cited rows — 42 S, 5 P, 1 F, 1 N (row 33 groups 16 repository checks and counts once); 1 reviewer-added.

## What is strong

- The reversal is handled entirely from primary sources with exact quotes and dates ([S22][S23][S25][S27][S28]), and the correction to BRIEF/VISION is stated explicitly (L201, L253). Re-verified here line by line; it holds.
- Vendor claims are labelled as claims ("100k+ builders", T3's "500,000"), stars are labelled "popularity, not usage", and HN evidence is quoted verbatim with comment ids, which made every one of the eleven quotes checkable in one hop.
- Rule 4 is applied correctly: one BLOCKING con, one elimination, verdict sentence says so, the "do not build" con is ADVISORY, and every confidence label has a reason clause.
- UNVERIFIED is used generously rather than papering over gaps (pricing for Orca, Paseo Hub, Emdash, Ona; Coder agent list), and the two cases where it was over-used (Findings 6, 7) are the cheap kind of error.
- Table 3e covers every tool the task named plus the long tail, with licence and isolation model for each; the OSS licence nuances (Paseo NOASSERTION vs Apache-2.0 file; Claude Squad and Coder AGPL) are noted.
- The in-house builders section uses engineering-blog primary sources and extracts the *reason* each company built (codebase-specific tooling), which is the evidence R3 v4 leans on for audience 1.
- The gap statement is narrow and specific ("narrower than nothing exists"), and R3 v4 quotes it verbatim, so the two documents agree on the central claim.
- Rule 5 is clean in the body: id-only X URLs, no usernames, no founder names, no reproduced customer list.

## Learnings / mistakes

- I first flagged L55 "six further releases (0.95.0–0.99.0)" as an error because the RSS showed 0.100.0 and 0.101.0. It was correct for R1's date; the defect is the missing "as of", not the count. Lesson: check the document date before calling a fact wrong, and report currency separately from accuracy.
- My first HN fetches used text extraction, which truncated long comments and hid comment ids; re-fetching raw and matching `commtext` by id fixed it. Verbatim-quote checks need the raw page.
- The Spotify page was slow and my fetch got promoted to a background task; piping its output delayed reading it. Fetch slow pages to a file first.
- The Conductor changelog HTML is paginated (22 pages); fetching page 1 cannot disprove early-release claims, so I recorded them as not re-verified rather than as failures.
- I initially read [S55] "Deploy in your VPC" as confirming R1's Devin claim; only [S56] (the page R1 actually cites for it) shows the deployment is vendor-hosted. Always open the specific page cited for a technical claim, not the marketing page beside it.
