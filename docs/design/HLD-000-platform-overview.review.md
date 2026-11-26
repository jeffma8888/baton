# HLD-000 platform overview - cross-model review

> status: COMPLETE (REVISE)

## Verdict

REVISE: unsupported BLOCKING eliminations, a PRD host-priority conflict, an omitted FR-25 warning and audit/timeout contradictions; seven required experiments remain UNRUN.

## Findings

1. [BLOCKING] §3.3 Option 4, lines 462-466: a runner that reads/writes the database directly need not be authorised to write *gate or approval* records; a restricted Postgres role can write only runner-owned records. I4 prohibits approving/merging but direct DB access does not itself grant those privileges (PRD FR-17, FR-26; R2 §Analysis/Self-host describes scoped roles). The claimed BLOCKING con does not follow from this option. Compare a scoped, table-restricted role as a viable but less preferred variant and demote the network/operations burden to ADVISORY; only an explicitly unrestricted gate-writing DB credential fails I4.

2. [BLOCKING] §3.2 lines 394-405: a default container tier and an explicit `trusted-local` opt-in are declared, but the eligibility determination depends on an UNRUN pilot threat review; *a pilot workload is not yet demonstrated eligible*. PRD §Handoffs item 3 explicitly says to record the pilot tier or mark it unresolved. The text marks it UNRESOLVED, so this is currently an approval hold rather than a contradiction; close only after R3 §G and pilot threat evidence (Action Item 1).

3. [BLOCKING] §3.2 Option 3, lines 413-417: the blanket BLOCKING con says a hosted-provider default necessarily needs a *paid* sandbox and violates I11/NFR-7. R3 §3 (line 43) documents E2B Hobby free with introductory credit, and R2 §1 lines 15-23 documents Hobby and provider contracts; PRD NFR-7 requires a no-paid-account *workflow*, not that the default itself use no free hosted service. Since the P0 local container backend still exists, this option may remain viable on advisory cost, availability and code-egress grounds. Distinguish a *mandatory paid-only* hosted dependency (BLOCKING) from a configurable hosted default with a no-paid local path (ADVISORY), and revise the comparison/verdict.

4. [BLOCKING] §5 Options 3 and 5, lines 602-618: R2 §3 lines 54-58 says production Temporal needs a Service, persistence and workers, and Trigger.dev has a Compose self-host with Redis and webapp/worker services. Neither source says a separate *cluster outside the product Compose* is necessary. PRD FR-31/NFR-9 demand a single Compose install without a separate workflow-engine cluster, not only three services. The asserted I7 BLOCKING for both alternatives is unsupported by these citations; compare a single-Compose topology with the extra service(s), tag operational burden ADVISORY, and reserve BLOCKING for variants demonstrably requiring a separately installed cluster.

5. [BLOCKING] §7.1 Option 1, lines 704-708 and comparison line 726: calling one-host P0 the "PRD default" contradicts approved PRD §P0 option set 3 (lines 92-95), where *GitHub plus GitLab* is provisionally preferred and a GitHub-only alpha becomes preferred only if pilot evidence selects it; FR-20 (line 124) becomes P0 if option A wins before HLD approval. Assumption 4 and Action Item 5 admit the survey is UNRUN. Keep this as a provisional conditional fallback, do not reassign FR-20 to P1 as a settled choice, and make approval depend on the survey or retain PRD option A as the P0 baseline.

6. [BLOCKING] §4 Option 4, lines 518-522: R2 §2 line 49 says A2A does not itself *grant* filesystem/process visibility; it does not establish that baton *cannot* start an A2A-speaking agent inside its own executor or supply the checkout outside the A2A protocol. The cited source supports the advisory limitation of A2A alone, not a categorical impossibility under I3/FR-10. Compare a baton-placed A2A agent as a viable but awkward variant or explicitly restrict the eliminated variant to external opaque agents with no baton runtime/checkout contract.

