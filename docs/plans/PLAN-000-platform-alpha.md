# PLAN-000 Platform alpha implementation plan

> Status: DRAFT v2 (re-verified 2026-11-27: REVISE; F2, F9 partly closed, F19 blocking; see the review file). HOLD inherited from HLD-000: the seven R3 §G experiments are UNRUN; no CODE story may start until the maintainer lifts that hold. Drafted under the HLD's "plans may be drafted" clause.

## Source design docs

Statuses are quoted from each file as read on 2026-11-27. If one changes, this plan is re-checked before use.

| Doc | Status and version |
|---|---|
| `docs/design/HLD-000-platform-overview.md` | "APPROVED v3 on document quality"; "HOLD: the seven R3 §G experiments are UNRUN"; "Implementation plans may be drafted against this HLD but no code is written while the hold stands." |
| `docs/design/PRD-000-platform.md` | "APPROVED v2"; "F22-F25 ADVISORY open" |
| VISION, R1, R2, R3 | "VISION APPROVED v2; PRD-000 APPROVED v2; R2 APPROVED v2; R3 APPROVED v4; R1 APPROVED v2" (HLD Inputs line); not re-read here, reached only through HLD and PRD citations |
| `docs/design/TEMPLATE-HLD.md`, `docs/design/TEMPLATE-LLD.md`, `docs/plans/README.md`, `docs/process/BRIEF.md` | Process files, unversioned |

`docs/plans/README.md` says "No plan is written before its HLD is approved." This draft relies on the drafting clause quoted above and cannot be approved for execution while the HOLD stands.

**HLD decisions this plan relies on.** Quoted in HLD-000's words; the bracket is the HLD section. In this plan a bare § is an HLD-000 Design section, I1 to I12 are its invariants, and FR/NFR ids are PRD-000's.

- **D1 [§2 Option 1, Preferred]** "B inside C, a baton team server and baton runners behind provider-neutral contracts"; reversible under Action Item 2 (Risk 2).
- **D2 [P0 Scope; §10.1 Option 1]** "One product binary in server or runner mode; one Compose file deploys server, Postgres and one runner; more runners join from other hosts".
- **D3 [§3.1 Option 1, Preferred]** "Every run gets its own checkout and branch (FR-9). Tier is a policy attribute shown before dispatch (FR-10)"; the tiers are `trusted-local`, "hardened container (checkout in the container's own volume), and a reserved microVM tier (Out of Scope 5)."
- **D4 [§3.2 Option 1]** "Hardened container by default, with a bounded eligibility envelope (Preferred, conditional on the pilot threat review)"; "Repos default to the container tier; `trusted-local` is a per-repo org-admin opt-in."; "The container tier takes cooperative and controlled work on the org's own repos; hostile code is refused in P0, not downgraded".
- **D5 [§3.3 Option 1, Preferred]** "The runner authenticates outbound, registers its platform, executors, adapters and labels, and claims runs under heartbeat-renewed leases with fencing epochs."; "Per-run credentials expire with the lease; the runner credential can call only these operations".
- **D6 [§4 Option 1, Preferred]** "One contract covers start, ordered events from a cursor, input, permission answers, cancel, resume, usage, artifacts, capability declaration and dispose". First pair: "the written vendor check selects".
- **D7 [§5 Option 1, Preferred]** "Postgres as system of record with queue, outbox, leases and a reconciler": "One transaction appends the event, changes run state, writes audit and enqueues the next step"; "Outbox workers run external effects (push, PR creation, status reads) at least once, deduplicated by tagged PR lookup and idempotency keys".
- **D8 [§5 state diagram]** States: Queued, Claimed, Preparing, Running, AwaitingInput, Candidate, Checks, Review, PROpen, MergeEligible, NeedsReconciliation, Cancelled, TimedOut, Failed, Rejected. "Terminal states: Cancelled, TimedOut, Failed, Rejected and merged; NeedsReconciliation is FR-13's explicitly unresolved state."
- **D9 [§5 retention]** "the product never deletes basic audit events in P0; run events and optional payloads (prompts, tool I/O, transcripts) have operator-set finite retention, default fixed in the policy-and-gates LLD, and each purge is audited."
- **D10 [§6.1 Option 1, Preferred]** "Pre-PR gate in baton: policy, pre-PR checks at the candidate SHA in a fresh executor, and a distinct human's approval of that SHA; baton then opens the PR; merge eligibility needs the host's PR-triggered checks passing at the same SHA, and a human merges under branch protection"; "A new commit stales both phases; an evaluation error, missing check or missing audit commit means ineligible."
- **D11 [§6.2 Option 1, Preferred]** "Bounded declarative document per org with per-repo overrides, evaluated in-process"; "Repo overrides may tighten freely; loosening needs an org admin; deny wins. Each version is hashed and recorded with every decision".
- **D12 [§6.3 Option 1, Preferred]** "Append-only audit events in the same transaction as the decision"; "The application's database role cannot update or delete audit rows (verified-by: LLD privilege test)."
- **D13 [§6.4 Option 1, Preferred]** "The server enforces concurrency at claim and monetary reservations per run, repo and org; the runner enforces time, CPU, memory and storage ceilings locally, even disconnected, and stops new agent calls at a supported cap."
- **D14 [§7.1 Option 1]** "Host contract with two P0 hosts, GitHub plus GitLab (Preferred, provisional: PRD option set 3 A)"; Option 2 is "preferred only if pilot demand is GitHub-only, and FR-20 then stays P1."
- **D15 [§7.2 Option 1, Preferred]** "Server pushes commits the runner transfers": "Runners hold only read-only, per-run clone credentials."
- **D16 [§8]** "One org per instance, every record carries an org identifier"; "Roles: org admin, member, reviewer; teams group members and repos; repo membership scopes every read, audit included"; "Execution identities have no role (I4)."
- **D17 [§9.1, §9.2 Option 1, Preferred]** "Compiled-in providers behind versioned contracts, plus out-of-process extensions"; "registration needs an admin grant"; "Enterprise module in the same repository, separate build": "the core never imports it".
- **D18 [§10.2 Option 1, Preferred]** "Forward-only migrations with dry-run, mandatory backup, restore as rollback"; "the server accepts the previous runner version"; "Upgrades run under a write freeze".

## Gates before code

HLD-000 Action Items 1 to 12. They are owned as the HLD states and are not agent stories; a story cites one as `AI-<n>`. "Closed" means the result is recorded in HLD-000's Action Items by the maintainer.

| Gate | Owner (HLD) | Subject (HLD, verbatim) | HLD "Blocks" (verbatim) | Blocks in this plan |
|---|---|---|---|---|
| AI-1 | maintainer | "Run and record all seven R3 §G experiments." | "HLD-000 approval (VISION §How we will know) and every UNRUN assumption." | The HOLD: every CODE, TEST and OPS story. M0 DOC stories may be drafted and reviewed. |
| AI-2 | HLD-000 author with the security reviewer, before approval | "B-versus-E spike: a Paseo-class daemon and the Coder API against leases, fencing, outbound-only runners and I2." | "§2 choice among Options 1, 3 and 4." | Approval of S0.1, S0.6 and S0.7 and, through them, every runner story. If it selects §2 Option 3 or 4, Risk 2 applies. |
| AI-3 | security reviewer | "Pilot threat review (experiment record row 8) with R3 §G item 1, recording the pilot tier, unresolved until then." | "approval of §3.2's conditional default, pilot eligibility and whether the microVM tier enters the alpha (Assumption 2)." | S4.6 (default tier), S6.11 (pilot repo), S6.12. Until it closes, container-tier runs use controlled fixture repos only. If it brings the microVM tier into the alpha or rules vendor CLIs out, this plan is re-planned (Risk 3): it has no microVM story and substitutes no tier. |
| AI-4 | maintainer | "Written vendor check (R3 §G item 7)." | "§4 first agent pair (PRD option set 2)." | Approval of S0.8; S1.17 and S4.12. If the check leaves FR-6's two credential paths unmet for an agent, the maintainer settles it with the PRD owner before S1.17; no consumer-login path is added. |
| AI-5 | maintainer | "Partner repo survey." | "approval of §7.1 Option 1 or 2 and FR-20 priority (PRD option set 3)." | Approval of S0.10 (host selection); whether S5.16 and S5.17 are P0 (option set 3 resolved to A before HLD approval). |
| AI-6 | runner LLD author | "NFR-1 install and NFR-2 latency benchmarks on the reference runner, including Assumption 8's rootless runtime." | "alpha release and §10.1's install target." | Rootless part: recorded inside S0.7, so M4's container stories wait on it. Benchmark part: S6.12 and the M6 exit. |
| AI-7 | policy-and-gates LLD author | "Default retention values for run events and payloads." | "the §5 retention default." | Closed inside S0.9; blocks S5.11 (purge). |
| AI-8 | Git-host LLD author | "Primary-source check of GitLab protection, stale-approval and status semantics." | "§7.1 Option 1's GitLab mapping (FR-20)." | The GitLab mapping in S0.10; S5.16 and S5.17. |
| AI-9 | execution HLD author | "Sources for a self-hosted user-space-kernel runtime and a Wasm-class plugin runtime." | "reassessing §3.2 Option 4 and an out-of-process Wasm host under §9.1 Option 1." | No story: both are outside P0. |
| AI-10 | maintainer | "Trigger.dev server licence (UNVERIFIED)." | "whether §5 Option 5 may ship as an optional backend under I2." | No story: §5 Option 5 is not built. |
| AI-11 | maintainer | "Dependency-licence check and DCO in the build." | "admission of each core dependency under I2 (NFR-9)." | S1.3 and every later story that adds a dependency. S1.1 and S1.2 supply the checks the maintainer accepts; a core dependency either needs is admitted by the maintainer, on record, before it is added. |
| AI-12 | PRD owner | "PRD Open question 5 still frames container-default eligibility; §3.2 proposes a conditional default." | "closing PRD Open question 5 and VISION Open question 4." | S4.6 and S6.11, together with AI-3. |