7. [BLOCKING] §6.2 comparison line 696 and Option 4 line 669: the table eliminates a "branch-read variant (I4)" without presenting it as an option with its own Pros, tagged Cons and blocking-only Why not, despite announcing it as an eliminated variant. Either enumerate the candidate-branch policy option and analyze it fully (including a sourced I4 BLOCKING) or remove it from the table of eliminated options and label the prose as a scoped caveat.

8. [BLOCKING] §10.1 Option 3, lines 789-810: OS packages, if an *alternative to* the required Compose install, fail exactly the FR-31/I7 requirement used to eliminate Kubernetes-*instead-of*-Compose Option 2, yet receive only an ADVISORY con. Define Option 3 explicitly as optional packages *alongside* the required Compose install (viable), or tag a packages-only install BLOCKING and update the table, consistently with Option 2.

9. [BLOCKING] §6.4 lines 681-689 and §Requirements item 6: PRD FR-25 (line 129) requires a priced run to *warn at its configured monetary threshold*. The HLD describes reservations, debits, caps and unknowns, but no threshold warning or where it appears in CLI/API, so one active P0 requirement is omitted. Add the server warning on crossing a configured threshold, auditable/visible in the team API and CLI, separate from optional hard stop.

10. [BLOCKING] §5 state diagram lines 556-579 versus §1 step 7 (line 212) and §11 failure table (line 836): the diagram maps a time-limit expiry to `Cancelled`, while both prose locations say it ends `timed out`; the diagram also has no normal agent-failure terminal edge. A single run cannot both be cancelled and timed out for the same event. Add distinct TimedOut and Failed transitions (including pre-run failure where appropriate), retaining FR-13 explicitly unresolved on uncertain cancellation, then align the workflow and failure table.

11. [BLOCKING] §5 retention line 583 versus §10.2 Option 1 line 799: basic audit events are promised never deleted in P0, while restoring a pre-upgrade database backup as rollback explicitly *loses writes made after upgrade*, including committed audit rows. That breaks the reconstructable committed audit guarantee in PRD NFR-10 (line 153) and creates a second internal contradiction. Require a write freeze from backup through cutover/rollback or an independently recoverable audit journal with verified replay; document the retention/rollback trade-off consistently.

12. [BLOCKING] §2 Options 1-6, lines 221-349: R2 §Options C (lines 132-137, 153-162) keeps a full provider-neutral control plane with independently pluggable sandbox and agent providers viable, separate from bounded B and E, but §2 compares only B-inside-C, two E variants, AGPL-embedded E, A and D. The full C alternative has no option, Pros, tagged Cons or Why not; “outside the 12-week slice” is ADVISORY in R2, not a BLOCKING elimination. Add a named full-C option with R2-sourced trade-offs and an advisory-only sequencing verdict, or explicitly demonstrate it is identical to a listed option without silently eliminating C.

13. [ADVISORY] §9.1 Option 4 lines 764-766: “fork to customize” does not *logically prevent* a fork from exposing versioned extension boundaries; the BLOCKING con follows only if this option means **fork instead of any versioned boundary**, not merely forking code. State the exclusive no-contract scope explicitly and preserve the same I8 tag only for that variant; otherwise mark upgrade drift ADVISORY.

14. [ADVISORY] §6.1 Option 4 line 656: “host CI cannot run before PR” overstates Git-host capabilities; PR-triggered checks cannot, but branch-push checks can. Limit the BLOCKING to the required PR-triggered checks in PRD FR-16, so the valid sequencing critique remains intact.

15. [ADVISORY] §9.1/§3 provider contracts (lines 359, 444, 760) versus PRD §Handoffs item 2 and R2 §Handoffs line 200: versioning, executor tiers and agent capabilities are present, but provider-neutral executor lifecycle, artifact import/export, egress/secrets capabilities and failure ownership are spread or underspecified. R2 explicitly names Coder and GitHub Codespaces as executor providers: Coder is in §2 Option 3, Codespaces is unmentioned and not clearly included in §3.2 hosted microVM list (Codespaces runs Docker on a hosted VM, R2 §4 line 71). Add a bounded provider-capability table and identify these variants; reserve protocol payloads for the LLD.