## Milestones

Scope is P0 of the 12-week self-hosted alpha. The first milestone with code, M1, is the walking skeleton: the thinnest slice of the §1 workflow (task -> run -> diff -> gate -> PR) through every boundary, on one tier, one adapter and one host.

**No story exists for:** FR-7, FR-8 and FR-30 (P1), FR-33 (withdrawn), any HLD Out of Scope item 1 to 19, or a host for out-of-process extensions (S0.12 specifies that boundary; no P0 acceptance line needs one built). FR-20 is conditional: S0.10 specifies it; only S5.16 and S5.17 build and test it. The component HLDs that HLD-000 names are attached there to out-of-scope items only, so M0 writes LLDs that trace to HLD-000 directly (Risk 6).

**Estimates.** ESTIMATES, LOW confidence (nothing is built; the stack is undecided, S0.1): about two accepted stories per working day, rounded up to a half week; M5 leaves out the conditional S5.16 and S5.17. M0 runs under the HOLD, outside the PRD's 12-week clock ("Release follows approved HLD/LLD/plan and R3 §G validation"), which starts when AI-1 closes.

| # | Milestone | Stories | Estimate | Exit test (one person, one sitting) |
|---|---|---|---|---|
| M0 | Design and decisions (no code) | 17 | under HOLD | Every document from S0.1 to S0.17 has a status line reading APPROVED and an APPROVE verdict from a different model in its `.review.md`. |
| M1 | Walking skeleton | 26 | 3 weeks | After one Compose command, a member of a registered test repo with branch protection on uses the CLI over TLS to create a task and start a run (first adapter, `trusted-local` allowed by policy); a reviewer of that repo approves the candidate SHA through the API, and a user without the reviewer role is refused; exactly one PR holding the agent's commit appears on the host. Not deployable outside a development host (Risk 8). |
| M2 | Durable run lifecycle and recovery | 17 | 2 weeks | Kill the runner mid-run, restart the server and deliver a claim twice: one run record remains and the run resumes under a new epoch or shows NeedsReconciliation; a cancel ends Cancelled; a run past its time limit ends TimedOut. |
| M3 | Policy, gates, audit and roles | 22 | 2.5 weeks | Add a commit to an approved candidate: approval and checks read stale, and if the PR was not yet open none opens. An injected evaluation error blocks PR creation and merge eligibility. A runner credential is refused on approve. With the audit write forced to fail the gate stays ineligible. The audit query reconstructs a completed run. |
| M4 | Container tier and second adapter | 14 | 1.5 weeks | Both agents complete a run (to Candidate, diff readable) on both tiers; inside a container run the host checkout, host agent credential and daemon socket are absent; an undeclared capability shows unavailable or denied. |
| M5 | Budgets, extension boundaries, retention and telemetry | 17 | 1.5 weeks | An excess concurrent run stays Queued; a priced run warns at its threshold; an unsupported extension version is rejected; a fresh install sends no telemetry; S5.17 passes if FR-20 is P0. |
| M6 | Install, upgrade and alpha evidence | 12 | 1.5 weeks | From the install guide alone on a clean reference host: a first run completes; upgrade under the freeze from its own backup, verify a prior run and gate decision, roll back with the last pre-freeze audit event present; the FR-3 scenario passes with no Enterprise code, paid sandbox or workflow cluster; S6.12 reads NFR-1 PASS and holds the NFR-2 report; every P0 story (S5.17 if FR-20 is P0) is accepted and every applicable gate closed. |

## Stories

Conventions, stated once:

- **Type.** DOC (one document), CODE (one behaviour, red then green: a failing test for the acceptance line first, then the change that passes it), TEST (one scenario suite), OPS (build, deployment or evidence artefact).
- **DOC-OK** means: written in `docs/design/TEMPLATE-LLD.md` format, every item traced to an HLD-000 section, Option / Pros / Cons with `[BLOCKING]` or `[ADVISORY]` wherever more than one implementation is viable, reviewed by a different model with verdict APPROVE in `<doc>.review.md`, a Review log row appended, status line APPROVED. "decides" marks a choice HLD-000 leaves open.
- **Blocked** lists direct gates. A blocked DOC story may be drafted and reviewed but not approved until the gate closes; a blocked CODE, TEST or OPS story may not start. Gates also bind through Deps. AI-1 also binds TEST and OPS, which write executable artefacts.
- **"a member"** and **"a reviewer"** are, from S1.6 on, signed-in users holding that role with membership of the repo; every API read or write of a repo's records is refused to anyone else. S1.8, S1.19 and S1.24 test this; S3.3 extends it to teams, audit records and the CLI.
- **"tests pass"** means the whole suite run by S0.1's entry command passes, the story's new test included.
- No story names a language, schema, path or library; each story after M0 takes them from the M0 documents it reaches through its Deps, directly or through an earlier story. "API" is the team API; "the host" is the first Git host S0.10 selects.

### M0 Design and decisions (no code)

HLD-000 names four of these documents: S0.6 with S0.7 is its "runner LLD", S0.9 its "policy-and-gates LLD", S0.10 its "Git-host LLD", S0.12 the P0 part of its "extension-point LLD" (Out of Scope 10 left out).

| Id | Title | Type | Traces | Acceptance | Deps | Blocked |
|---|---|---|---|---|---|---|
| S0.1 | Implementation stack decision | DOC | NFR-9; §2, §9, I2, I7 | DOC-OK; decides: implementation language or languages, whether the CLI ships in the product binary, build, test and migration tooling, dependency-licence tool with its allow-list, DCO mechanism, repository layout, the one entry command that runs the suite | none | AI-2 |
| S0.2 | Durable state LLD | DOC | FR-14, NFR-3; §5, §8, §10.2 | DOC-OK; specifies a schema for the §5 record categories with an org identifier on every record, the run state machine in exactly D8's names and §5's transitions, resolving against §1 step 7, with no new state, the path an unconfirmed stop takes from AwaitingInput, skip-locked claims, leases, fencing epochs, outbox, idempotency keys, runner sequence numbers, the reconciler's criteria for a confirmed baton-owned orphan executor, database roles; decides: backing of the server-managed artifact store | S0.1 | none |
| S0.3 | Team API LLD | DOC | FR-1, FR-2, FR-15, FR-27; §1, §2 | DOC-OK; specifies an operation for every member and reviewer step of §1, each carried over TLS; decides: wire format, API versioning, event delivery to clients | S0.1 | none |
| S0.4 | Access-control LLD | DOC | FR-26, FR-34, NFR-5, NFR-6; §8 | DOC-OK; specifies OIDC sign-in, the user-delegated CLI token, the one-time install token, roles org admin, member and reviewer, teams, repo-membership scoping of every read, the fixture assignment of member and reviewer roles with repo membership that M1 enforces until M3 adds administration, agent API keys as secret references per org or repo; decides: OIDC flow for the CLI, secret storage, the role allowed for each operation HLD-000 does not assign (repo registration among them) | S0.1 | none |
| S0.5 | CLI LLD | DOC | FR-28; §2 | DOC-OK; specifies each FR-28 command as calls to S0.3's operations under a user-delegated token and the rule that the CLI holds no source of truth; decides: command surface beyond FR-28's list | S0.3, S0.4 | none |
| S0.6 | Runner LLD: runner-to-server contract | DOC | FR-12, FR-13, FR-14; §3.3, §7.2, §10.2 | DOC-OK; specifies registration, runner eligibility for a run, leased claims, heartbeat renewal, fencing epochs, the up channel (ordered events, artifacts, candidate commits, terminal status), the down channel (cancels, permission decisions) with stop confirmation, credential expiry with the lease, runner-credential operation list, protocol versioning; decides: transport of the outbound channel, commit-transfer format, heartbeat and cancel-latency bounds | S0.2 | AI-2 |
| S0.7 | Runner LLD: executor providers | DOC | FR-9, FR-10, FR-11, NFR-4; §3, §6.4, §10.1 | DOC-OK; specifies the executor-provider contract (create, stop, dispose, list orphans; checkout import and artifact export; egress and secret capabilities; failure ownership: provider reports, reconciler decides), the worktree executor on a runner-owned mirror, container hardening (rootless or user-namespaced, dropped capabilities, seccomp, no daemon socket, quotas, egress policy) with the eligibility envelope it supports (hostile code refused), path confinement, local ceilings; decides: supported runner OS and architecture, container runtime, egress enforcement mechanism; records the Assumption 8 result the AI-6 owner measured on the reference runner and which of the two modes it used | S0.6 | AI-2 |
| S0.8 | Agent adapter LLD | DOC | FR-4, FR-5, FR-6; §4 | DOC-OK; specifies D6's contract operation by operation, capability declaration and versioning, golden event fixtures, per-run scoped API keys with no consumer login, the rule for an agent with no permission callback; names the pair the written vendor check selects and each adapter's transport; records how each selected adapter meets FR-6's API-key and enterprise-billing path support | S0.6, S0.7 | AI-4 |
| S0.9 | Policy-and-gates LLD | DOC | FR-16, FR-17, FR-21, FR-22, FR-24, FR-25; §5, §6.1, §6.2, §6.4 | DOC-OK; specifies the policy document (agents, tiers, tools, network, paths, time and concurrency, money, required approvals and checks), override and deny-wins rules, version hashing, tier eligibility (a run enters a tier only where policy affirmatively records its repo eligible; ineligible or unknown work is denied, never moved to another tier), evaluation and staleness for both gate phases, budget reservation, debit, warning and cap; decides: how a pre-PR check is declared, whether "distinct" excludes the submitting member, default retention values (closes AI-7) | S0.2, S0.4 | none |
| S0.10 | Git-host LLD | DOC | FR-18, FR-19, FR-20; §7.1, §7.2 | DOC-OK; specifies the host contract (read-only clone credentials, candidate-branch push, PR creation, PR-triggered status and protection reads, webhooks with polling as the floor), a contract test suite, the first host's mapping; decides: form of the server's Git-host identity, per-run credential issuance, PR tagging; records the surveyed host selection and, if a second host is selected, its mapping from primary sources (closes AI-8) | S0.2 | AI-5 |
| S0.11 | Audit LLD | DOC | FR-22, FR-23, NFR-5, NFR-10; §5, §6.3 | DOC-OK; specifies the field inventory (who acted on what, decision and reason, policy version, candidate SHA, order, redaction), the same-transaction write path, secret scrubbing, the database-role privilege test, query scoping, the payload store apart from audit, purge auditing | S0.2, S0.4 | none |
| S0.12 | Extension-point LLD | DOC | FR-29; §9.1, §9.2, I8 | DOC-OK; specifies versioned contracts for agents, executor providers, policy gates, roles and presentation metadata, rejection of unsupported versions, registration by admin grant, compiled-in P0 providers with out-of-process extensions only on runners or as separate services, the build rule that the core never imports the Enterprise module; decides: out-of-process extension protocol | S0.7, S0.8, S0.9 | none |
| S0.13 | Install LLD | DOC | FR-31, NFR-1; §10.1 | DOC-OK; specifies the Compose topology with where client-to-server TLS terminates, runner join from another host, the NFR-1 prerequisites; decides: configuration and secret provisioning | S0.2, S0.6 | none |
| S0.14 | Upgrade LLD | DOC | NFR-8; §10.2 | DOC-OK; specifies the recorded schema version with refusal of a newer schema, forward-only migrations with dry-run on an explicit operator step, backup of Postgres with the artifact store, the write freeze, restore as rollback, acceptance of the previous runner version, the compatibility artefact (the previous release's runner build) that the upgrade test runs | S0.2, S0.6 | none |
| S0.15 | Observability LLD | DOC | NFR-2; §11 | DOC-OK; specifies metrics for queue depth, lease expiries, reconciliations, gate outcomes and per-stage durations (checkout, container ready, ready-to-prompt), the NFR-2 report; decides: metrics exposure format | S0.2 | none |
| S0.16 | Opt-in heartbeat LLD | DOC | FR-32; §11 | DOC-OK; specifies a heartbeat that is off by default, the org-admin opt-in, the visible opt-in status, every field sent; decides: heartbeat destination and who operates it | S0.4 | none |
| S0.17 | Test harness decision | DOC | FR-3, NFR-3; §11 | DOC-OK; decides per test level: test doubles or live test accounts for the Git host, OIDC provider and agents, fault injection for audit-write, policy-evaluation, runner-death and duplicate-delivery cases, the end-to-end scenario runner, how red-then-green is evidenced for the reviewer | S0.1, S0.3 | none |

### M1 Walking skeleton

M1 defers breadth, not invariants. Deferred: role and policy administration (S0.4's fixture assigns roles and repo membership), running pre-PR checks (the fixture policy names none; one that does is ineligible), lease renewal and lease-loss recovery (until S2.1 an M1 run fits one lease interval), the container tier, the second adapter, budgets. Held from the first PR: authorised reviewer, gate current at the SHA when the PR opens, audit with each transition, eligible runner under a fenced lease, TLS on the API.

| Id | Title | Type | Traces | Acceptance | Deps | Blocked |
|---|---|---|---|---|---|---|
| S1.1 | Dependency-licence check in the build | OPS | NFR-9; §9, I2 | A build with a dependency outside S0.1's allow-list fails; a clean build passes; the check adds no core dependency the maintainer has not admitted on record (AI-11) | S0.1 | AI-1 |
| S1.2 | DCO check in the build | OPS | NFR-9; §9 | A change without sign-off fails the check; a signed-off change passes; the check adds no core dependency the maintainer has not admitted on record (AI-11) | S0.1 | AI-1 |
| S1.3 | One binary, server mode or runner mode | CODE | FR-31; §2, D2 | One built artefact starts in either mode and reports mode and version; tests pass | S1.1, S1.2 | AI-1, AI-11 |
| S1.4 | Schema baseline with recorded version | CODE | NFR-8; §5, §10.2 | The baseline applies only on the explicit operator step; the database records its schema version; the binary refuses a newer schema; tests pass | S0.2, S0.14, S1.3 | AI-1 |
| S1.5 | Compose file | OPS | FR-31; §10.1, I7 | S0.13's one Compose command on the reference host, with its documented prerequisites in place, starts server, Postgres and one runner; the server reports healthy; no separate workflow cluster runs | S0.13, S1.4 | AI-1 |
| S1.6 | OIDC sign-in on the API | CODE | FR-27, FR-34, NFR-5; §8, §11 | A member of the test OIDC provider obtains a user token and calls the API over TLS within the role and repo membership S0.4's fixture assigns; an unknown identity, a missing token or a token sent to the documented endpoint without TLS is denied; tests pass | S0.3, S0.4, S0.13, S0.17, S1.3 | AI-1 |
| S1.7 | Register a repo | CODE | FR-1; §7, §8 | A user whose role S0.4 allows registers a repo on the host and another signed-in user is refused; a task for an unregistered repo is refused; tests pass | S0.10, S1.4, S1.6 | AI-1 |
| S1.8 | Create a task (API) | CODE | FR-1, NFR-3; §1 step 1 | A member of the repo creates a task: the call returns a durable ID and initial state, and both survive a server restart; a signed-in user outside the repo's membership is refused; tests pass | S1.7 | AI-1 |
| S1.9 | CLI creates a task | CODE | FR-1, FR-28; §2 | The CLI, with a user-delegated token, creates a task and prints the ID and state the API returns; tests pass | S0.5, S1.8 | AI-1 |
| S1.10 | Policy admission at run start | CODE | FR-21, FR-10; §3.2, §6.2 | A run whose repo policy does not affirmatively allow its agent and its tier is denied and never queued, with no fallback to another tier; the decision records the policy version hash; tests pass | S0.9, S1.7 | AI-1 |
| S1.11 | Start a run in one transaction | CODE | FR-2, FR-22, NFR-10; §1 step 2, I9 | Start writes the run (Queued), its audit event and its queue entry together; with the audit write forced to fail, none is written; tests pass | S0.11, S1.8, S1.10 | AI-1 |
| S1.12 | Runner registers outbound | CODE | FR-12; §3.3 | The runner authenticates outbound and registers platform, executors, adapters and labels; its host listens on no inbound port; tests pass | S0.6, S1.3 | AI-1 |
| S1.13 | Per-run read-only clone credential | CODE | NFR-5; §7.2 | The server issues a per-run credential that can clone the repo; the host refuses a push made with it; tests pass | S1.7 | AI-1 |
| S1.14 | Agent API key as a secret reference | CODE | FR-6, NFR-5; §4, §8 | An agent key is stored as a secret reference per org or repo; its value appears in no API read; tests pass | S1.7 | AI-1 |
| S1.15 | Leased claim with fencing epoch | CODE | FR-12, FR-14; §3.3, §5 | A registered runner eligible under S0.6's rule (platform, executors, adapters, labels) claims the Queued run and receives run spec, fencing epoch, a lease with S0.6's expiry and per-run credentials; state Claimed; a second claim on it fails; with no runner connected, or only an ineligible one, the run stays Queued; tests pass | S1.11, S1.12, S1.13, S1.14 | AI-1 |
| S1.16 | Worktree executor prepares the checkout | CODE | FR-9, FR-10; §3.1 | Through the executor-provider contract the run gets a fresh worktree and branch from a runner-owned mirror; states Preparing, Running; the run reads `trusted-local`; tests pass | S0.7, S1.15 | AI-1 |
| S1.17 | First agent adapter | CODE | FR-4, FR-6; §4 | The first agent of the selected pair starts in the run's executor with a per-run scoped key and no consumer login; its output maps to normalized events matching golden fixtures; tests pass | S0.8, S1.16 | AI-1, AI-4 |
| S1.18 | Ordered event ingest | CODE | FR-2, FR-14; §3.3, §5 | The runner sends events with sequence numbers under its fencing epoch; a resent number is stored once; stored order equals sent order; an event under any epoch but the run's current one, or sent after the lease expired, is refused; tests pass | S1.15 | AI-1 |
| S1.19 | Run events from a cursor (API) | CODE | FR-2, FR-27, NFR-6; §4, §8 | A second member of the repo reads run state and events in order from a cursor; a user outside its membership is refused; tests pass | S1.18 | AI-1 |
| S1.20 | CLI starts a run | CODE | FR-2, FR-28 | The CLI starts a run with a named agent and executor and shows state and events equal to API reads; tests pass | S1.9, S1.19 | AI-1 |
| S1.21 | Candidate commit capture | CODE | FR-14, FR-15; §3.3, §7.2 | When the agent finishes, the runner commits outside the agent's control and uploads commits and diff; a later write by the agent process does not alter them; an upload under a stale epoch is refused; tests pass | S1.17, S1.18 | AI-1 |
| S1.22 | Server pushes the candidate branch | CODE | FR-15; §5, §7.2 | An outbox worker pushes the uploaded commits with the server's Git-host identity and records the SHA; state Candidate; running the worker twice leaves one branch at that SHA; tests pass | S1.21 | AI-1 |
| S1.23 | Candidate diff read (API) | CODE | FR-15, FR-27 | The API returns proposed files, diff and recorded SHA; the SHA equals the branch head on the host; tests pass | S1.22 | AI-1 |
| S1.24 | Approval bound to the candidate SHA | CODE | FR-16, FR-17, FR-26; §6.1, §8, I4, I9 | With a fixture policy naming no required pre-PR check the run passes Checks and waits in Review; a policy naming one stays ineligible (no check runs before M3); a reviewer's approval is stored against the SHA with its audit event in one transaction; a non-reviewer, a user outside the repo's membership, one S0.9 rules not distinct and a runner credential are each denied on approve; tests pass | S1.23 | AI-1 |
| S1.25 | One PR after the gate | CODE | FR-18, FR-19, FR-22; §5, §6.1, I6, I9 | The gate decision, its audit event and the PR outbox entry commit in one transaction before any host call; the outbox worker opens one PR only if, at that moment, a reviewer's approval is still current at the candidate head SHA; a repeated worker finds the tagged PR and opens none; with no approval, or with the SHA changed after approval, no PR exists; state PROpen; tests pass | S1.24 | AI-1 |
| S1.26 | Walking-skeleton end-to-end test | TEST | FR-3; §1 | The M1 exit scenario runs unattended against the Compose stack on S0.17's harness, the test repo's branch protection active throughout; tests pass | S1.5, S1.20, S1.25 | AI-1 |

### M2 Durable run lifecycle and recovery

| Id | Title | Type | Traces | Acceptance | Deps | Blocked |
|---|---|---|---|---|---|---|
| S2.1 | Lease renewal by heartbeat | CODE | FR-12, FR-14; §3.3 | A lease renews on each heartbeat and expires after S0.6's interval without one; tests pass | S1.15 | AI-1 |
| S2.2 | Stale-epoch terminal status rejected | CODE | FR-14; §3.3, §5 | Terminal status carrying an older epoch is rejected, as events (S1.18) and commits (S1.21) already are; tests pass | S1.21, S2.1 | AI-1 |
| S2.3 | Per-run credentials expire with the lease | CODE | NFR-5; §3.3 | After lease expiry the clone credential and the scoped agent key are refused; tests pass | S2.1 | AI-1 |
| S2.4 | Reconciler requeues a confirmed-dead run | CODE | FR-14; §5 | When the reconciler confirms process and executor dead, the run returns to Queued with a new epoch and one run record; tests pass | S0.7, S2.2 | AI-1 |
| S2.5 | Ambiguous lease loss enters NeedsReconciliation | CODE | FR-13, FR-14; §5 | From Claimed, Preparing or Running an unconfirmed loss enters NeedsReconciliation and no second writer is sent to that checkout; tests pass | S2.4 | AI-1 |
| S2.6 | Operator resolves NeedsReconciliation | CODE | FR-14; §5 | An operator confirming no writer returns the run to Queued; closing it ends Failed; each is audited; tests pass | S2.5 | AI-1 |
| S2.7 | Member cancel (API) | CODE | FR-13; §1 step 7 | Cancelling a Running run stops new tool work; it ends Cancelled, or NeedsReconciliation when the stop is unconfirmed; a second user sees the state; tests pass | S0.8, S2.5 | AI-1 |
| S2.8 | CLI cancels a run | CODE | FR-28, FR-13 | The CLI cancels a run and shows the state the API returns; tests pass | S1.20, S2.7 | AI-1 |
| S2.9 | Runner-local hard time limit | CODE | FR-24; §6.4, I10 | The runner stops a run past its time limit, disconnected from the server or not; it ends TimedOut; tests pass | S0.7, S2.2 | AI-1 |
| S2.10 | Failed state with a reason | CODE | FR-2; §5 | A preparation failure in Preparing and an agent failure in Running each end Failed with a recorded reason; tests pass | S1.17 | AI-1 |
| S2.11 | Executor disposal at run end | CODE | FR-9; §1 step 14, §3 | Every terminal state disposes the executor; its ID is recorded and no worktree remains; tests pass | S2.10 | AI-1 |
| S2.12 | Orphaned executor cleanup | CODE | FR-14; §11 | The reconciler matches recorded executor IDs against the provider's orphan list and disposes, with a record, only an executor meeting S0.2's criteria for a confirmed baton-owned orphan; an executor assigned to an active run and one baton did not create both survive; tests pass | S2.4, S2.11 | AI-1 |
| S2.13 | Outbox retry after Git-host failure | CODE | FR-18; §5, §11 | With the host unreachable, push and PR steps retry and complete on recovery with one branch and one PR; tests pass | S1.25 | AI-1 |
| S2.14 | Artifact upload to the server-managed store | CODE | NFR-8; §3.3, §5 | The runner uploads the run's log and transcript; a user S0.4 authorises reads them back; tests pass | S1.18 | AI-1 |
| S2.15 | Checkout path confinement | CODE | FR-9; §3.1 | A checkout path containing a symlink and one containing a traversal segment are each rejected under S0.7's rule, even when the path would stay inside the runner's checkout root; tests pass | S1.16 | AI-1 |
| S2.16 | Concurrent-run separation test | TEST | FR-2, FR-9 | Two simultaneous runs edit the same filename: distinct run IDs, ordered events and independent outcomes visible to a second user, no shared writable checkout, no crossed change; tests pass | S2.15 | AI-1 |
| S2.17 | Recovery fault suite | TEST | FR-12, FR-14, NFR-3; §11 | Forced runner death, duplicate delivery and forced server restart each leave one run record and a visible task, with safe resume or NeedsReconciliation; with every runner disconnected an unclaimed run stays Queued; tests pass | S2.6, S2.12 | AI-1 |

### M3 Policy, gates, audit and roles

| Id | Title | Type | Traces | Acceptance | Deps | Blocked |
|---|---|---|---|---|---|---|
| S3.1 | Install token binds the first admin | CODE | FR-26; §8 | The one-time install token makes the first signed-in user org admin; a second use is refused; tests pass | S1.6 | AI-1 |
| S3.2 | Role checks on API operations | CODE | FR-26, FR-34; §8 | A user without the org admin role cannot change policy and one without the reviewer role cannot approve; an org admin can change policy; a reviewer can decide a gate; every other operation follows S0.4's role table; tests pass | S3.1 | AI-1 |
| S3.3 | Repo-membership scoping of reads | CODE | NFR-6; §8 | A user outside a repo's membership cannot read its tasks, runs or audit records by API or CLI while another team's reads succeed; tests pass | S3.2 | AI-1 |
| S3.4 | Runner credential operation limit | CODE | FR-17, FR-26, NFR-5; §3.3, I4 | A runner credential succeeds only on S0.6's operation list and is denied on admin, approve and merge calls; tests pass | S0.6, S3.2 | AI-1 |
| S3.5 | Versioned policy document | CODE | FR-21, FR-23; §6.2 | An org admin stores a policy document covering every §6.2 rule category; an invalid one is refused; each version is hashed and the hash is recorded with every decision; tests pass | S1.10, S3.2 | AI-1 |
| S3.6 | Per-repo policy overrides | CODE | FR-21; §6.2 | A tightening override is accepted; a loosening one needs an org admin; deny wins; two repos with different policies give the same proposed run different recorded outcomes; tests pass | S3.5 | AI-1 |
| S3.7 | Policy evaluation failure denies | CODE | FR-22; §6.2, §11 | With evaluation made unavailable no run starts and no gate passes, and the operator sees an error; tests pass | S3.5 | AI-1 |
| S3.8 | Audit rows append-only by database role | CODE | FR-23, NFR-10; §6.3 | The application's database role is refused update and delete on audit rows (S0.11's privilege test); tests pass | S1.11 | AI-1 |
| S3.9 | Audit query for a run | CODE | FR-23; §6.3 | An authorised operator retrieves actor, action, decision, reason, policy version, candidate SHA and ordering for a completed run, whose trail holds every task, run, policy, approval and PR decision made for it; tests pass | S3.3, S3.8 | AI-1 |
| S3.10 | Secret scrubbing before audit write | CODE | NFR-5; §6.3 | A known test secret injected into run output is absent from every corresponding audit event; tests pass | S3.8 | AI-1 |
| S3.11 | Pre-PR checks in a fresh executor | CODE | FR-16; §6.1 | Required checks run at the candidate SHA in a new executor of the run's tier, not the agent's session; results bind to that SHA; a failed check ends Failed; tests pass | S2.11, S3.5 | AI-1 |
| S3.12 | Gate evaluation fails closed | CODE | FR-16, FR-22; §6.1, I5 | A missing, denied or stale approval (a SHA changed before the PR opens included), an approval from an identity S0.9 rules not distinct, a missing, failed or stale pre-PR check, an injected evaluation error or a missing audit commit each block PR creation; tests pass | S3.7, S3.11 | AI-1 |
| S3.13 | Reviewer rejection | CODE | FR-17; §5 | A reviewer's denial of the SHA ends the run Rejected with an audit event; no PR opens; tests pass | S3.12 | AI-1 |
| S3.14 | PR check status by webhook | CODE | FR-18; §7.1 | A host webhook for a PR-triggered check updates the status stored for that SHA only; tests pass | S1.25 | AI-1 |
| S3.15 | PR status polling floor | CODE | FR-18; §7.1, §11 | With webhooks dropped, polling brings stored status and head SHA to the host's values; tests pass | S3.14 | AI-1 |
| S3.16 | Merge eligibility at the same SHA | CODE | FR-16, FR-18, FR-22; §6.1, I6, I9 | MergeEligible needs approval, pre-PR checks and the host's PR-triggered required checks current at one SHA; a missing, failed or stale host check or an injected evaluation error blocks it; the transition commits only with its audit event; the API has no merge operation; a merge on the host is recorded as merged; tests pass | S3.12, S3.15 | AI-1 |
| S3.17 | New commit stales both phases | CODE | FR-16; §5, §6.1 | A commit added to the candidate branch returns the run from PROpen or MergeEligible to Candidate; prior approval and checks read stale; tests pass | S3.16 | AI-1 |
| S3.18 | Gate re-evaluation after restart | CODE | NFR-3; §5, §11 | After a server restart every gate reads ineligible until re-evaluated; none auto-passes; tests pass | S3.16 | AI-1 |
| S3.19 | CLI review view | CODE | FR-15, FR-28 | The CLI shows files, diff, SHA, gate state and PR link equal to API reads; tests pass | S1.20, S3.16 | AI-1 |
| S3.20 | Audit-failure injection test | TEST | FR-22, NFR-10; I9 | With audit persistence forced to fail during approval the gate stays ineligible; forced to fail at the PR transition, no PR work is queued and no PR opens; forced to fail at merge eligibility, the run stays PROpen; the operator sees an error each time; tests pass | S3.16 | AI-1 |
| S3.21 | Scripted API workflow test | TEST | FR-27 | A scripted client with no CLI or desktop session creates a task, observes a run, reads a diff, records an approval and retrieves the PR; tests pass | S3.16 | AI-1 |
| S3.22 | Gate end-to-end test | TEST | FR-16, FR-17, FR-18, FR-26 | The M3 exit scenario passes on the host with branch protection active throughout; tests pass | S3.4, S3.9, S3.17, S3.20 | AI-1 |

### M4 Container tier and second adapter

Until AI-3 closes, every container-tier run in M4 uses a controlled fixture repo of the test organisation; a pilot repo first runs in S6.11.

| Id | Title | Type | Traces | Acceptance | Deps | Blocked |
|---|---|---|---|---|---|---|
| S4.1 | Container executor with its own checkout volume | CODE | FR-11, NFR-4; §3.1, §3.2 | Through the executor-provider contract a container-tier run gets its checkout in the container's own volume with no host mount; tests pass | S0.7, S2.11 | AI-1 |
| S4.2 | Container hardening profile | CODE | FR-11, NFR-4; §3.2, §10.1, I12 | The container runs rootless or user-namespaced with dropped capabilities, seccomp and quotas; no executor receives the daemon socket; tests pass | S4.1 | AI-1 |
| S4.3 | Default-deny egress with allowlists | CODE | FR-21; §3.2, §11 | Inside a container run a destination outside the policy's model-provider and package allowlists is unreachable and an allowlisted one is reachable; tests pass | S3.5, S4.2 | AI-1 |
| S4.4 | Runner-local resource ceilings | CODE | FR-24; §6.4, I10 | The runner, even disconnected, stops a run that exceeds its CPU, memory or storage ceiling; tests pass | S4.2 | AI-1 |
| S4.5 | Tier label before dispatch | CODE | FR-10, FR-11, NFR-4; §3.1, §3.2, §11 | API and CLI show the tier before dispatch; a policy requiring sandboxing rejects a run that selects `trusted-local`; an unknown tier or executor is denied; a container-tier run is admitted only where policy records the repo eligible (S0.9), and ineligible or unknown work is denied, never moved to another tier; tests pass | S0.5, S3.6, S4.1 | AI-1 |
| S4.6 | Container tier as the repo default (conditional) | CODE | FR-11; §3.2 | CONDITIONAL on the pilot threat review confirming §3.2 Option 1: a repo with no tier rule runs on the hardened container tier and `trusted-local` needs a per-repo org-admin opt-in; tests pass | S4.5 | AI-1, AI-3, AI-12 |
| S4.7 | Pre-PR checks in a fresh container | CODE | FR-16; §6.1 | For a container-tier run the required checks execute at the candidate SHA in a new container; tests pass | S3.11, S4.2 | AI-1 |
| S4.8 | Isolation test | TEST | NFR-4, FR-10, FR-11 | On the supported runner, inside a sandbox-required run the host checkout, host agent credential and container daemon socket are absent, and the FR-10 policy denial holds; tests pass | S4.3, S4.5 | AI-1 |
| S4.9 | Capability declaration at registration | CODE | FR-5; §4 | Each adapter declares versioned capabilities; a feature the agent lacks shows unavailable before dispatch in API and CLI; tests pass | S1.17 | AI-1 |
| S4.10 | Undeclared capability requests denied | CODE | FR-5; §4 | A resume, steering or approval request the adapter did not declare is denied and audited, never reported as successful; tests pass | S4.9 | AI-1 |
| S4.11 | Tool-permission requests | CODE | FR-5, FR-13, FR-22, FR-23; §1 steps 6 and 7, §4, I9 | A permission request puts the run in AwaitingInput; policy or a member decides it; decision, state change, audit event and the answer for the runner commit in one transaction before the answer is delivered, and the run returns to Running; S3.9's query returns the decision, which leaves the change gate pending; with policy evaluation unavailable or the audit write rejected the tool is not granted; from AwaitingInput a confirmed stop ends Cancelled (cancel) or TimedOut (time limit), and an unconfirmed stop shows NeedsReconciliation by the path S0.2 specifies, never a completed stop; tests pass | S2.7, S2.9, S3.7, S3.9, S4.9 | AI-1 |
| S4.12 | Second agent adapter | CODE | FR-4, FR-6; §4 | The second agent of the selected pair starts in the run's executor on a permitted API-key or enterprise path with no consumer login; its events match golden fixtures; tests pass | S0.8, S4.9 | AI-1, AI-4 |
| S4.13 | Adapter-by-executor matrix test | TEST | FR-4, FR-6 | Each agent completes a run on the worktree executor and on the container executor with subscription login disabled: ordered events, state Candidate and a diff read through the API; at least one runs through its local CLI; each adapter's credential-path support is as S0.8 records; tests pass | S1.23, S4.3, S4.12 | AI-1 |
| S4.14 | Container-tier end-to-end test | TEST | FR-3, NFR-7 | On a controlled fixture repo the FR-3 scenario passes on the container tier with each adapter and no paid sandbox account; tests pass | S3.17, S4.7, S4.8, S4.11, S4.13 | AI-1 |

### M5 Budgets, extension boundaries, retention and telemetry

| Id | Title | Type | Traces | Acceptance | Deps | Blocked |
|---|---|---|---|---|---|---|
| S5.1 | Concurrency ceilings at claim | CODE | FR-24; §6.4, I10 | With a repo or org concurrency ceiling reached an excess run stays Queued and is claimed when a slot frees; tests pass | S2.1, S3.5 | AI-1 |
| S5.2 | Monetary reservation at admission | CODE | FR-25; §6.4 | Starting a run reserves budget per run, repo and org in the admission transaction; tests pass | S3.5 | AI-1 |
| S5.3 | Usage debit from adapter reports | CODE | FR-25; §4, §6.4 | Priced usage an adapter reports debits the reservation atomically and reads back through the API; tests pass | S4.9, S5.2 | AI-1 |
| S5.4 | Monetary warning threshold | CODE | FR-25; §6.4 | Crossing a configured org or repo threshold writes an audit warning and shows it on the run in API and CLI; tests pass | S0.5, S5.3 | AI-1 |
| S5.5 | Hard cap stops new agent calls | CODE | FR-25; §6.4 | At a configured hard cap with observable priced usage the runner stops new agent calls; tests pass | S5.3 | AI-1 |
| S5.6 | Opaque usage shows `unknown` | CODE | FR-25; §6.4 | A run whose adapter reports no priced usage displays `unknown` and no dollar cap is claimed for it; tests pass | S5.3 | AI-1 |
| S5.7 | Extension registry with versioned contracts | CODE | FR-29; §9.1, I8 | The registry lists contract versions for agents, executor providers, policy gates, roles and presentation metadata; a registration with an unsupported version is rejected; tests pass | S0.12, S4.9 | AI-1 |
| S5.8 | Registration by admin grant | CODE | FR-29; §9.1, I8 | A registration takes effect only with an org admin's grant; an unknown executor that registers itself gains no server privilege and no run; tests pass | S5.7 | AI-1 |
| S5.9 | Extension boundary test | TEST | FR-29; I8 | Both FR-29 acceptance cases pass end to end and the server process loads no extension code at run time; tests pass | S5.8 | AI-1 |
| S5.10 | Payload store apart from audit | CODE | NFR-10; §5 | Prompts, tool I/O and transcripts are stored apart from audit events; a user who can read audit but lacks payload access is refused the payload; tests pass | S2.14, S3.9 | AI-1 |
| S5.11 | Retention purge | CODE | NFR-10; §5 | Run events and payloads older than the operator-set retention are purged and each purge is audited; no audit event is deleted; defaults equal S0.9's values; tests pass | S5.10 | AI-1, AI-7 |
| S5.12 | Per-stage duration metrics | CODE | NFR-2; §11 | Metrics report queue depth, lease expiries, reconciliations, gate outcomes and durations for checkout, container ready and ready-to-prompt; tests pass | S0.15, S4.1 | AI-1 |
| S5.13 | Latency report | CODE | NFR-2; §11 | One command prints median and tail duration per recorded stage for cold and warm worktree and container runs and labels no provider boot time as agent-ready time; tests pass | S5.12 | AI-1 |
| S5.14 | Opt-in heartbeat | CODE | FR-32; §11 | Off by default; after an org admin opts in, the instance reports activation and adapter-use counters, shows its opt-in status and lists every field sent; tests pass | S0.16, S3.2 | AI-1 |
| S5.15 | No-telemetry test | TEST | FR-32, NFR-5 | A fresh install watched at its network boundary sends no product telemetry; tests pass | S5.14 | AI-1 |
| S5.16 | Second Git-host provider (conditional) | CODE | FR-20; §7.1 | CONDITIONAL: P0 only if the maintainer records, before HLD-000 approval, that the partner repo survey resolves PRD option set 3 to A; otherwise not built in the alpha (S0.10 keeps the seam). The second selected host (GitLab under §7.1 Option 1) passes S0.10's contract test suite; tests pass | S0.10, S3.16 | AI-1, AI-5, AI-8 |
| S5.17 | Second-host end-to-end test (conditional) | TEST | FR-20, FR-3 | CONDITIONAL as S5.16: the FR-3 scenario passes on both hosts without changing gate rules; tests pass | S5.16 | AI-1, AI-5, AI-8 |

### M6 Install, upgrade and alpha evidence

| Id | Title | Type | Traces | Acceptance | Deps | Blocked |
|---|---|---|---|---|---|---|
| S6.1 | Runner joins from another host | CODE | FR-12, FR-31; §10.1 | A runner started from the same image on a second host registers outbound and executes a queued run; tests pass | S0.13, S1.15 | AI-1 |
| S6.2 | Backup of Postgres with the artifact store | CODE | NFR-8; §5, §10.2 | One documented backup step captures Postgres and the artifact store; tests pass | S0.14, S5.10 | AI-1 |
| S6.3 | Restore from backup | CODE | NFR-8; §5, §11 | A restore into a test environment returns runs, policies, audit and artifacts to the backup point and writes an audit event for the restore; tests pass | S6.2 | AI-1 |
| S6.4 | Upgrade under a write freeze | CODE | NFR-8; §10.2 | Upgrade offers a dry-run; it freezes writes and claims, then takes that upgrade's own backup and does not proceed without it; it migrates only on the explicit operator step; writes and claims stay refused until the operator accepts the new version or rolls back; tests pass | S1.4, S6.3 | AI-1 |
| S6.5 | Previous runner version accepted | CODE | NFR-8; §3.3, §10.2 | The previous release's runner build, unmodified (S0.14's compatibility artefact), registers and completes a run against the upgraded server; runs from before the upgrade stay readable; tests pass | S6.4 | AI-1 |
| S6.6 | Upgrade drill | TEST | NFR-8 | Upgrade a populated instance: a write and a claim attempted during the freeze are refused; a prior run and gate decision verify; a rollback restores that upgrade's own backup in a test environment, and the last audit event and run state committed before the freeze are present; tests pass | S6.5 | AI-1 |
| S6.7 | Core-only build check | OPS | NFR-9, FR-3; §9.2, I2 | The release build contains no Enterprise module and fails if core code imports one; S1.1's licence check passes on the release artefact | S0.12, S1.1 | AI-1 |
| S6.8 | Install guide | DOC | FR-31, NFR-1; §10.1, §11 | States the NFR-1 prerequisites, the single Compose command, runner join, backup, upgrade and restore, the §11 tier table and the heartbeat fields; claims no 1.0, HA, air-gap, Enterprise readiness, hostile-code isolation or zero exfiltration; a different model following only the guide reaches a first run and records APPROVE | S5.14, S6.1, S6.4 | none |
| S6.9 | Team API reference | DOC | FR-27; §2 | Every operation S3.21 uses is documented; a different model re-creates S3.21's client from the reference alone and records APPROVE | S3.21 | none |
| S6.10 | Install benchmark protocol | DOC | NFR-1; §10.1 | A protocol and record sheet for at least three fresh installs from the NFR-1 prerequisites: each stage and failure recorded, median time to first successful run, a failed attempt counted as over 60 minutes; a different model records APPROVE | S6.8 | none |
| S6.11 | Clean-install release test | TEST | FR-3, FR-19, FR-28, FR-31, NFR-7, NFR-9 | On a reference host holding only the NFR-1 prerequisites, with no Enterprise licence, paid sandbox account or workflow cluster, the guide's Compose command then the FR-3 scenario passes on a pilot repo on the host, on a tier the recorded AI-3 result makes that repo eligible for; CLI results, read over TLS, equal API reads; tests pass | S2.8, S3.19, S4.14, S6.7, S6.8 | AI-1, AI-3, AI-12 |
| S6.12 | Alpha evidence report | OPS | NFR-1, NFR-2 | One report holds the benchmark results the AI-6 owner measured with S6.10 and S5.13 on the pilot repos the AI-6 owner names, and the pilot tier from AI-3; NFR-1 reads PASS only at a median of at most 60 minutes, and a miss reads FAIL with a VISION target review, not a waiver | S5.13, S6.10, S6.11 | AI-1, AI-3, AI-6 |