16. [ADVISORY] §2 Option 1 pros line 266 claims “all code is Apache-2.0 and baton’s”; the diagram itself integrates model providers and a Git host, and dependencies remain to be reviewed (Action Item 11). Say “baton-owned first-party core code” rather than “all code”, pending dependency licence review.

17. [ADVISORY] Action Items 2, 7, 10 and 11 (lines 883, 888, 891-892): item 2 assigns the pre-approval B-versus-E architecture spike to a runner LLD author even though component LLD follows approved HLD (PHASE-1 lines 14-16, 39-42); items 7, 10 and 11 block an acceptance criterion, table cell or release rather than naming the actual design decision. Name the HLD author/security reviewer as pre-approval spike owner and name the blocked decision for each (retention default, Trigger.dev licensing/viability, core dependency admission). All items otherwise have owner roles and clear blockers.

18. [ADVISORY] §6.3 line 673 enumerates audit event *record fields*, and §5 lines 553-579 lists persistence entities and detailed state-machine labels; no class names, config keys or test file names were found, but enumerating event fields approaches LLD schema definition (TEMPLATE-HLD lines 3-5). Keep required evidence categories and high-level states in HLD; leave record-field inventory and specific schema to the LLD.

19. [ADVISORY] Body byte budget: 69,746 bytes before Review log (70,000 cap; 254 bytes remain), measured on disk. Requested substantive fixes cannot fit in 254 bytes without equivalent trims; **raise the cap to 72,000 bytes for v2** rather than stripping design evidence, and record this explicit cap adjustment.

20. [BLOCKING] §9.1 Option 3, lines 758-764 versus I8 line 164 and PRD FR-29 (line 133): “sandboxed in-server plugins (Wasm-class)” *run untrusted plugin code in the server*, directly contrary to the approved PRD’s explicit no-untrusted-server-plugins condition, yet the option has only ADVISORY cons and remains viable. This is the same I8 invariant tagged BLOCKING for in-process dynamic plugins (Option 2) and ADVISORY for in-process Wasm without an approved exception. Either move sandboxed Wasm to a runner/separate process (renaming the option) or mark the in-server variant BLOCKING under I8; update §9.1 comparison and Appendix A.

## Citation spot-check table