## Verification strategy

**Test levels.** Which doubles or live accounts each level uses is S0.17's decision.

1. Story level: red then green on every CODE story, both shown to the reviewer.
2. Contract level: golden event fixtures per adapter (S0.8), the host-contract suite (S0.10), the executor-provider contract (S0.7) and the runner protocol (S0.6), each run against every implementation.
3. Database level: transactions, skip-locked claims and the audit privilege test run against a real Postgres, not a substitute.
4. Fault level: injected audit-write, policy-evaluation, runner-death, duplicate-delivery, restart, webhook-loss and host-outage cases (S2.13, S2.17, S3.7, S3.15, S3.20).
5. End-to-end level: black-box scenarios against the Compose stack, one per milestone, kept and re-run in every later milestone.
6. Human level: the install guide walk-through (S6.8) and the NFR-1 and NFR-2 measurements (AI-6); these pass only when measured.

**Per milestone.**

| # | End-to-end scenario | Proved by | A reviewer runs |
|---|---|---|---|
| M0 | None; documents only | Different-model review of each document | A status-line search over the new documents; each `.review.md` verdict |
| M1 | §1 steps 1 to 5, 8, 10, 11; step 9 only as a policy naming no required check | S1.26 | The Compose command, then S1.26; checks one PR on the host |
| M2 | M1's scenario under faults, cancel and time limit | S2.16, S2.17; story tests of S2.7 (cancel) and S2.9 (time limit) | The fault suite, then the S2.7 and S2.9 tests; reads run state after each |
| M3 | §1 steps 9 to 13 with stale, evaluation-error and audit-failure cases | S3.20, S3.21, S3.22 | S3.22 with branch protection on; the audit query for the run |
| M4 | FR-3 on the container tier with each adapter | S4.8, S4.13, S4.14 | S4.8 on the supported runner; S4.14 |
| M5 | Budget, extension, retention and heartbeat cases on M4's stack | S5.9, S5.15, story tests of S5.1 to S5.6 and S5.11 | S5.9, S5.15; the S5.1 and S5.4 tests; the latency report command |
| M6 | Clean install, FR-3, backup, upgrade, restore | S6.6, S6.11, S6.12 | The install guide by hand; S6.6; S6.11; S6.12's NFR-1 verdict and NFR-2 report |