| # | HLD claim / line | Opened source | Grade | Note |
|---|---|---|---|---|
| 1 | I1 §Design, l157 | VISION §Strategy ll38-46; PRD §P0 ll53-69 | supports | Customer-hosted team server first. |
| 2 | I2 §Design, l158 | R3 §B ll89-102; R2 §Analysis/Durability ll98-102 | supports | Apache core, no BSL/SSPL bundled server, no AGPL source embedding under approved boundary. |
| 3 | I3 §Design, l159 | VISION §Design principles ll54-60; PRD FR-3/4/11 ll105-115 | supports | Two adapters and two execution backends in core. |
| 4 | I4 §Design, l160 | PRD FR-17/26 ll121,130 | supports | Execution identity cannot approve or merge; does not prove unrestricted DB privilege. |
| 5 | §3.3 Option 4, l465 | PRD FR-17/26 ll121,130; R2 §Analysis/Self-host l106 | partly | Source prohibits approval by execution identity, but does not establish every direct DB connection has approval-write grants. |
| 6 | I5 §Design, l161 | PRD FR-16 l120 | supports | Fail-closed same-SHA checks. |
| 7 | I6 §Design, l162 | PRD FR-18 l122; Out of scope 9 l165 | supports | PR after human gate, human merges under host protection. |
| 8 | I7 §Design, l163 | PRD FR-31 l135; NFR-9 l152 | supports | Single Compose deployment, no required external workflow cluster. |
| 9 | I8 §Design, l164 | PRD FR-29 l133 | supports | Versioned extensions, no untrusted server plugin. |
| 10 | I9 §Design, l165 | PRD FR-22/23 ll126-127; NFR-10 l153 | supports | Privilege transitions require audit commit. |
| 11 | I10 §Design, l166 | PRD FR-24 l128 | supports | Run and repo/org time/concurrency ceilings. |
| 12 | I11 §Design, l167 | PRD NFR-7 l150 | supports | Clean core workflow without paid sandbox. |
| 13 | I12 §Design, l168 | PRD FR-11 l115; NFR-4 l147; VISION §Design principles l55 | supports | No host checkout or socket mount. |
| 14 | §2 Option 4, l312 | R2 §Options E ll146-150; R3 §B ll89-102 | supports | AGPL embedded source is incompatible with approved Apache-only core. |
| 15 | §2 Option 5, l323 | R2 §Options A ll118-123; VISION §Strategy ll40-46 | supports | Local-only lacks shared team plane. |
| 16 | §2 Option 6, l334 | R2 §Options D ll139-144; VISION §Strategy l40 | supports | Vendor-hosted first fails customer-hosted server. |
| 17 | §3.1 Option 2, l373 | PRD FR-11 l115; FR-4 l108 | supports | Worktree-only omits required container. |
| 18 | §3.1 Option 3, l379 | PRD FR-4 l108; FR-10 l114 | supports | Both worktree and container acceptance required. |
| 19 | §3.1 Option 4, l385 | R2 §1 l10; PRD FR-10 l114 | supports | Clone not security boundary. |
| 20 | §3.2 Option 3, l416 | PRD NFR-7 l150; R2 §1 ll15-23; R3 §3 l43 | contradicts | Free Hobby sandbox exists; no-paid-account path remains via local executor. |
| 21 | §3.2 default, ll390-405 | R2 §1 ll11-23,29-36 | supports | Firecracker KVM and hosted pricing/persistence/network/secrets trade-off; container shared-kernel caveat. |
| 22 | §3.3 Option 1, ll442-448 | R2 §Options B ll125-130; §Analysis/Durability l100 | supports | Outbound claims and fencing recovery. |
| 23 | §4 Option 3, l515 | R2 §2 l49; PRD FR-13 l117 | supports | MCP tools are not session/cancel protocol. |
| 24 | §4 Option 4, l521 | R2 §2 l49; PRD FR-10 l114 | partly | A2A alone offers no control; supervised local deployment not ruled out. |
| 25 | §4 adapter capability table, ll480-490 | R2 §2 ll40-49 | supports | ACP adapters/native modes, CLI and SDK distinctions. |
| 26 | §5 Option 3, l605 | R2 §3 l54; PRD FR-31 l135 | partly | Extra Temporal Service is supported; separately installed *cluster* is not established. |
| 27 | §5 Option 4, l611 | R2 §3 ll55-56; R3 §B ll89-102 | supports | Restate BSL and Inngest SSPL bundled servers violate core boundary. |
| 28 | §5 Option 5, l617 | R2 §3 l57; PRD FR-31 l135 | contradicts | Source says Compose self-host exists; extra services not necessarily separate cluster. |
| 29 | §5 Option 1, ll551-594 | R2 §Analysis/Durability ll98-102; S46 index l250 | supports | Postgres SKIP LOCKED, at-least-once effects, fenced leases. |
| 30 | §6.1 Option 2, l652 | PRD FR-18 l122 | supports | Draft PR precedes required human gate. |
| 31 | §6.1 Option 3, l654 | PRD Out of scope 9 l165 | supports | Agent merge bypasses host/human authority. |
| 32 | §6.1 Option 4, l656 | PRD FR-16 l120; R2 S73 index l277 | partly | PR-triggered CI cannot run before PR; generic host CI can run on branch push. |
| 33 | §6.2 branch variant, ll669,696 | PRD FR-17 l121; R2 §Analysis/Execution l90 | supports | Self-authorising change gate conflicts with separated approval; presentation lacks option discipline. |
| 34 | §6.3 Option 2, l677 | PRD FR-22/23 ll126-127; NFR-10 l153 | supports | Audit sink-only asynchronous commit before persistence is prohibited. |
| 35 | §6.3 Option 3, l679 | R2 §Analysis/Execution l94; PRD FR-23 l127 | supports | Mutable activity feed not audit. |
| 36 | §6.4 Option 2, l687 | PRD FR-24 l128; R2 §Analysis/Execution l92 | supports | Partitioned runner needs local time enforcement. |
| 37 | §6.4 Option 3, l689 | PRD FR-24 l128; R2 §Analysis/Execution l92 | supports | Runners alone cannot coordinate org-wide cap. |
| 38 | §7.1 Option 4, l712 | PRD FR-18 l122 | supports | Plain push lacks required PR creation/status. |
| 39 | §7.1 baseline, ll706-710 | PRD §P0 option set 3 ll92-95; FR-20 l124 | contradicts | A two-host alpha is provisional preference until survey; GitHub-only conditional. |
| 40 | §9.1 Option 2, l762 | PRD FR-29 l133; R2 §Analysis/Self-host l108 | supports | Native untrusted plugin obtains host privileges. |
| 41 | §9.1 Option 4, l766 | PRD FR-29 l133; R2 §4 l66 | partly | Fork-only is against versioned extension goal, but a fork can still expose versioned contracts. |
| 42 | §9.2 Option 2, l772 | PRD FR-3 l107; NFR-9 l152; R3 §B l102 | supports | Only combined enterprise-containing artifact fails explicit clean-core test; approved split calls for core without enterprise code. |
| 43 | §10.1 Option 2, l793 | PRD FR-31 l135 | supports | Chart *instead of* Compose contradicts FR-31. |
| 44 | §10.1 Option 3, l795 | PRD FR-31 l135 | partly | Packages alone fail same FR-31; packages in addition remain viable. |
| 45 | §3.2 provider default, l394 | PRD §Handoffs item 3 l202; VISION §Open questions item 4 l77 | supports | Default answers OQ4/OQ5; actual pilot tier unresolved pending evidence. |
| 46 | §5 audit retention, l583 | PRD NFR-10 l153; §Handoffs item 6 l205; R2 §Analysis/Execution l94 | supports | HLD decides indefinite basic-audit retention; payload access/retention separated. |
| 47 | §9 core/Enterprise, l756 | R3 §B ll89-104 | supports | Basic OIDC/audit/gates open; SSO/SCIM/export/HA/branding paid. |
| 48 | §2 OSS reuse, ll277-304 | R2 §Options E ll146-162; §5 l78 | supports | Paseo and customer-installed Coder API viable, embedded-AGPL variant excluded. |
| 49 | §9.1 Option 3, l764 | PRD FR-29 l133; HLD I8 l164 | contradicts | In-server Wasm still runs untrusted plugin inside server; PRD expressly prohibits this P0. |
| 50 | §2 missing full C, ll221-349 | R2 §Options C ll132-137, verdict l162 | partly | Full-C platform remains viable but is not independently compared. |
| 51 | §1 market gap, ll13-19 | R1 §1 ll38-59; §Analysis ll215-225; R2 §5 ll75-80 | supports | Active incumbent, GitHub Agent HQ, existing OSS servers, narrow open governance gap. |
| 52 | §8 OIDC and single-org, ll729-743 | PRD FR-26/34 ll130,138; NFR-6 l149; R3 §B l102 | supports | Core OIDC and single-org roles; Enterprise SSO/SCIM reserved. |
| 53 | §11 opt-in heartbeat, l839 | PRD FR-32 l136; R3 §G ll163-173 | supports | Telemetry opt-in, report opt-in denominator. |
| 54 | §11 vendor egress, l822 | R2 §2 ll40-50; R3 §F Q5 l161 | supports | Proprietary vendor inference still crosses network. |
| 55 | §10.2 migrations, l799 | PRD NFR-8 l151; R2 §Analysis/Self-host l108 | supports | Backup, dry run and restore boundary, though audit rollback needs reconciliation (finding 11). |
| 56 | §11 cost provider idle, l841 | R2 §1 ll15-23; §Analysis/Execution l92 | supports | Idle memory/storage cost can exceed model token budgets. |
| 57 | §9 DCO and Enterprise packaging, l756 | R3 §B l102; §Handoffs l212 | supports | DCO without CLA and paid-only enterprise capabilities. |
| 58 | §3.3 scoped DB alternative, l465 | R2 S46 index l250; PostgreSQL privileges page verified directly this run | partly | R2 S46 covers queue/locking, not privilege grants; primary Postgres documentation confirms table-scoped grants. |

## Option-analysis audit table

| Sub-decision | Options | Pros/cons and why | Verdict / source | Assessment |
|---|---:|---|---|---|
| §2 Control plane | 6 listed, C missing | No: full C omitted | R2 §Options B/C/E survive; A/D and embedded E appropriately eliminated | finding 12: BLOCKING |
| §3.1 Tier model | 4 | Yes | I3/FR-10 sourced | clean |
| §3.2 Default tier | 5 | Yes, but hosted free path mis-tagged | I11 cannot eliminate free hosted defaults; pilot eligibility UNRUN | findings 2, 3: BLOCKING |
| §3.3 Runner contract | 4 | Yes | Direct DB falsely assumed unrestricted | finding 1: BLOCKING |
| §4 Agent contract | 6 | Yes | MCP unsupported; A2A supervisor inference overclaims | finding 6: BLOCKING |
| §5 Durable state | 7 | Yes | Temporal and Trigger cluster inference unsupported; timeout/audit rollback inconsistent | findings 4, 10, 11: BLOCKING |
| §6.1 Gate placement | 4 | Yes | Pre-PR + PR-triggered separation holds; host CI wording overbroad | clean (advisory wording) |
| §6.2 Policy representation | 4 named + 1 eliminated variant | Variant missing full analysis | Own-branch policy fails I4 if self-authorised | finding 7: BLOCKING |
| §6.3 Audit write | 3 | Yes | I9 and FR-23 supported | clean |
| §6.4 Budget point | 3 | Yes | I10 supported; FR-25 warning omitted | finding 9: BLOCKING |
| §7.1 Host breadth | 4 | Yes | FR-18 elimination sound; preferred P0 conflicts PRD | finding 5: BLOCKING |
| §7.2 Candidate writer | 3 | Yes | No BLOCKING eliminations; local shared-user risk advisory | clean |
| §8 Tenancy | 4 | Yes | P0 single org; no BLOCKING eliminations | clean |
| §9.1 Extensions | 4 | Yes | Option 3 in-server Wasm contradicts I8/FR-29; fork scope ambiguous | finding 20: BLOCKING |
| §9.2 Packaging | 4 | Yes | Combined-only binary fails core-only acceptance | clean |
| §10.1 Install | 3 | Yes | Option 3 packages-only must carry I7 too | finding 8: BLOCKING |
| §10.2 Upgrade | 3 | Yes | Advisory only; all viable | clean |