**Review of each story.** A model other than the author runs S0.1's entry command on a clean checkout, repeats the acceptance behaviour, checks the change against its DOC story and writes APPROVE or REVISE with BLOCKING or ADVISORY findings. The lead commits.

**Traceability.** Every FR and NFR in PRD-000. Specification: the M0 row whose Traces names the id, repeated here only in parentheses. "Delivered by": the stories that build the behaviour. "Verified by": the stories whose acceptance lines together cover the PRD's Accept line. A story in parentheses is specification, supplemental or conditional evidence, as labelled, not a full check.

| Req | Priority | Delivered by | Verified by |
|---|---|---|---|
| FR-1 | P0 | S1.7, S1.8, S1.9 | S1.8, S1.9 |
| FR-2 | P0 | S1.11, S1.18, S1.19, S1.20, S2.10 | S2.16 |
| FR-3 | P0 | S1.26 as skeleton, then every P0 story | S4.14, S6.11 (supplemental: S6.7, core-only build; conditional: S5.17) |
| FR-4 | P0 | S1.17, S4.12 | S4.13 |
| FR-5 | P0 | S4.9, S4.10, S4.11 | S4.9, S4.10 |
| FR-6 | P0 | S1.14, S1.17, S4.12 | S4.13 |
| FR-7 | P1 | No story (Out of Scope 6); seam: the §4 contract | n/a in P0 |
| FR-8 | P1 | No story (Out of Scope 7); seam as FR-7 | n/a in P0 |
| FR-9 | P0 | S1.16, S2.11, S2.15 | S2.16 |
| FR-10 | P0 | S1.10, S1.16, S4.5 | S4.8 |
| FR-11 | P0 | S4.1, S4.2, S4.5, S4.6 | S4.8 |
| FR-12 | P0 | S1.12, S1.15, S2.1, S6.1 | S2.17 |
| FR-13 | P0 | S2.5, S2.7, S2.8 | S2.7, S4.11 |
| FR-14 | P0 | S1.15, S1.18, S1.21, S2.1, S2.2, S2.4, S2.5, S2.6, S2.12 | S2.17 |
| FR-15 | P0 | S1.21, S1.22, S1.23, S3.19 | S1.23, S3.19 |
| FR-16 | P0 | S1.24, S3.11, S3.12, S3.16, S3.17, S4.7 | S3.22 |
| FR-17 | P0 | S1.24, S3.4, S3.13 | S3.22 |
| FR-18 | P0 | S1.25, S2.13, S3.14, S3.15, S3.16 | S3.22 |
| FR-19 | P0 | S1.25 (specification: S0.10) | S6.11 |
| FR-20 | P1; "becomes P0 if option set 3 resolves to A before HLD approval" | S5.16 (conditional; specification: S0.10) | (conditional: S5.17) |
| FR-21 | P0 | S1.10, S3.5, S3.6, S4.3 | S3.6 |
| FR-22 | P0 | S1.11, S1.25, S3.7, S3.12, S3.16, S4.11 | S3.20 |
| FR-23 | P0 | S3.5, S3.8, S3.9, S4.11 | S3.9 |
| FR-24 | P0 | S2.9, S4.4, S5.1 | S2.9, S5.1 |
| FR-25 | P0 | S5.2, S5.3, S5.4, S5.5, S5.6 | S5.4, S5.5, S5.6 |
| FR-26 | P0 | S1.24, S3.1, S3.2, S3.4 | S3.2, S3.22 |
| FR-27 | P0 | S1.6, S1.19, S1.23, S6.9 | S3.21 |
| FR-28 | P0 | S1.9, S1.20, S2.8, S3.19 | S1.20, S2.8, S3.19, S6.11 |
| FR-29 | P0 | S5.7, S5.8 (specification: S0.12) | S5.9 |
| FR-30 | P1, Enterprise, contingent on PRD Assumption 1 | No story (Out of Scope 9). Seam: the presentation extension point of §9 | n/a in P0 |
| FR-31 | P0 | S1.3, S1.5, S6.1, S6.8 | S6.11 |
| FR-32 | P0 | S5.14 | S5.15 |
| FR-33 | withdrawn in PRD-000 | No story | n/a |
| FR-34 | P0 | S1.6, S3.2 | S1.6, S3.2 |
| NFR-1 | P0 | S6.8, S6.10 | S6.12 after AI-6 |
| NFR-2 | P0 | S5.12, S5.13 | S6.12 after AI-6 |
| NFR-3 | P0 | S1.8, S3.18 | S2.17, S3.18 |
| NFR-4 | P0 | S4.1, S4.2, S4.5 | S4.8 |
| NFR-5 | P0 | S1.6, S1.13, S1.14, S2.3, S3.4, S3.10 | S3.4, S3.10, S5.15 |
| NFR-6 | P0 | S1.19, S3.3 | S3.3 |
| NFR-7 | P0 | None dedicated: a constraint on every story | S4.14, S6.11 |
| NFR-8 | P0 | S1.4, S2.14, S6.2, S6.3, S6.4, S6.5 | S6.6 |
| NFR-9 | P0 | S1.1, S1.2, S6.7 | S6.11 |
| NFR-10 | P0 | S1.11, S3.8, S5.10, S5.11 | S3.20 |