## Requirement coverage table

One row per active requirement (33 FR + 10 NFR). “Satisfied” means a coherent HLD design or explicit test boundary, **not** a claim that the alpha test has run. FR-33 is withdrawn and not counted; conditional FR-20 is provisionally deferred but finding 5 blocks declaring that final.

| Requirement (PRD priority) | Coverage | HLD evidence or missing item |
|---|---|---|
| FR-1 (P0) | satisfied | §1 steps 1-2; §2 thin CLI and API. |
| FR-2 (P0) | satisfied | §1 steps 1, 5; §5 ordered run events. |
| FR-3 (P0) | satisfied | §1 full workflow; §9 open-core packaging. |
| FR-4 (P0) | satisfied | §3.1 two executors; §4 two adapters and provisional pair. |
| FR-5 (P0) | satisfied | §4 capability registration and runtime deny. |
| FR-6 (P0) | satisfied | §4 permitted keys/enterprise billing; no consumer dependency; vendor check UNRUN. |
| FR-7 (P1) | deferred | Out of Scope 6: ACP ingestion via §4 transport seam, adapter HLD. |
| FR-8 (P1) | deferred | Out of Scope 7: MCP-scoped tools, adapter/policy HLD. |
| FR-9 (P0) | satisfied | §3.1 separate checkout/branch; §5 fenced lease. |
| FR-10 (P0) | satisfied | §3.1 `trusted-local` label and tier selection. |
| FR-11 (P0) | satisfied | §3.1-3.2 hardened self-hosted container; §11 mounts excluded. |
| FR-12 (P0) | satisfied | §3.3 outbound leased runner; disconnect leaves queued. |
| FR-13 (P0) | satisfied | §1 step 7; §5 cancel/reconciliation (state inconsistency finding 10). |
| FR-14 (P0) | satisfied | §5 fence/lease, explicit needs-reconciliation. |
| FR-15 (P0) | satisfied | §1 steps 8-10; §7.2 candidate SHA authority. |
| FR-16 (P0) | satisfied | §6.1 two-phase SHA gate, stale and error deny. |
| FR-17 (P0) | satisfied | §1 distinct human; §8 execution no reviewer role. |
| FR-18 (P0) | satisfied | §6.1 PR after gate, §7 status and human host merge. |
| FR-19 (P0) | satisfied | §7.1 one tested Git-host PR path. |
| FR-20 (P1, conditional P0) | deferred | Out of Scope 8: Git-host LLD, contract seam; **priority conflict finding 5** until pilot selects. |
| FR-21 (P0) | satisfied | §6.2 org/repo rules and overrides. |
| FR-22 (P0) | satisfied | §6.2 policy error deny; §6.3 same-transaction audit. |
| FR-23 (P0) | satisfied | §5 retention and §6.3 queryable append-only audit. |
| FR-24 (P0) | satisfied | §6.4 runner hard time/server concurrency. |
| FR-25 (P0) | **omitted** | §6.4 reservations/debits/caps/unknown exist, but *monetary threshold warning* absent; finding 9. |
| FR-26 (P0) | satisfied | §8 admin/member/reviewer and scoped runner. |
| FR-27 (P0) | satisfied | §1 full API path, §2 headless team API. |
| FR-28 (P0) | satisfied | §2 thin CLI delegates to server; §1 and §3.3 workflow/cancel. |
| FR-29 (P0) | satisfied | §9.1 versioned boundaries and server privilege separation. |
| FR-30 (P1) | deferred | Out of Scope 9: Enterprise HLD; presentation seam in §9; payer evidence UNRUN. |
| FR-31 (P0) | satisfied | §10.1 single Compose server/Postgres/runner, no required external engine. |
| FR-32 (P0) | satisfied | §11 heartbeat off by default, field-documented; wire fields are LLD. |
| FR-34 (P0) | satisfied | §8 core OIDC, user role, unrecognised user denied. |
| NFR-1 (P0) | satisfied | §10.1 install boundary; §11 stage measurements; benchmark Action Item 6 UNRUN. |
| NFR-2 (P0) | satisfied | §11 cold/warm checkout, container-ready, prompt-ready timings; measurements UNRUN. |
| NFR-3 (P0) | satisfied | §5 restart/no auto-pass and §11 failure handling. |
| NFR-4 (P0) | satisfied | §3.1-3.2 hardened tier; §11 explicit shared-kernel limitation. |
| NFR-5 (P0) | satisfied | §3.3 credential scope, §6.3 redaction, §7.2 server-only Git writes, §8 roles. |
| NFR-6 (P0) | satisfied | §8 one org, many teams/repos, read/audit scoped by repo membership. |
| NFR-7 (P0) | satisfied | §3.2 own container, §6.4 ceilings, §9 Apache open core. |
| NFR-8 (P0) | satisfied | §5 artifact+DB backup; §10.2 migrations/restore (audit rollback conflict finding 11). |
| NFR-9 (P0) | satisfied | §5 no required non-OSI backend; §9 Apache-only core build. |
| NFR-10 (P0) | satisfied | §5 indefinite basic-audit retention, §6.3 atomic write and redaction (finding 11). |