## Risks

Plan-level only. HLD-000's product risks are not restated.

| # | Risk | Trigger | Response |
|---|---|---|---|
| 1 | The HOLD outlasts M0 | AI-1 is not closed when the last M0 document is approved | No CODE, TEST or OPS story starts. When results arrive, re-check each UNRUN assumption against the stories before M1; a failed assumption sends the affected milestone back to planning. |
| 2 | AI-2 selects §2 Option 3 or 4 | The spike records that reuse is faster without weakening I2, I3 or I5 | S0.1, S0.6 and S0.7 are redrafted; the runner stories of M1, M2 and M4 are re-planned before any is started. |
| 3 | AI-3 breaks the tier or egress assumption | The pilot threat review records hostile-code work, or strict no-egress (HLD Assumption 9) | Stop before M4; the maintainer commissions the execution HLD; this plan and its 12-week estimate are revised. S4.6 is not built on an unconfirmed default. |
| 4 | AI-4 closes late or rules an agent out | AI-4 is open when S1.16 is done | M1 stories that do not depend on S1.17 continue; M1 cannot exit. A swapped agent means S0.8 is revised and re-reviewed first. |
| 5 | Estimates are wrong | M1's exit test passes later than 1.5 times its estimate | Re-estimate from measured stories per week and report to the maintainer; no P0 requirement is dropped to recover time. |
| 6 | A component HLD is required after all | A reviewer or the maintainer rules that an LLD cannot trace straight to HLD-000 | Add the HLD as a DOC story ahead of the affected LLD in M0; code stories keep their Deps. |
| 7 | Seventeen documents before code hide integration errors | A CODE story finds an LLD wrong or two LLDs in conflict | The story stops until the LLDs are revised and re-reviewed. |
| 8 | M1 and M2 builds lack administration | Any wish to run a build outside a development host before M3 exits | Refuse: until M3 roles and repo membership are fixture-assigned, no required pre-PR check can run and host check status is not read. |
| 9 | Test accounts are missing | No Git-host test organisation, OIDC test client, agent API key or reference runner when S0.7 or M1 needs one | The maintainer provisions them; S0.17 names what a double can replace. |

## Review log

| Date | Reviewer (model) | Verdict | Summary of changes requested |
|---|---|---|---|
| 2026-11-27 | GPT-class reviewer | REVISE | Review 1, of DRAFT v1: 18 findings (15 BLOCKING, 3 ADVISORY). Invariants deferred past the first PR (authorised reviewer, gate current at the SHA, audit with each transition, fenced lease, TLS); tier eligibility ahead of the pilot decision; credential paths; upgrade rollback; missing specification dependencies; release evidence; literal quotation and traceability. Dispositions are in the table below; the reviewer re-verifies each. |

Review 1 dispositions (DRAFT v2). No story id was added, removed or renumbered: 125 stories before and after. S2.2 keeps its id and is retitled "Stale-epoch terminal status rejected". No finding was rebutted.

| Finding | Tag | Disposition (changed / rebutted) | Where (story ids, section) | What changed, in one line |
|---|---|---|---|---|
| F1 | BLOCKING | changed | S0.4, S1.6, S1.7, S1.8, S1.19, S1.24, S1.25, S1.26, S3.12, S3.16; Stories conventions; M1 note; M1 and M3 exit tests; Verification M1 and M3 rows; Risk 8 | M1 enforces S0.4's fixture member and reviewer roles with repo membership; only a reviewer's approval counts and it is re-checked at the head SHA when the PR opens; a policy naming a required check is ineligible before M3; branch protection stays on; S3.12 and S3.16 add the SHA-change and merge-eligibility evaluation-error cases. |
| F2 | BLOCKING | changed | S0.6, S1.15, S1.18, S1.21, S2.2; M1 note | Only an eligible runner claims, under a fencing epoch and a lease with S0.6's expiry; an ineligible runner leaves the run Queued; stale-epoch or post-expiry events and uploads are refused from M1; S2.2 narrows to terminal status. |
| F3 | BLOCKING | changed | S1.25, S3.9, S3.16, S3.20, S4.11 | Gate decision, audit event and PR outbox entry commit in one transaction before any host call; merge eligibility commits only with its audit event; a tool answer commits with decision, state and audit before delivery and is not granted when the audit write is rejected; S3.20 adds rejected-audit cases at the PR transition and at merge eligibility; S3.9's trail holds every decision kind. |
| F4 | BLOCKING | changed | S0.2, S0.6, S4.11 | S0.2 resolves the unconfirmed-stop path from AwaitingInput against §1 step 7 with no new state; S0.6 adds stop confirmation; S4.11 tests the return to Running, a confirmed stop and an unconfirmed stop that shows NeedsReconciliation. |
| F5 | BLOCKING | changed | S0.7, S0.9, S1.10, S4.5, S4.14, S6.11; M4 note; gate rows AI-3 and AI-12; Risk 3 | A run enters a tier only where policy affirmatively records its repo eligible, with no fallback tier; container tests use controlled fixture repos until AI-3 closes; S6.11 is blocked on AI-3 and AI-12 and runs the pilot repo on a tier the recorded AI-3 result makes eligible; a result that invalidates the plan sends it to re-planning. |
| F6 | BLOCKING | changed | S0.8, S4.13; gate row AI-4 | S0.8 records how each selected adapter meets FR-6's API-key and enterprise-billing path support; S4.13 checks it; infeasibility goes to the PRD owner before S1.17 and no consumer-login path is added. |
| F7 | BLOCKING | changed | S6.4, S6.6; M6 exit test | Upgrade freezes writes and claims before taking its own backup and keeps them refused until acceptance or rollback; the drill tests refusal during the freeze and rolls back from that backup with the last pre-freeze audit event and run state present. |
| F8 | BLOCKING | changed | S1.1, S1.2; gate row AI-11 | Neither build check adds a core dependency the maintainer has not admitted on record. |
| F9 | BLOCKING | changed | Deps of S2.4, S2.7, S2.9, S3.4, S4.5, S5.4, S4.13, S4.14, S6.11; S4.13 acceptance; Stories conventions; Verification M2 row; M4 exit test | Six document edges added (S2.4 to S0.7, S2.7 to S0.8, S2.9 to S0.7, S3.4 to S0.6, S4.5 to S0.5, S5.4 to S0.5) and five test prerequisites (S4.13 to S1.23; S4.14 to S3.17 and S4.8; S6.11 to S2.8 and S3.19); the convention sentence now says documents are reached through Deps, directly or through an earlier story; M2 lists the S2.7 and S2.9 story tests. |
| F10 | BLOCKING | changed | S1.5 | S0.13's one Compose command, with documented prerequisites, starts server, Postgres and one runner and the server reports healthy, with no separate workflow cluster; image layout stays with S0.13. |
| F11 | BLOCKING | changed | S5.16, S6.11, S6.12; M5 and M6 exit tests; Verification M6 row; FR-20 trace row; gate rows AI-5 and AI-6 | The M6 exit needs S6.12's NFR-1 PASS and NFR-2 report, every P0 story accepted and every applicable gate closed; S5.16 and S5.17 are P0 only if the maintainer records, before HLD-000 approval, that option set 3 resolves to A. |
| F12 | ADVISORY | changed | Gates before code table; Source design docs (VISION row); D3 | The gate table quotes each Action Item's subject and "Blocks" text verbatim beside this plan's own mapping; the Inputs line and D3 are quoted as exact spans with no ellipsis. |
| F13 | ADVISORY | changed | Traceability intro and rows FR-3, FR-11, FR-14, FR-19, FR-20, FR-22, FR-23, FR-26, FR-28, FR-29, NFR-5, NFR-6, NFR-7; S5.17 Traces; "No story exists for" paragraph | FR-28 is verified by S1.20, S2.8, S3.19 and S6.11; specification, supplemental and conditional evidence are labelled in parentheses; S6.7 and S5.17 appear under FR-3; S0.10 is named as FR-20's specification; the table mirrors every changed Traces cell. |
| F14 | BLOCKING | changed | S2.15 | A path containing a symlink and one containing a traversal segment are each rejected under S0.7's rule, even inside the checkout root. |
| F15 | ADVISORY | changed | S0.7 | Hardening reads "rootless or user-namespaced" and S0.7 records which of the two modes the AI-6 owner measured. |
| F16 | BLOCKING | changed | S0.3, S0.13, S1.6, S6.11; M1 exit test | S0.3 carries each operation over TLS and S0.13 specifies where client-to-server TLS terminates; S1.6 denies a token sent without TLS; S6.11 reads CLI results over TLS. |
| F17 | BLOCKING | changed | S0.2, S0.7, S2.12 | Disposal needs S0.2's criteria for a confirmed baton-owned orphan; an executor of an active run and one baton did not create both survive. |
| F18 | BLOCKING | changed | S0.14, S6.5 | S6.5 runs the previous release's unmodified runner build against the upgraded server and keeps the old-run read assertion; S0.14 describes that compatibility artefact. |

Also changed from the review's check tables, outside a numbered finding: S0.12 (compiled-in P0 providers, with out-of-process extensions only on runners or as separate services), S2.14 (reads "a user S0.4 authorises"), the Verification M5 row (adds the S5.1 and S5.4 tests) and the Verification fault-level line (now cites S2.13, S2.17, S3.7, S3.15 and S3.20, the stories that hold the injected cases it lists). Prose was shortened to keep the body under its byte limit: the sources table lost its "Used for" column, several decision quotations became shorter exact spans, and the estimates and review paragraphs were tightened. No acceptance criterion was removed.