**Tally: 38 satisfied by design, 4 deferred with P1 reason and design seam, 1 omitted (FR-25 monetary warning).** Experimental acceptance remains unmeasured and approval is separately held (finding 2).

## What is strong

- §1, §5 and §6 solve the nontrivial sequencing problem: candidate SHA captured, checks on that SHA before PR, human approval before PR, PR-triggered checks after PR, then host-authorised human merge. The fresh-executor pre-PR check and at-least-once outbox avoid common false exactly-once/CI-before-PR promises.
- §2 faithfully preserves R2’s B-inside-C preference and E’s two reuse paths; bundled AGPL, BSL and SSPL cases are distinguished from operator-installed out-of-process integration. The open core retains gates, budgets, audit, OIDC and both backends.
- §3 correctly distinguishes checkout separation from security isolation, records provider boot versus workload-ready latency and makes default container eligibility expressly conditional on an UNRUN threat review. Twelve invariants are explicit and separately referenced.
- §5 and §11 model fencing and ambiguous writer recovery rather than silently creating a second active writer; §5 decides indefinitely retained basic audit separately from finite payload retention. §8 keeps P0 single-org while reserving tenancy and avoids claiming an air gap.
- All seven R3 §G experiments are individually UNRUN in §P0 Scope; 14 assumptions disclose dependencies. All 12 action items have an owner role; most name the decision they block. No em dash, extra URL, operator identity, class name, config key or test filename found in the HLD body.

**Audits:** 58 individually graded citation claims spanning all design areas and every one of the 25 tagged BLOCKING cons; 17 sub-decisions, **7 clean / 10 with findings** (including the documented external approval hold); 33 active FR + 10 NFR, **38 satisfied by design / 4 deferred / 1 omitted**; **13 BLOCKING / 7 ADVISORY findings**. PRD FR-33 is withdrawn.

## Learnings / mistakes

- Initially treated the unresolved pilot tier as an HLD contradiction; checked HLD §3.2 and PRD handoff 3 and corrected it to a separate approval hold. Rule: distinguish honest uncertainty from an unsupported claim.
- Initially proposed missing Coder and Codespaces as independent default-tier options; checked R2 §Options and §Handoffs and found the clearer missing architectural option was full C, while Codespaces belongs in an advisory provider-contract gap. Rule: compare at the option set’s actual decision level.
- An initial audit marked §9.1 clean based on native plugin analysis; checked the explicit FR-29 text and found sandboxed **in-server** Wasm breaks the same invariant. Rule: read the invariant against every surviving option, not just every eliminated one.
- Confirmed Temporal’s production guide permits Docker deployment and Trigger.dev self-host consists of containers; did not infer from extra services that a separate cluster outside the product Compose is mandatory. Rule: source the exact elimination, not a stronger paraphrase.
- Initial citation audit had 48 claims but omitted §8 and §11; extended to 58 after checking section coverage. Rule: sample across every design area, not merely every BLOCKING con.

- Caught the HLD treating a database connection as automatic approval-write authority; checked PRD identity rule against official Postgres privilege grants. Rule: eliminate the actual privilege set, not a broader deployment assumption.
- Caught one invariant tagged BLOCKING for native plugins but ADVISORY for still-in-server Wasm; compared both against the exact PRD FR-29 exclusion. Rule: propagate invariants to attractive survivor variants as well as rejected options.
- Caught omitted monetary warnings by checking each PRD FR rather than trusting the HLD Requirements summary. Rule: acceptance behavior, not a requirement-ID mention, is coverage.
