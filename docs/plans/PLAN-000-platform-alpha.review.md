# Independent review — PLAN-000 platform alpha

## Verdict

**REVISE.** The draft preserves the HLD HOLD, but it does not yet give agents a faithful, executable path after the maintainer lifts it. The blocking findings alone determine this verdict: the first PR can be opened without the required authorization and gate checks, several safety or compatibility requirements can be marked done without proving the specified behavior, and the dependency and release criteria omit required evidence. The advisory findings do not affect the verdict.

## Findings

1. **F1 [BLOCKING] — The walking skeleton can open a real PR without an authorized, fail-closed gate.** P112 says “Omitted on purpose until later milestones: roles, pre-PR checks (policy requires none here, so Checks passes with nothing to run)”; P139 accepts “a signed-in user's approval” and P337 confirms “until M3 any signed-in user can register a repo and approve.” P140 only checks “With approval current at the SHA”; P180 tests evaluation error for PR creation only, and P184 tests host-check failure, not an injected evaluation error at merge eligibility. H207 requires “A distinct authorised reviewer sees diff, SHA and checks and approves or rejects that SHA”; H208 says “With approval and checks current at that SHA and audit committed, the server opens exactly one PR”; H739 says “Every query is scoped by org and repo membership in the API layer”; R120 says “an injected gate evaluation error blocks PR creation and merge eligibility.” A development host and two signed-in users do not enforce those invariants. **Smallest fix:** put the minimum fixture member/reviewer and repo scopes in M1 via S0.4, require the authorized reviewer and a current same-SHA gate at PR effect time (atomic audit separately in F3), and use an explicit fixture policy with no required pre-PR checks (deny an unsupported nonempty requirement). Keep full administration and host status in M3, keep branch protection active on the M1 test repo (H154), and add merge-eligibility evaluation-error and SHA-change-before-PR assertions to the existing gate suite. Leave the submitter-exclusion detail to S0.9, not the reviewer role itself.
2. **F2 [BLOCKING] — Claims do not prove runner eligibility and fencing.** P130 accepts “A registered runner claims the Queued run”; P112 defers “lease expiry,” and P148 postpones rejecting “an older epoch”. H200 says “An eligible runner claims the run outbound under a fenced lease”; H441 requires “heartbeat-renewed leases with fencing epochs”; H583 says an ambiguous lease “enters NeedsReconciliation and never gets a second writer on that checkout”. **Smallest fix:** require a mismatched registered runner to leave the run queued; establish/verify the minimum expiring fenced lease and stale-write refusal on the M1 path, leaving fault recovery scenarios in M2.
3. **F3 [BLOCKING] — Tool grants and PR/decision transitions lack a fail-closed atomic-audit contract.** P206 accepts “the decision is audited” and “with policy evaluation unavailable the tool is denied”; it says nothing about an audit write failing before delivery of an allowed answer. Likewise P140 and P184 can pass without testing the audit outcome of the PR/eligibility transition; P177 tests read fields, not that task, permission and PR decisions appear. H157 requires “A privilege transition commits only with its audit event”; H550 says “One transaction appends the event, changes run state, writes audit and enqueues the next step”; R126 tests “with policy evaluation unavailable or the audit write rejected, a run cannot gain tools or pass a change gate”; R127 names “task, run, permission, policy, approval and PR decisions.” **Smallest fix:** make the existing permission and gate/outbox rows commit decision, state/audit and next work before answers/effects; add rejected-audit denial for granting a tool and preventing PR work to the existing fault suite, and exercise the named event kinds in S3.9. External host I/O stays at least once, outside the database transaction.
4. **F4 [BLOCKING] — AwaitingInput can report a successful stop without confirmation.** P206 says “cancel and the time limit end AwaitingInput as Cancelled and TimedOut”, with no unresolved case or return from permission resolution; P153 reserves “NeedsReconciliation when the stop is unconfirmed” for `Running`. H204 explicitly lists “NeedsReconciliation (stop unconfirmed)”; H559 shows `AwaitingInput --> Running`; R117 requires “cancelling an active run prevents new tool work and records cancellation or a named reconciliation state in the shared view.” H573–576 draw normal cancel/timeout transitions but do not draw the ambiguous path from AwaitingInput. **Smallest fix:** have S0.2/S0.6 resolve that diagram/detail against the HLD workflow before consuming code; test confirmed stop, unresolved stop and permission resolution without ever reporting a successful stop for an unconfirmed writer. Keep the existing states.
5. **F5 [BLOCKING] — Lower-trust eligibility can be bypassed before the pilot decision.** P196 only asks for a container volume; P200 checks sandbox-required selection and unknown tier/executor; P201 gates the *default* only; P247 already requires “the FR-3 scenario on a pilot repo” with no AI-3 dependency. H391 says “The container tier takes cooperative and controlled work on the org's own repos; hostile code is refused in P0, not downgraded” and “the pilot tier is unresolved (Assumption 2), and approval waits for it (Action Item 3).” H886 explicitly blocks “pilot eligibility”; H138 says “If strict no-egress is required, vendor CLIs are ineligible”. **Smallest fix:** in existing admission/tier stories and S0.7/S0.9 require affirmative approved eligibility, deny ineligible/unknown work without falling back, and restrict synthetic container tests to controlled fixtures until AI-3. Gate actual pilot tests/rollout on the recorded tier/egress decision, and on AI-12 wherever relying on the default. If the decision invalidates this plan, re-plan rather than add a microVM or substitute a tier here.
6. **F6 [BLOCKING] — Adapter acceptance can stop at one credential path for a shipped agent.** P99 specifies “per-run scoped API keys with no consumer login”; P132 tests “a per-run scoped key and no consumer login”; P207 permits “a permitted API-key or enterprise path with no consumer login”. P208 disables subscription login, but no row owns support for both paths. R110 says “Support API-key and enterprise-billing credential paths for each shipped agent; do not require consumer-subscription login”; its Accept line is “each shipped adapter completes a run on a permitted non-consumer billing path with subscription login disabled.” H542 leaves the pair to the written check. **Smallest fix:** have S0.8 record how each selected adapter satisfies the exact R110 support obligation (a permitted integration may serve both billing paths); keep the stated non-consumer run test and check the documented path support in the existing adapter matrix. If vendor results make the written requirement infeasible, use AI-4's selection/fallback and resolve it with the PRD owner before code, not by inventing an unsupported login path.
7. **F7 [BLOCKING] — Upgrade acceptance does not tie rollback to the frozen upgrade backup.** P238 accepts “One documented backup step captures Postgres and the artifact store”; P240 “refuses to proceed without a backup” and freezes writes “from backup until the operator accepts the new version”; P242 verifies restoring a test backup, not rollback of that exact frozen upgrade under refused writes. A prior ordinary backup can satisfy the letter of the upgrade criterion without preserving writes up to the upgrade boundary. H803 says “from backup until the operator accepts the new version the server refuses writes and claims, so a rollback restore loses no committed write, audit included”; H585 separates rollback from disaster restore. **Smallest fix:** S6.4/6 must establish the write-and-claim freeze before taking the specific upgrade backup, test that writes/claims are refused through acceptance or rollback, then restore that backup and verify the last pre-freeze committed audit/state. Keep S6.3's disaster restore case separate.
8. **F8 [BLOCKING] — The build-check bootstrap can add an unadmitted dependency.** P55 says “S1.3 and every later story that adds a dependency”; P116–117 introduce the checks before S1.3 with only AI-1, while P92 chooses a “dependency-licence tool with its allow-list”. If those checks introduce a dependency, P55 leaves that addition outside the gate. H894 blocks “admission of each core dependency under I2 (NFR-9)”; H258 says “dependencies still pass the licence check”. **Smallest fix:** require explicit maintainer approval before either bootstrap story adds any core dependency (or implement the check using already-admitted tooling); don't require the completed check as a prerequisite for writing that same check.
9. **F9 [BLOCKING] — The DAG omits specifying documents and executable test prerequisites.** P84 says “each takes them from the DOC story in its Deps”; S3.4 at P172 tests “S0.6's operation list” with only `S3.2` in Deps and no path to S0.6. The subject audit found five more missing DOC paths (check 5), including S2.9→S0.7 for executor ceilings. P267 claims the M2 fault suite covers “cancel, time limit”, yet P163's suite neither tests those cases nor depends on S2.7/S2.9. H441 fixes the operation boundary, H687 says “the runner enforces time, CPU, memory and storage ceilings locally, even disconnected,” and H204 defines the observable cancel/timeout results. **Smallest fix:** add the subject-specific predecessor edges in check 5 (or an explicit equivalent approved-spec barrier), and require only the actual implementation dependencies for each suite. For M2, either list and run the existing cancel/time-limit story tests at exit or include them in S2.17 with those edges. Tie M4/M6 claims to the tests that really exercise them; do not blindly add unrelated prerequisite edges.
10. **F10 [BLOCKING] — The Compose criterion leaves the image topology and success boundary ambiguous.** P120 says “starts server, Postgres and one runner from one image” and “nothing else is needed”. The acceptance can require Postgres to be embedded in the product image or treat the documented external credentials/runtime as unacceptable, neither dictated by H795 (“One Compose file: server, Postgres, one runner; more runners from the same image”) or R144 (“assumes a supported host, container runtime, runner resources, Git and agent credentials, a reachable OIDC provider with a test client, and documented Compose prerequisites”). **Smallest fix:** have S1.5 test that S0.13's one Compose command starts the three services and the server reports healthy on documented prerequisites, with no separate workflow cluster; leave image layout to the install LLD, preserving the same-image remote runner rule.
11. **F11 [BLOCKING] — Release exit can pass without required measured and conditional evidence.** P248 allows a completed report where “a miss reads FAIL”; P74 lists first run, upgrade/restore and FR-3 but no evidence verdict, despite P50 stating that AI-6 blocks “S6.12 and the alpha release.” P230 says S5.16 “exists only if the partner repo survey resolves PRD option set 3 to A” but omits the approval timing that P298 itself quotes. R103 says “**P0** must pass before the 12-week alpha”; R144 says “A miss fails this NFR and triggers a VISION target review, not a waiver”; R124 says “becomes P0 if option set 3 resolves to A before HLD approval”; H889 blocks “alpha release and §10.1's install target.” **Smallest fix:** make release exit require the S6.12 NFR-1 PASS and NFR-2 evidence, and completion of P0 and applicable gates, not merely a report artifact. Make the M5/release exit require S5.17 only under the exact recorded pre-approval FR-20 condition; otherwise retain the seam. Nothing in this fix authorizes release or code while the relevant gates stand.
12. **F12 [ADVISORY] — Source gate blockers and abridged decision quotations are not literal.** P43 labels the last column “Blocks in this plan”; for example, P53 says “No story: §3.2 Option 4 and a Wasm host are outside P0”, while H892 says “Blocks: reassessing §3.2 Option 4 and an out-of-process Wasm host under §9.1 Option 1.” All 12 owner strings match, but all 12 source blocker strings are replaced by plan impacts. P22's `` `trusted-local` ... `` and P13's inputs are not literal contiguous matches at H356/H8. **Smallest fix:** preserve the plan-impact column, add verbatim HLD subject/blocker text, and either quote exact source spans in P13/P22 or mark deletions as editorial. Operational gate failures are tracked separately in F5/F8/F11, not raised here again.
13. **F13 [ADVISORY] — The trace table sometimes overclaims or omits the actual proof.** P275 says “"Verified by" names the story whose acceptance repeats the PRD's Accept line.” P306 lists only S6.11, whose acceptance P247 says “CLI results equal API reads”; R132 explicitly says “the CLI starts and cancels a run, shows its diff/gate state and links to its PR, with results matching API reads.” The behavior is separately covered at P135/P154/P187. P281 also leaves out non-M0 S5.17 and S6.7 from the literal mapping, and P62 mentions S5.16/S5.17 only, missing the FR-20 spec trace in S0.10. **Smallest fix:** list the actual S1.20/S2.8/S3.19 CLI checks, and separate specifying, delivery and supplemental/conditional evidence labels so the matrix does not claim a full check from a partial acceptance. Keep intentional M0 omissions explicitly distinguished from missing evidence.
14. **F14 [BLOCKING] — Checkout confinement accepts a weaker path policy than the HLD.** P161 rejects “A checkout path that leaves the runner's checkout root by symlink or traversal”; that lets symlink/traversal paths pass if they resolve inside the root. H356 says “Checkout paths are confined; symlinks and traversal are rejected”. **Smallest fix:** use S0.7 semantics and require symlink and traversal rejection in S2.15 as distinct cases, even for paths that appear to remain under the root.
15. **F15 [ADVISORY] — The runner LLD shorthand hides the allowed runtime alternative.** P98 says “container hardening (rootless, dropped capabilities, seccomp, no daemon socket, quotas, egress policy)”; P197 instead tests “rootless or user-namespaced”; H795 says “it drives a rootless or user-namespaced container runtime, and no executor gets the daemon socket”. S0.7 owns the runtime choice and records the Assumption 8 evidence, so the apparent inconsistency in shorthand does not force a specific runtime if that choice is respected. **Smallest fix:** write “rootless or user-namespaced” in S0.7 as well and record which eligible mode the AI-6 owner measured.
16. **F16 [BLOCKING] — No story owns the client-to-server TLS boundary.** P121 tests “A member of the test OIDC provider obtains a user token and calls the API”; P124 uses “a user-delegated token”; S0.4 and S0.13 cover identity/secret provisioning (P95/P104) but neither assigns TLS. H818 says “client to server (OIDC, TLS)”; H238 shows HTTPS. The plan can mark sign-in, CLI and release install done on a plaintext connection. **Smallest fix:** have the install/API LLDs allocate TLS termination and have the existing sign-in/install acceptance require the token-bearing API path over TLS. Do not decide a termination library or topology in the CODE row.
17. **F17 [BLOCKING] — Orphan cleanup uses a list mismatch as authority to dispose.** P158 says the reconciler “disposes the unmatched” after comparing recorded IDs against the provider's orphan list. H350 assigns “failure ownership (provider reports, reconciler decides)”; H841 describes “Reconciler matches recorded executor IDs” and “Dispose and record”. An unexplained mismatch does not prove baton ownership or that the executor is no longer assigned to an active run. **Smallest fix:** require the S0.7/S0.2 reconciliation criteria for a confirmed baton-owned orphan and test that an active executor survives. Don't make ID mismatch by itself the disposal condition.
18. **F18 [BLOCKING] — A protocol-number test does not prove previous-runner compatibility.** P241 accepts “A runner on the previous protocol version registers and completes a run against the upgraded server”; the current binary can be made to speak that version and pass while the previous binary does not. H803 explicitly says “the server accepts the previous runner version; old runs stay readable.” **Smallest fix:** run the actual previous runner build in S6.5 against the upgraded server, retaining the old-run read assertion; describe the compatibility artifact in S0.14.

## Check tables

### 1. Requirement coverage

Sources: [plan](PLAN-000-platform-alpha.md), [HLD](../design/HLD-000-platform-overview.md), [PRD](../design/PRD-000-platform.md), [format](README.md). Below, `P`, `H` and `R` with a line number denote these three documents; review judgments are based on these sources, not on outside claims.

Script: parsed only the two PRD requirement sections, parsed story titles/Traces/acceptance before reading the plan matrix, then diffed the resulting literal IDs with the union of its Delivered/Verified columns. Output: **44 IDs; 39 unconditional P0; P0 without a literal story: `[]`; unknown IDs: `[]`.** A literal match is not acceptance evidence. `−` = row reference omitted from the matrix; no matrix-only IDs were found. Its stated omission of specifying M0 DOCs (P275) explains most deltas; no priority is substantively different.

| ID | PRD line; exact priority/withdrawal | Recomputed story IDs | Against matrix at P279–322 | Acceptance review |
|---|---|---|---|---|
| FR-1 | 105; P0 | S0.3, S1.7, S1.8, S1.9 | −S0.3 | ID/state and CLI equality S1.8–9; auth sequencing F1. |
| FR-2 | 106; P0 | S0.3, S1.11, S1.18, S1.19, S1.20, S2.10, S2.16 | −S0.3 | Concurrent outcomes S2.16; eligibility F2. |
| FR-3 | 107; P0 | S0.17, S1.26, S4.14, S5.17, S6.7, S6.11 | −S0.17, S5.17, S6.7; `all P0 stories` is not an ID | S6.11 calls full case; gate holes F1. |
| FR-4 | 108; P0 | S0.8, S1.17, S4.12, S4.13 | −S0.8 | Cross-tier matrix and local CLI S4.13, subject to AI-4. |
| FR-5 | 109; P0 | S0.8, S4.9, S4.10, S4.11 | −S0.8 | S4.9–10 unavailable/denied, not silent success. |
| FR-6 | 110; P0 | S0.8, S1.14, S1.17, S4.12, S4.13 | −S0.8 | S4.13 tests permitted non-consumer use, not support for both stated paths (F6). |
| FR-7 | 111; P1 | — | None | Seam only. |
| FR-8 | 112; P1 | — | None | Seam only. |
| FR-9 | 113; P0 | S0.7, S1.16, S2.11, S2.15, S2.16 | −S0.7 | Concurrent separate checkouts S2.16; branch S1.16. |
| FR-10 | 114; P0 | S0.7, S1.10, S1.16, S4.5, S4.8 | −S0.7 | Label/denial S4.5 and S4.8. |
| FR-11 | 115; P0 | S0.7, S4.1, S4.2, S4.6, S4.8 | −S0.7 | Own checkout and absence S4.1, S4.8; eligibility F5. |
| FR-12 | 116; P0 | S0.6, S1.12, S1.15, S2.1, S2.17, S6.1 | −S0.6 | Offline/online covered; ineligible registered runners and lease sequencing F2. |
| FR-13 | 117; P0 | S0.6, S2.5, S2.7, S2.8, S4.11 | −S0.6 | Running cancel S2.7; unconfirmed wait cancellation F4. |
| FR-14 | 118; P0 | S0.2, S0.6, S2.1, S2.2, S2.4, S2.5, S2.6, S2.12, S2.17 | −S0.2, S0.6 | Fault suite S2.17 and stale epoch S2.2; sequencing F2/F9 and orphan guard F17. |
| FR-15 | 119; P0 | S0.3, S1.21, S1.22, S1.23, S3.19 | −S0.3 | S1.23 and S3.19 check files/diff/exact SHA. |
| FR-16 | 120; P0 | S0.9, S1.24, S3.11, S3.12, S3.16, S3.17, S3.22, S4.7 | −S0.9 | Cases split; pre-PR/new-SHA and exit-test gaps F1. |
| FR-17 | 121; P0 | S0.9, S1.24, S3.4, S3.13, S3.22 | −S0.9 | Final permissions tested; M1 admits unauthorized approvals (F1). |
| FR-18 | 122; P0 | S0.10, S1.25, S2.13, S3.14, S3.15, S3.16, S3.22 | −S0.10 | One PR and later protection evidence; M1 gate F1. |
| FR-19 | 123; P0 | S0.10, S1.25, S6.11 | None | Pilot repo on selected host in S6.11. |
| FR-20 | 124; P1; becomes P0 if option set 3 resolves to A before HLD approval | S0.10, S5.16, S5.17 | −S0.10; condition quoted in matrix, no priority change | Both hosts on same rules if selected; timing/exit wiring F11. |
| FR-21 | 125; P0 | S0.9, S1.10, S3.5, S3.6, S4.3 | −S0.9 | Different decisions on two repos S3.6. |
| FR-22 | 126; P0 | S0.9, S0.11, S1.11, S3.7, S3.12, S3.20, S4.11 | −S0.9, S0.11 | Audit failure at approval tested; at tool grant not tested (F3). |
| FR-23 | 127; P0 | S0.11, S3.5, S3.8, S3.9 | −S0.11 | Reconstructs fields S3.9; full event coverage and authorization before commit F3. |
| FR-24 | 128; P0 | S0.9, S2.9, S4.4, S5.1 | −S0.9 | Disconnected timeout S2.9; excess queue S5.1. |
| FR-25 | 129; P0 | S0.9, S5.2, S5.3, S5.4, S5.5, S5.6 | −S0.9 | Warning, cap and opaque cases S5.4–6. |
| FR-26 | 130; P0 | S0.4, S3.1, S3.2, S3.4, S3.22 | −S0.4 | Final role checks S3.2/S3.4; M1 sequencing F1. |
| FR-27 | 131; P0 | S0.3, S1.6, S1.19, S1.23, S3.21, S6.9 | −S0.3 | Scripted workflow S3.21, docs-only replay S6.9. |
| FR-28 | 132; P0 | S0.5, S1.9, S1.20, S2.8, S3.19, S6.11 | −S0.5 | Distributed: start S1.20, cancel S2.8, diff/gate/PR S3.19; S6.11 alone isn't the full comparison. |
| FR-29 | 133; P0 | S0.12, S5.7, S5.8, S5.9 | None | Unsupported version and unknown executor checked; packaging fidelity below. |
| FR-30 | 134; P1 | — | Matrix appends `Enterprise, contingent on PRD Assumption 1` | No story; PRD condition in R134, no scope change. |
| FR-31 | 135; P0 | S0.13, S1.3, S1.5, S6.1, S6.8, S6.11 | −S0.13 | Guided clean Compose first run S6.11. |
| FR-32 | 136; P0 | S0.16, S5.14, S5.15 | −S0.16 | Negative network test and positive counters/status/fields. |
| FR-33 | 137; withdrawn, see Review log | — | Matrix wording `withdrawn in PRD-000` | No story, as required. |
| FR-34 | 138; P0 | S0.4, S1.6, S3.2 | −S0.4 | S1.6 plus role behavior S3.2. |
| NFR-1 | 144; P0 | S0.13, S6.8, S6.10, S6.12 | −S0.13 | Correct measured rule in stories; release exit does not require it (F11). |
| NFR-2 | 145; P0 | S0.15, S5.12, S5.13, S6.12 | −S0.15 | Per-stage cold/warm median/tail plus pilot report. |
| NFR-3 | 146; P0 | S0.2, S0.17, S1.8, S2.17, S3.18 | −S0.2, S0.17 | Restore task/recovery S2.17 and no auto-pass S3.18. |
| NFR-4 | 147; P0 | S0.7, S4.1, S4.2, S4.5, S4.8 | −S0.7 | S4.8 verifies mounts and policy denial on runner. |
| NFR-5 | 148; P0 | S0.4, S0.11, S1.13, S1.14, S2.3, S3.4, S3.10, S5.15 | −S0.4, S0.11 | Credential denials S3.4, redaction S3.10; unscoped earlier builds F1; missing client TLS F16. |
| NFR-6 | 149; P0 | S0.4, S3.3 | −S0.4 | S3.3 tests cross-team reads; absent before M3 (F1). |
| NFR-7 | 150; P0 | S4.14, S6.11 | None | S6.11 no paid account or licence. |
| NFR-8 | 151; P0 | S0.14, S1.4, S2.14, S6.2, S6.3, S6.4, S6.5, S6.6 | −S0.14 | S6.6 drill; pre-backup freeze sequencing F7. |
| NFR-9 | 152; P0 | S0.1, S1.1, S1.2, S6.7, S6.11 | −S0.1 | S6.11 clean, S6.7 release licence boundary; AI-11 sequencing F8. |
| NFR-10 | 153; P0 | S0.11, S1.11, S3.8, S3.20, S5.10, S5.11 | −S0.11 | S3.20 matches direct test; event atomicity outside approval F3. |

### 2. Decision fidelity

Script: literal substring search of every double-quoted decision; semicolon-separated results follow quote order. HLD search excludes its review log; source-status rows checked against the identified doc (P12 script also found PRD review-log occurrences, irrelevant to the status).

| Item | Plan line | Literal search result | Story fidelity |
|---|---|---|---|
| Source status | P11/P12 | H3; H3; H3 / R3; R3 | Holds, see check 4. |
| Source inputs/format | P13/P16 | P13 **NOT FOUND** as a contiguous quote (ellipsis; underlying words H8); P16 **NOT FOUND in HLD**, found README 10 | No architecture change. |
| D1 | P20 | H217; H267 | First-party baseline retained; AI-2 before approvals. |
| D2 | P21 | H42 | S1.3/S1.5/S6.1: single server/runner binary, Postgres in Compose, remote runner. Image detail under check 3. |
| D3 | P22 | **NOT FOUND** verbatim (ellipsis replaces H356 parenthetical) | S1.16/S4.1/S4.5 retain tier separation and no host mount; S2.15 only tests escapes, not all forbidden path forms (F14). |
| D4 | P23 | H389; H391; H391; H391 | Conditional default/egress retained; eligibility omission F5. |
| D5 | P24 | H439; H441; H441 | Outbound runner claims correctly; eligibility and partial lease sequencing F2. |
| D6 | P25 | H489; H491; H542 | Baton-owned D6 via S0.8, then S1.17/S4.12; permission behavior F3–F4, billing F6. |
| D7 | P26 | H548; H550; H550 | Postgres and at-least-once outbox retained; no competing store/queue; atomicity F3. |
| D8 | P27 | H583; diagram first hits: Queued H554, Claimed H555, Preparing H556, Running H557, AwaitingInput H558, Candidate H560, Checks H561, Review H562, PROpen H563, MergeEligible H564, NeedsReconciliation H568, Cancelled H573, TimedOut H575, Failed H572, Rejected H580 | Canonical names preserved; cancellation/timeout semantics F4. |
| D9 | P28 | H585 | S5.10–11 separate payloads, purge audited, no basic audit deletion, exact retention default left to S0.9/AI-7. |
| D10 | P29 | H649; H649; H649 | SHA-bound two phases intended, early PR gate/staleness acceptance F1. |
| D11 | P30 | H662; H662 | S3.5–7 retain declarative override/hash/deny. |
| D12 | P31 | H677; H677 | Database role negative test S3.8; transaction coverage F3. |
| D13 | P32 | H687; H687 | S2.9/S4.4/S5.1–6 place limits on runner and concurrency/accounting on server. |
| D14 | P33 | H710; H712 | S0.10/S5.16–17 conditional; release wiring F11. |
| D15 | P34 | H720; H720 | S1.13, S1.21–22 server-only writable identity. |
| D16 | P35 | H739; H737; H737; H737 | S0.2/4 and S3.1–4 keep single org + scoped reads; S0.2 says every record has org identifier, not a schema name. M1 permissive user path F1. |
| D17 | P36 | H764; H764; H774; H774 | S0.12/S5.7–9/S6.7 versioned/admin-granted seam and no runtime server loads, no build of later optional host; explicitly require the H764 compiled-in rule in the LLD (check 6). |
| D18 | P37 | H795; H803; H803; H803 | Compose, explicit forward migration, dry-run/backup/restore. Freeze acceptance can still admit a backup-window write (F7); version compatibility uses a protocol rather than a build (F18). |

### 3. Decisions reserved for LLDs

Script: scan of all 125 story rows (title, Traces, acceptance) for technology/mechanism names, protocol/credential/record words and path/literal candidates, followed by manual review of every row for unlabelled commitments. Logical records and display values below are not SQL identifiers or configuration keys.

| Candidate / script match IDs | HLD source or disposition |
|---|---|
| Implementation language, framework, library: `NONE` | Stack reserved by S0.1. No concrete schema/table/column names or config keys found. `schema`: S0.2, S0.14, S1.4; `schema version`: S0.14, S1.4 (H803); schema choice remains S0.2. |
| Compose: S0.13, S1.5, S1.26, S6.8, S6.11; Postgres: S0.14, S1.5, S6.2 | H42, H550, H795. S1.5 `from one image` (P120) is ambiguous: H795 specifies the same image for additional **runners**, not for Postgres. If one product image for server+runner is meant, name those two; packaging decision belongs to S0.13 (F10). |
| DCO S0.1/S1.2; GitLab S5.16; outbox S0.2/S1.22/S1.25; skip-locked S0.2 | H760; H710 conditional/AI-5 and AI-8; H550 for both Postgres mechanisms. |
| rootless and seccomp S0.7/S4.2; user-namespaced S4.2; runtime/OS choice S0.7 | H137, H387, H795; exact runtime/runner OS open and assigned to S0.7. `rootless` alone in S0.7 drops H795's allowed alternative, see F15. |
| OIDC S0.4/S0.17/S1.6; delegated token S0.5/S1.9; install token S0.4/S3.1; API key/secret references S0.4/S1.14/S4.12 | H31, H491, H737; exact OIDC flow/storage delegated to S0.4. H818 also fixes TLS, not a cipher/library; TLS is absent from the story rows (F16). |
| Webhook(s) S0.10/S3.14/S3.15, polling S0.10/S3.15; versioning S0.6/S6.5 | H710, H803; no concrete payload format in a code row. |
| Wire format S0.3; commit-transfer format S0.6; extension protocol S0.12; metrics exposure format S0.15; heartbeat destination S0.16, fields S6.8 | Open; inventoried as topics assigned to DOCs, not invented concrete values (H441/H764/H843/R136). |
| Org identifier S0.2; idempotency keys S0.2; sequence numbers S0.2/S1.18; fencing/epoch S1.15/S2.2/S2.4; lease expiry S2.3 | H739, H550, H441/H583. |
| Actor/action/decision/reason S3.9; candidate SHA S0.11/S1.24/S3.9/S3.11/S4.7; policy version/hash S0.11/S1.10/S3.9 | H677, H649, H662. No physical column spellings mandated. |
| Executor IDs S2.12; queue depth, lease expiries, gate outcomes, ready-to-prompt S0.15/S5.12 | H841/H843. |
| Display literals `trusted-local` S1.16/S4.5/S4.6; `unknown` S5.6 | H356/391; H687/R129. State list in check 2. |
| Path scan: no literal file paths in any story row; `checkout root` S2.15 | H356 requires confinement; root location from S0.7. P81 paths are format conventions, not product paths. No script selected a concrete path or wire format. |
| Allow-list/tool S0.1/S1.1; allowlists S4.3; command surface S0.5; config/secrets S0.13; retention values S0.9; threshold S5.4/cap S5.5 | Open values assigned to DOCs; mechanisms H760, H391, H662/H687, H585. No hard-coded names or values. |
| Other implementation-level readings from manual review | `unmatched` orphan disposal (S2.12) is an unsupported algorithmic condition, not an identifier (F17); treating `distinct` as a choice in S0.9 is legitimate only beyond the fixed authorized reviewer invariant (F1). `protocol version` used as proxy for `runner version` (S6.5) requires an actual previous-build check (F18). |

### 4. HOLD and gates

Script compared `Owner`, subject and `Blocks` text parsed from H884–895 to P45–56. Output: owner is **exact for 12/12**, blocker wording differs **12/12** (plan uses its own downstream-impact column). Corrective distinction: the exact source blocker and downstream plan mapping are different columns (F12). No executable lacking direct AI-1: `[]`.

| Gate | P/H lines | Owner / subject / source Blocks literal | Direct story gates printed by script | Operational result |
|---|---|---|---|---|
| AI-1 | 45/884 | exact / exact / different: “HLD-000 approval (VISION §How we will know) and every UNRUN assumption.” | Every CODE, TEST, OPS | PASS: P3/P82/P330 retain hold. P3 requires maintainer to lift it, not merely an experiment starting. |
| AI-2 | 46/885 | exact / different / different: “§2 choice among Options 1, 3 and 4.” | S0.1, S0.6, S0.7 | Drafting allowed; inherited for runner work; P331 explicitly replans. |
| AI-3 | 47/886 | exact / different / different: “approval of §3.2's conditional default, pilot eligibility and whether the microVM tier enters the alpha (Assumption 2).” | S4.6, S6.12 | Pilot scenarios S4.14/S6.11 can start without this result if settings/fixtures do not inherit S4.6; split dev-only and actual pilot evidence (F5/F11). |
| AI-4 | 48/887 | exact / different / different: “§4 first agent pair (PRD option set 2).” | S0.8, S1.17, S4.12 | Correct selection gate; adapter tests inherit. |
| AI-5 | 49/888 | exact / exact / different: “approval of §7.1 Option 1 or 2 and FR-20 priority (PRD option set 3).” | S0.10, S5.16, S5.17 | Gates first-host spec and second-host work; source's timing needs explicit carry-through (F11). |
| AI-6 | 50/889 | exact / exact / different: “alpha release and §10.1's install target.” | S6.12 | Rootless-spike evidence is in S0.7 itself, inherited by M4; don't gate the producer on itself. NFR-1/2 report can be missed by exit (F11). |
| AI-7 | 51/890 | exact / exact / different: “the §5 retention default.” | S5.11 | Correct: S0.9 owns/defaults and closes; purge waits. |
| AI-8 | 52/891 | exact / exact / different: “§7.1 Option 1's GitLab mapping (FR-20).” | S5.16, S5.17 | Correct: S0.10 owns mapping closure and the two consumers wait. |
| AI-9 | 53/892 | exact / different / different: “reassessing §3.2 Option 4 and an out-of-process Wasm host under §9.1 Option 1.” | none | Deferred options, not an omitted alpha gate. |
| AI-10 | 54/893 | exact / different / different: “whether §5 Option 5 may ship as an optional backend under I2.” | none | Optional backend not built. |
| AI-11 | 55/894 | exact / exact / different: “admission of each core dependency under I2 (NFR-9).” | S1.3 | P55 also globally applies to later dependency additions. No gate on its two producers is right; any dependencies of the producers themselves need the maintainer's admission first (F8). |
| AI-12 | 56/895 | exact / different / different: “closing PRD Open question 5 and VISION Open question 4.” | S4.6 | Does not displace AI-3. S4.6 waits on both, as intended. |

### 5. Dependency graph

Script: parsed `Deps` and `Blocked`, checked references and depth-first cycles, then computed transitive predecessors. A second pass assigned each CODE row one or more subject-specific M0 specs before checking reachability. A generic stack DOC is not a substitute for the relevant spec.

| Probe | Script output / assessment |
|---|---|
| Counts | **125**: DOC 20, CODE 86, TEST 14, OPS 5. |
| M0 | 17 DOC = 17. |
| M1 | 22 CODE, 1 TEST, 3 OPS = 26. |
| M2 | 15 CODE, 2 TEST = 17. |
| M3 | 19 CODE, 3 TEST = 22. |
| M4 | 11 CODE, 3 TEST = 14. |
| M5 | 14 CODE, 3 TEST = 17 (15 if second-host branch omitted). |
| M6 | 5 CODE, 2 TEST, 2 OPS, 3 DOC = 12. |
| Structural errors | Unknown story/gate `[]`; later milestones `[]`; later rows within same milestone `[]`; cycles `[]`. |
| Any transitive DOC | CODE without any predecessor DOC `[]`. Direct DOC: **17/86**: S1.4, S1.6, S1.7, S1.9, S1.10, S1.11, S1.12, S1.16, S1.17, S4.1, S4.12, S5.7, S5.12, S5.14, S5.16, S6.1, S6.2. P84 inaccurately implies direct edges throughout. |
| Subject-specific DOC closure | **80/86** have all assigned links; six missing pairs: S2.4→S0.7, S2.7→S0.8, S2.9→S0.7, S3.4→S0.6, S4.5→S0.5, S5.4→S0.5. S2.7 needs the adapter stop semantics, not selection of a specific vendor for all cancellation logic. F9. |
| Explicit acceptance IDs absent in closure | `[(S3.4, S0.6)]`. Its operation allowlist is unresolved without the runner LLD. F9. |
| Semantic dependency probes | S1.26 reaches S1.17/S1.24/S0.17, not S3.2/S3.11 (F1); S2.17 reaches neither S2.7 nor S2.9; S4.13 reaches S4.12 but not the completed candidate read S1.23; S4.14 reaches neither S4.8 nor S3.17/S3.18; S6.11 reaches neither S2.8 nor S3.19; wire only prerequisites of the test's actual cases (F9). |
| Effective release-gate links | S4.14 and S6.11: AI-1, AI-2, AI-4, AI-5, AI-11; **no AI-3/AI-12**. S6.12: AI-1, AI-2, AI-3, AI-4, AI-5, AI-6, AI-11. Neither S6.11 nor S6.12 reaches S6.6, S6.9, or conditional S5.17; those are milestone completion requirements, not inherently prerequisites for one clean-install test. F11. |

### 6. Story quality

Script sampled the first, one-third, two-thirds and last story in each milestone (28); defects in S2.12/S2.17/S3.22/S4.14/S6.12 prompted widening to **all 125**. The second pass read every acceptance for one behavior, one scenario suite/artefact or one DOC; the marker screen found **0/100 CODE/TEST missing `tests pass`**. No exact `as appropriate`, `robust`, or `properly` in the story rows. Script candidates and manual adjudication follow; multiple assertions on the same behavior are not independent stories. Every ID in each continuous milestone range was checked; rows without a flagged exception passed the quality screen (this is not an architecture or gate pass).

| Milestone | Initial IDs | Final tally / population | Quality result after widening |
|---|---|---|---|
| M0 | S0.1, S0.6, S0.12, S0.17 | 17/17 (S0.1–17) | Each one document with DOC-OK. `supported` in S0.7 delegates a platform choice, not subjective success; retain the allowed runtime alternatives (F15). S0.12 should state the compiled-in provider boundary and locations for the out-of-process seam (H764), not build the deferred host; the summary at P36 sets the correct rule. |
| M1 | S1.1, S1.9, S1.18, S1.26 | 26/26 (S1.1–26) | `clean` (S1.1) = allow-list compliant; `current` (S1.25) needs an actual SHA at effect time (F1). `nothing else is needed` (S1.5) is unbounded and conflicts with documented prerequisites if literal; clarify topology instead (F10). S1.26 references a runnable harness; its security acceptance is too weak (F1). |
| M2 | S2.1, S2.6, S2.12, S2.17 | 17/17 (S2.1–17) | `unmatched` (S2.12) does not say what counts as an orphan (F17); `authorised` on artifact reads (S2.14) should resolve to the LLD policy. Independent fault injections can be a suite, but the suite/exit does not provide matching cancel/timeout proof (F9). Confinement scope F14. |
| M3 | S3.1, S3.8, S3.15, S3.22 | 22/22 (S3.1–22) | `authorised` audit query (S3.9) is observable using repo membership; `current` checks (S3.16) are explicitly tied to SHA. S3.22 is one suite; attach the exact missing/errored gate cases (F1) and external gate dependency (F9). |
| M4 | S4.1, S4.5, S4.10, S4.14 | 14/14 (S4.1–14) | `supported` runner (S4.8) is defined by S0.7. S4.11 combines tool holds, allow/deny, audit failure coverage and terminal handling; use one permission suspension/resolution behavior with its existing cancel/timeout cases/ties (F3/F4). S4.14 references an observable PRD scenario but could run against ungated pilot scope (F5). |
| M5 | S5.1, S5.6, S5.12, S5.17 | 17/17 (S5.1–17) | Individual budget assertions, negative extension cases and telemetry checks are observable; S5.12's metric inventory is one metric surface (H843); S5.13 is one command; S5.17 has two host runs of one conditional suite, not two independent behaviors. |
| M6 | S6.1, S6.5, S6.9, S6.12 | 12/12 (S6.1–12) | `clean` host (S6.11) and `representative` repos (S6.12) must use the PRD prerequisites/AI-6 inputs, not reviewer discretion. S6.8–10 each one document with a reviewer exercise or explicit record fields. Upgrade sequence, prior-runner proof and release evidence need tightening (F7/F18/F11). |

### 7. Scope

Script crossed each story's title, Traces and acceptance against P1/withdrawn IDs, perimeter terms and conditional words; manual sweep covered every excluded item in H106–124 and R157–166.

| Source boundary | Script hits and result |
|---|---|
| P1/withdrawn | `S0.10, S5.16, S5.17` reference FR-20; no story hit for FR-7, FR-8, FR-30 or FR-33. No build of those P1 seams. |
| Second host | S0.10 AI-5; S5.16/S5.17 AI-1/5/8. R124: “becomes P0 if option set 3 resolves to A before HLD approval”. P298 copies it, but P230 only conditions on resolving to A; H710/888 place confirmation before approval. Require explicit approval record and conditional release evidence (F11). |
| First pair | S1.17/S4.12 `selected pair`, both AI-4; S0.8 waits for written check, downstream tests inherit. H542 first pair remains conditional; use permitted paths, with FR-6 obligations (F6). |
| Pilot tier | S4.6 explicitly conditional; AI-3/AI-12. S6.12 `pilot tier` AI-3/6. S6.11 `pilot repo` only direct AI-1, no AI-3 in closure; see F5/F11. H131/H138 and H391 can require redesign, not a microVM or no-egress build in this plan. |
| Tenancy/edition/plugins | Hits: S0.12, S4.12, S6.7, S6.8, S6.11. S4.12 `enterprise` is agent billing, not Enterprise module; other hits enforce packaging/exclusions. No hosted, embedding, multi-org, branding, marketplace or in-server untrusted plugins. P0 versioned seam does not require a shipped external host (R162). |
| Isolation/agent interaction | Hits: S0.8, S1.17, S2.17, S4.10, S4.12, S4.13, S6.8. No ACP/MCP ingestion, parity or consumer login; `resume`/`steering` are recovery or capability denials. No shipped hostile/microVM, extra OS, provider matrix or guaranteed exfiltration/opaque dollar cap; refusal needs to be enforced (F5). |
| Merge/clients/ops | Hits: S3.4, S3.16, S3.21, S6.8, S6.11. Negative merge/no desktop/no workflow cluster, not those features. No HA, air-gap, single-user no-Postgres, audit signing/export or alternate clients found. |

### 8. Milestones

Script joined P68–74 with P265–271 by milestone, and enumerated H198–211. Output: 7 matched exits, 14 source workflow steps, 12.0 estimated post-HOLD weeks, M1 claims steps `1 to 5, 8, 10, 11`; P112 expressly omits roles, pre-PR checks, lease expiry, container tier, second adapter and budgets. Manual coverage below distinguishes an intentional thin slice from an invariant violation.

| Milestone / §1 step | Lines | Observability, prerequisite or coverage result |
|---|---|---|
| M0 | P68/P265 | Runnable by human status/review search; external gate closure may delay approvals; no code needed. |
| M1 | P69/P266 | Compose+S1.26 with test OIDC identities and one test repo/agent/runner; measurable one PR. Current thin slice permits unauthorized approval; cannot assume M3 to validate it (F1). |
| M2 | P70/P267 | Faults and outcomes observable; S2.16/S2.17 do not test the listed cancel/time-limit exits, and no path from S2.17 to their implementations. Test by S2.7/S2.9 or include them in existing suite (F9). |
| M3 | P71/P268 | Staleness, audit failure, denied runner and reconstructable query are observable; claimed steps 9–13 need actual authorized reviewer/check/status/protection fixtures (F1). No later-milestone implementation required on trusted-local. |
| M4 | P72/P269 | Isolation+adapter matrix are measurable; “Both agents complete a run on both tiers” is stronger than “produce event streams and diffs” unless termination is also asserted. Full container FR-3 remains testable on a controlled fixture; a pilot needs AI-3 (F5/F9). |
| M5 | P73/P270 | All explicit outcomes measurable with S5.1/S5.4/S5.9/S5.15; `A reviewer runs` lists extension/telemetry/latency, not the budget proof. Conditional host proof belongs to M5 or release only when eligible (F11). |
| M6 | P74/P271 | Install and restore observable, using existing stories; proof column names S6.12 but runnable exit omits its benchmark verdict and does not require S5.17 if selected. Complete all required stories and gates (F11). |
| §1.1 | H198 | Yes: S1.7–9/S1.20, configured repo plus task/run, member role only nominal before M3 (F1). |
| §1.2 | H199 | Partial: S1.10–11 policy/audit/enqueue; money reservation intentionally in S5.2. |
| §1.3 | H200 | Partial: outbound claim/spec/epoch/credentials S1.12–15; eligibility and lease safety F2. |
| §1.4 | H201 | Yes for one tier: S1.16 worktree; container intentionally M4. |
| §1.5 | H202 | Yes: S1.17–19 adapter and ordered events. |
| §1.6 | H203 | Deferred to S4.11; happy-path fixture need not prompt. |
| §1.7 | H204 | Deferred to M2/M4; happy-path fixture need not cancel or time out. |
| §1.8 | H205 | Yes: S1.21–23 runner commit transfer, server push, diff and recorded SHA. |
| §1.9 | H206 | No: P112 sets zero required checks; qualify it as a synthetic no-check policy, not a bypass where checks are required (F1). |
| §1.10 | H207 | Partial: SHA and second identity S1.24/exit, but role authorization omitted, rejection deferred (F1). |
| §1.11 | H208 | Partial: S1.25 deduplicates PR and requires approval; fail-closed gate bit beyond user approval not established until M3 (F1). |
| §1.12 | H209 | Outside task→PR exit; S3.14–17 add host checks/status and staleness. |
| §1.13 | H210 | Outside task→PR exit; host-only human merge in M3 scenario, no baton merge. |
| §1.14 | H211 | Deferred: S2.11 disposal and S3.9 audit query; do not claim M1 reports whole lifecycle. |

### 9. Format and hygiene

| Probe | Command or script output | Result |
|---|---|---|
| README format | headings: P5 Source design docs; P39 Gates before code; P58 Milestones; P76 Stories; P250 Verification strategy; P324 Risks; P340 Review log | All README 7–8 sections in order; additional gate section does not displace any. Story table has IDs, titles, acceptance and dependencies. README 10 read with H3's express draft permission. |
| Body byte cap | `LC_ALL=C awk '/^## Review log/{exit} {print}' <plan> \| wc -c` → `54340`; file `54441` bytes, 342 lines | PASS ≤60,000 bytes. |
| HTML comments | delimiter `rg`: exit 1; no stdout | PASS. |
| Review log | P341–342 header and separator only; data rows: `0` | PASS. |
| Required private-token scan | specified case-insensitive `rg`: exit 1; no stdout | PASS. |
| Required absolute-home scan | specified path `rg`: exit 1; no stdout | PASS. |
| Milestone names/counts and story IDs | mismatches `[]`; undefined references `[]`; duplicate story definitions `[]` | PASS across milestone table, story headings, story references and verification. |
| Run states | `Near state forms: []`; all 15 diagram state spellings present; `merged` stays lowercase, as in H583 | PASS. Lower-case casefold hits like `candidate`, `failed`, `running` and `review` were inspected as ordinary prose, not alternate state labels. |
| Tiers | scan hits: `trusted-local` (P22/23/69/131/200/201/266); `hardened container` and `container tier`; `microVM` and `microVM tier` | PASS on identifier spelling; the latter two labels vary only as prose, as in H356/391/820–824. No different runtime tier identifier is assigned. |

## What is strong

- P3/P82/P330 carry H3's HOLD expressly; all 105 executable stories carry direct AI-1, and DOC drafts are distinguished from permission to execute code.
- The plan makes most provisional choices visible: written vendor check before the pair, survey and primary-source verification before the second Git host, and pilot threat review/PRD question before the default. Runtime, wire and schema choices are mostly left to named LLDs.
- The DAG is structurally clean and spans a coherent task-to-PR slice through CLI/API, Postgres, runner, executor, adapter and host. Red/green verification, golden fixtures, real Postgres privilege tests, and named fault cases are concrete.
- All PRD entries have an accounting, and all unconditional P0 IDs have a named story. The plans for idempotent host effects, separate purgeable payloads, no runtime extension loading, and a clean core-only build preserve important constraints.

## Learnings / mistakes

- My first whole-plan view omitted text. I caught both truncation markers, then re-read contiguous ranges through line 342; never infer completeness from a rendered full-file command.
- A graph pass initially showed every CODE story reached some DOC. Checking the relevant document instead exposed missing paths such as S3.4→S0.6; reachability to any DOC is not specification coverage.
- The hygiene-table insertion emitted a Python invalid-escape warning for a Markdown pipe. I verified the saved text and used raw strings or literal shell input for such snippets thereafter.
- My first evidence pass found punctuation added inside several quotation marks. I moved it outside and checked exact source spans; a faithful fragment must match the source bytes, not just its meaning.

## Re-verification of DRAFT v2 (2026-11-27)

### Verdict on v2

**REVISE (HIGH confidence: exact source, graph and byte checks).** Of F1–F18, 16 are CLOSED, F2 and F9 are PARTLY, and none is NOT CLOSED; new F19 is BLOCKING. The verdict follows from BLOCKING items alone: M1 candidate upload lacks lease-expiry refusal, one CODE story misses its specifying CLI document, and release NFR-2 evidence no longer requires representative pilot repos. The revised reviewer/SHA/audit contracts are sufficient on M1's own path; its remaining fencing gap does not require renewal or a later milestone. The byte-neutral F2/F9 corrections and byte-saving F19 correction fit the cap; with the recommended status below the body is 59,952 bytes. The HOLD and maintainer-owned decisions remain intact, and neither this review nor a future plan-quality APPROVE authorizes code. **Exact recommended status line:** `> Status: DRAFT v2 — REVISE (re-verified 2026-11-27; F2/F9 PARTLY; F19 BLOCKING open). HOLD inherited from HLD-000: the seven R3 §G experiments are UNRUN; no CODE, TEST or OPS story may start until the maintainer lifts that hold. Drafted under the HLD's "plans may be drafted" clause.`

### Findings F1 to F18

V denotes the DRAFT v2 plan; H and R denote HLD-000 and PRD-000, with the same source line numbering as review 1. Historic P citations above remain v1 citations. Closure is checked against the source requirement, not the author's disposition label or my preferred wording.

| Finding | Tag | Result (CLOSED / PARTLY / NOT CLOSED) | Evidence in v2 (line number and exact quote) | Source line it now satisfies | What remains, if anything |
|---|---|---|---|---|---|
| F1 | BLOCKING | CLOSED | V83: “signed-in users holding that role with membership of the repo”; V141: “the outbox worker opens one PR only if, at that moment, a reviewer's approval is still current at the candidate head SHA”; V140: “a policy naming one stays ineligible (no check runs before M3)”; V185: “an injected evaluation error blocks it” | H207–208 (authorised reviewer and current-SHA gate), H739 (repo scoping), R120 (both evaluation-error phases). | S0.4's fixture roles plus the M1 negative tests suffice; S1.26 need not reach S3.2 or S3.11, since role administration and running nonempty pre-PR checks are not M1's claimed behavior. V69/V142 keep branch protection active; S3.12/S3.16 own the later full gate cases. |
| F2 | BLOCKING | PARTLY | V131: “only an ineligible one, the run stays Queued”; V134: “or sent after the lease expired, is refused”; V137: “an upload under a stale epoch is refused”; V113: “until S2.1 an M1 run fits one lease interval” | H200 (eligible fenced claim), H441 (fencing and credential expiry with lease), H583 (no second ambiguous writer). | Eligibility, second-claim refusal and event fencing are closed, and deferring renewal for a bounded M1 fixture is sufficient. Candidate-upload expiry is not tested: expiry and an old epoch are distinct (V134 itself tests both), and a slow or interrupted run can outlive its lease even without a second claim. S2.3 postpones credential-expiry refusal to M2. Smallest fix: V137 replace `stale epoch` with `stale epoch or expired lease` (+17 bytes); V131 reorder `a lease with S0.6's expiry and per-run credentials` to `a lease and per-run credentials with S0.6's expiry` (0 bytes) so the claim grants lease-bound credentials. Equal safe trim at V18: replace `Quoted in HLD-000's words; ` with `HLD-000's words; ` (−10 bytes) and `a bare § is an HLD-000 Design section` with `a bare § is an HLD-000 section` (−7 bytes). Net 0; body remains 59,958 bytes. |
| F3 | BLOCKING | CLOSED | V141: “The gate decision, its audit event and the PR outbox entry commit in one transaction before any host call”; V185: “the transition commits only with its audit event”; V209: “decision, state change, audit event and the answer for the runner commit in one transaction before the answer is delivered”; V209: “with policy evaluation unavailable or the audit write rejected the tool is not granted” | H157 (audit before privilege), H550 (transaction and outbox), R126–127 (denied grants and complete audit). | The M1 PR transition has its own atomic audit contract, without a later-milestone dependency. S3.20 tests failed audit at approval, PR and merge eligibility; S4.11 adds permission decisions to S3.9's query. Host I/O remains outside the transaction. |
| F4 | BLOCKING | CLOSED | V94: “resolving against §1 step 7, with no new state, the path an unconfirmed stop takes from AwaitingInput”; V209: “an unconfirmed stop shows NeedsReconciliation by the path S0.2 specifies, never a completed stop”; V209: “the run returns to Running” | H204 (unconfirmed stop), H559/H573–576 (state paths), R117 (observable unresolved cancellation). | No remaining condition. S0.2 owns resolving the diagram/detail ambiguity before code; S0.6 specifies stop confirmation. |
| F5 | BLOCKING | CLOSED | V203: “ineligible or unknown work is denied, never moved to another tier”; V212: “On a controlled fixture repo”; V250: “on a tier the recorded AI-3 result makes that repo eligible for”; V250 Blocked: “AI-1, AI-3, AI-12” | H391 (bounded envelope; hostile code refused), H886 (pilot eligibility), H138 (strict no-egress). | S4.14 need not carry AI-3 or AI-12: V195 explicitly confines pre-decision container runs to controlled fixture repos, S0.7/S0.9 specify affirmative eligibility, and S6.11 carries both pilot gates. V335 sends an invalidated tier/egress assumption to re-planning rather than substitution. |
| F6 | BLOCKING | CLOSED | V100: “records how each selected adapter meets FR-6's API-key and enterprise-billing path support”; V211: “each adapter's credential-path support is as S0.8 records” | R110 (both support paths and non-consumer execution), H542 (written vendor selection). | No remaining condition. S4.13 retains the non-consumer run checks and checks the specified support; V48 escalates infeasibility to the PRD owner before S1.17. The requirement is not changed to one arbitrary path. |
| F7 | BLOCKING | CLOSED | V243: “it freezes writes and claims, then takes that upgrade's own backup”; V245: “a rollback restores that upgrade's own backup in a test environment, and the last audit event and run state committed before the freeze are present” | H803 (freeze and no committed-write loss), H585 (rollback versus disaster restore), R151 (upgrade/restore). | No remaining condition. S6.4 keeps refusal through acceptance or rollback; S6.6 tests both a write and a claim during the freeze. S6.3 remains the separate disaster-restore case. |
| F8 | BLOCKING | CLOSED | V117/V118: “the check adds no core dependency the maintainer has not admitted on record (AI-11)”; V55: “a core dependency either needs is admitted by the maintainer, on record, before it is added” | H894 (each core dependency admitted), H258 (licence check). | No self-dependency is introduced: the maintainer admits any bootstrap dependency before it is added; the checks need not already exist to be written. |
| F9 | BLOCKING | PARTLY | V173 Deps: “S0.6, S3.2”; V270: “The fault suite, then the S2.7 and S2.9 tests”; V85: “directly or through an earlier story” | H441 (runner operation boundary), H687 (local ceilings), H204 (cancel/timeout cases). | S2.17 need not reach S2.7/S2.9: the M2 verification row explicitly runs their story tests in addition to the fault suite. The six cited DOC edges and five test prerequisites are closed, but the same missing-spec cause also affects S4.9: V207 promises “a feature the agent lacks shows unavailable before dispatch in API and CLI”, with only “S1.17” in Deps and no path to S0.5, the specifying CLI LLD (V97; H221; R109). Missed in review 1; merged here to preserve one cause, one finding. Smallest fix: add `S0.5, ` to S4.9's Deps (+6 bytes); equal safe trim V7: `Statuses are quoted from each file as read on` → `Each file's status is quoted as read on` (−6 bytes). Net 0; body stays 59,958 bytes. S5.6 promises no CLI behavior and does not need that edge. |
| F10 | BLOCKING | CLOSED | V121: “with its documented prerequisites in place, starts server, Postgres and one runner”; V121: “no separate workflow cluster runs” | H795 (Compose topology and same-image runners), R144 (external prerequisites). | No remaining condition. Image layout stays with S0.13 rather than mandating Postgres inside the product image; S6.1 retains the same-image additional runner. |
| F11 | BLOCKING | CLOSED | V74: “S6.12 reads NFR-1 PASS and holds the NFR-2 report”; V74: “every P0 story (S5.17 if FR-20 is P0) is accepted and every applicable gate closed”; V233: “P0 only if the maintainer records, before HLD-000 approval, that the partner repo survey resolves PRD option set 3 to A” | R103 (all P0 before alpha), R144 (miss is not waived), R124 (FR-20 timing), H889 (release benchmark gate). | Release now requires the measured verdict, report, applicable gates and conditional test; producing a FAIL report cannot pass M6. The changed sampling criterion in S6.12 is assessed separately as a revision-introduced cause. |
| F12 | ADVISORY | CLOSED | V13: “VISION APPROVED v2; PRD-000 APPROVED v2; R2 APPROVED v2; R3 APPROVED v4; R1 APPROVED v2”; V22: “Every run gets its own checkout and branch (FR-9). Tier is a policy attribute shown before dispatch (FR-10)”; V53: “reassessing §3.2 Option 4 and an out-of-process Wasm host under §9.1 Option 1.” | H8 (Inputs), H356 (tier statement), H884–895 (all action-item subjects, owners and blockers). | Literal source columns are now distinct from the plan-impact column; all 12 owner, subject and blocker comparisons pass. Exact fragments, not ellipses presented as source text, suffice. |
| F13 | ADVISORY | CLOSED | V278: “the stories whose acceptance lines together cover the PRD's Accept line”; V309 Verified by: “S1.20, S2.8, S3.19, S6.11”; V284: “supplemental: S6.7, core-only build; conditional: S5.17”; V301: “conditional; specification: S0.10” | R132 (distributed CLI acceptance), R107 (core-only workflow), R124 (conditional second host). | Four columns suffice: parentheses explicitly distinguish specification, supplemental and conditional evidence from full acceptance. The corrected FR-28 set covers start/cancel/diff/gate/PR. The recomputed matrix comparison below accounts for all remaining literal deltas. |
| F14 | BLOCKING | CLOSED | V162: “A checkout path containing a symlink and one containing a traversal segment are each rejected”; V162: “even when the path would stay inside the runner's checkout root” | H356 (symlinks and traversal rejected). | No remaining condition: the weaker outside-root-only policy is gone. |
| F15 | ADVISORY | CLOSED | V99: “rootless or user-namespaced”; V99: “which of the two modes it used” | H795 (runtime alternative), H137 (measured runner support). | S0.7 retains both permitted modes and records the AI-6 owner's measured mode; it does not select a concrete runtime. |
| F16 | BLOCKING | CLOSED | V95: “each carried over TLS”; V105: “where client-to-server TLS terminates”; V122: “a token sent to the documented endpoint without TLS is denied”; V250: “CLI results, read over TLS, equal API reads” | H818 (client/server TLS), H238 (HTTPS). | API/install LLDs allocate the boundary; M1 and release tests enforce it. No library or termination topology is invented. |
| F17 | BLOCKING | CLOSED | V159: “only an executor meeting S0.2's criteria for a confirmed baton-owned orphan”; V159: “an executor assigned to an active run and one baton did not create both survive”; V99: “failure ownership: provider reports, reconciler decides” | H350 (provider/reconciler ownership), H841 (reconciliation then disposal). | A list mismatch is no longer the disposal authority; both protected negative cases are observable. |
| F18 | BLOCKING | CLOSED | V106: “the compatibility artefact (the previous release's runner build) that the upgrade test runs”; V244: “The previous release's runner build, unmodified”; V244: “runs from before the upgrade stay readable” | H803 (previous runner version and old-run reads). | The actual prior build, not a protocol-number surrogate, is now required. |

### New findings

1. **F19 [BLOCKING] — The alpha report can accept an unrepresentative pilot sample.** Introduced by the revision: v1 P248 required representative pilot repos; V251 now accepts results “on the pilot repos the AI-6 owner names”. R145 explicitly requires “on representative pilot repos”. The owner must choose and measure the sample, but that does not remove the required sampling property; naming arbitrary convenient pilot repos can satisfy the new acceptance while building the wrong NFR-2 evidence artifact. S0.15 merely specifies the NFR-2 report and S5.13 calculates stages, so neither restores that testable criterion. **Smallest fix:** V251 replace `on the pilot repos the AI-6 owner names` with `on representative pilot repos` (−10 bytes). It saves rather than costs bytes, so no compensating deletion is required; AI-6's owner remains stated earlier in the same cell. With this fix alone the body is 59,948 bytes; with F2/F9's byte-neutral fixes it is still 59,948 bytes.

### Check tables (v2)

#### 1. Requirement coverage, independently recomputed

Script parsed the 44 PRD requirement entries and story title/Traces/acceptance before comparing with Delivered/Verified columns; output: **39 unconditional P0; P0 without a story `[]`; unknown requirement IDs `[]`; matrix-only story mappings `[]`.** All 44 priority/withdrawal dispositions match the source, including the literal FR-20 condition. Omitted M0 specifications are intentional under V278; the only other literal delta is NFR-1→S6.11, because that test consumes the prerequisites but does not perform the three-attempt benchmark. It is not missing NFR-1 proof. These are literal mappings, not claims that an ID mention itself proves acceptance; F2/F9/F19 qualify the safety, dependency and NFR evidence.

| Requirement | Source line; exact priority/withdrawal | Recomputed story IDs | Difference from matrix | Result |
|---|---|---|---|---|
| FR-1 | R105; P0 | S0.3, S1.7, S1.8, S1.9 | omitted M0: S0.3 | Mapped |
| FR-2 | R106; P0 | S0.3, S1.11, S1.18, S1.19, S1.20, S2.10, S2.16 | omitted M0: S0.3 | Mapped |
| FR-3 | R107; P0 | S0.17, S1.26, S4.14, S5.17, S6.7, S6.11 | omitted M0: S0.17 | Mapped |
| FR-4 | R108; P0 | S0.8, S1.17, S4.12, S4.13 | omitted M0: S0.8 | Mapped |
| FR-5 | R109; P0 | S0.8, S4.9, S4.10, S4.11 | omitted M0: S0.8 | Mapped; F9 dependency open |
| FR-6 | R110; P0 | S0.8, S1.14, S1.17, S4.12, S4.13 | omitted M0: S0.8 | Mapped |
| FR-7 | R111; P1 | — | none | No alpha story; P1 |
| FR-8 | R112; P1 | — | none | No alpha story; P1 |
| FR-9 | R113; P0 | S0.7, S1.16, S2.11, S2.15, S2.16 | omitted M0: S0.7 | Mapped |
| FR-10 | R114; P0 | S0.7, S1.10, S1.16, S4.5, S4.8 | omitted M0: S0.7 | Mapped |
| FR-11 | R115; P0 | S0.7, S4.1, S4.2, S4.5, S4.6, S4.8 | omitted M0: S0.7 | Mapped |
| FR-12 | R116; P0 | S0.6, S1.12, S1.15, S2.1, S2.17, S6.1 | omitted M0: S0.6 | Mapped; F2 open |
| FR-13 | R117; P0 | S0.6, S2.5, S2.7, S2.8, S4.11 | omitted M0: S0.6 | Mapped |
| FR-14 | R118; P0 | S0.2, S0.6, S1.15, S1.18, S1.21, S2.1, S2.2, S2.4, S2.5, S2.6, S2.12, S2.17 | omitted M0: S0.2, S0.6 | Mapped; F2 open |
| FR-15 | R119; P0 | S0.3, S1.21, S1.22, S1.23, S3.19 | omitted M0: S0.3 | Mapped |
| FR-16 | R120; P0 | S0.9, S1.24, S3.11, S3.12, S3.16, S3.17, S3.22, S4.7 | omitted M0: S0.9 | Mapped |
| FR-17 | R121; P0 | S0.9, S1.24, S3.4, S3.13, S3.22 | omitted M0: S0.9 | Mapped |
| FR-18 | R122; P0 | S0.10, S1.25, S2.13, S3.14, S3.15, S3.16, S3.22 | omitted M0: S0.10 | Mapped |
| FR-19 | R123; P0 | S0.10, S1.25, S6.11 | none | Mapped |
| FR-20 | R124; P1; becomes P0 if option set 3 resolves to A before HLD approval | S0.10, S5.16, S5.17 | none | Mapped |
| FR-21 | R125; P0 | S0.9, S1.10, S3.5, S3.6, S4.3 | omitted M0: S0.9 | Mapped |
| FR-22 | R126; P0 | S0.9, S0.11, S1.11, S1.25, S3.7, S3.12, S3.16, S3.20, S4.11 | omitted M0: S0.9, S0.11 | Mapped |
| FR-23 | R127; P0 | S0.11, S3.5, S3.8, S3.9, S4.11 | omitted M0: S0.11 | Mapped |
| FR-24 | R128; P0 | S0.9, S2.9, S4.4, S5.1 | omitted M0: S0.9 | Mapped |
| FR-25 | R129; P0 | S0.9, S5.2, S5.3, S5.4, S5.5, S5.6 | omitted M0: S0.9 | Mapped |
| FR-26 | R130; P0 | S0.4, S1.24, S3.1, S3.2, S3.4, S3.22 | omitted M0: S0.4 | Mapped |
| FR-27 | R131; P0 | S0.3, S1.6, S1.19, S1.23, S3.21, S6.9 | omitted M0: S0.3 | Mapped |
| FR-28 | R132; P0 | S0.5, S1.9, S1.20, S2.8, S3.19, S6.11 | omitted M0: S0.5 | Mapped |
| FR-29 | R133; P0 | S0.12, S5.7, S5.8, S5.9 | none | Mapped |
| FR-30 | R134; P1 | — | none | No alpha story; P1 |
| FR-31 | R135; P0 | S0.13, S1.3, S1.5, S6.1, S6.8, S6.11 | omitted M0: S0.13 | Mapped |
| FR-32 | R136; P0 | S0.16, S5.14, S5.15 | omitted M0: S0.16 | Mapped |
| FR-33 | R137; withdrawn, see Review log | — | none | No story; withdrawn |
| FR-34 | R138; P0 | S0.4, S1.6, S3.2 | omitted M0: S0.4 | Mapped |
| NFR-1 | R144; P0 | S0.13, S6.8, S6.10, S6.11, S6.12 | omitted M0: S0.13; other: S6.11 | Mapped |
| NFR-2 | R145; P0 | S0.15, S5.12, S5.13, S6.12 | omitted M0: S0.15 | Mapped; F19 sample condition open |
| NFR-3 | R146; P0 | S0.2, S0.17, S1.8, S2.17, S3.18 | omitted M0: S0.2, S0.17 | Mapped |
| NFR-4 | R147; P0 | S0.7, S4.1, S4.2, S4.5, S4.8 | omitted M0: S0.7 | Mapped |
| NFR-5 | R148; P0 | S0.4, S0.11, S1.6, S1.13, S1.14, S2.3, S3.4, S3.10, S5.15 | omitted M0: S0.4, S0.11 | Mapped |
| NFR-6 | R149; P0 | S0.4, S1.19, S3.3 | omitted M0: S0.4 | Mapped |
| NFR-7 | R150; P0 | S4.14, S6.11 | none | Mapped |
| NFR-8 | R151; P0 | S0.14, S1.4, S2.14, S6.2, S6.3, S6.4, S6.5, S6.6 | omitted M0: S0.14 | Mapped |
| NFR-9 | R152; P0 | S0.1, S1.1, S1.2, S6.7, S6.11 | omitted M0: S0.1 | Mapped |
| NFR-10 | R153; P0 | S0.11, S1.11, S3.8, S3.20, S5.10, S5.11 | omitted M0: S0.11 | Mapped |

#### 2–3. Changed decisions and reserved implementation choices

Script: literal substring search of every double-quoted span in V13 and all D1–D18 rows, restricted to the HLD body; row-by-row source comparison for the action items. Results are in quotation order. This also checks shortened D1/D4/D15/D16 spans, not only the seven singled out in the handoff.

| Item | V line | Script output (H lines) | Result |
|---|---|---|---|
| Inputs | 13 | `[8]` | Exact contiguous source span. |
| D1 / D2 | 20 / 21 | `[217]` / `[42]` | Exact; AI-2 reversibility and Compose remain. |
| D3 | 22 | `[356, 356]` | Both spans exact; removed parenthetical is not represented by a false ellipsis. |
| D4 | 23 | `[389, 391, 391]` | Exact; bounded eligibility/default condition retained. |
| D5 | 24 | `[441, 441]` | Exact; eligibility/fencing acceptance is assessed in F2. |
| D6 | 25 | `[491, 542]` | Exact; vendor selection still owns the first pair. |
| D7 | 26 | `[548, 550, 550]` | Exact; at-least-once deduplication retained. |
| D8 / D9 | 27 / 28 | `[583]` / `[585]` | Exact; canonical states and separate retention retained. |
| D10 | 29 | `[649, 649]` | Exact; both gate phases and error/ineligible condition retained. |
| D11 / D12 | 30 / 31 | `[662, 662]` / `[677, 677]` | Exact. |
| D13 | 32 | `[687]` | Exact; local/disconnected ceilings retained. |
| D14 / D15 | 33 / 34 | `[710, 712]` / `[720, 720]` | Exact; survey condition and server-only writable identity retained. |
| D16 / D17 | 35 / 36 | `[739, 737, 737]` / `[764, 764, 774, 774]` | Exact; tenancy, contracts and edition boundary retained. |
| D18 | 37 | `[803, 803, 803]` | Exact; Compose remains in D2/S0.13/S1.5/S6.1, not lost by shortening D18. |
| Gate subjects/blockers | 45–56 | 12/12 subjects and 12/12 blockers exact at H884–895 | Passed; owners separately 12/12 exact. |

Changed-acceptance script: 42 cells; added/removed/renumbered story IDs `[]`; technology scan `[]`; literal-path scan `[]`. Manual reading of all 42 cells finds **no concrete language, library, schema/table/column name, wire format, configuration key or file path**, and no newly selected choice reserved by the HLD. The schema/wire/protocol/configuration terms occur as topics for S0.2/S0.3/S0.6/S0.7/S0.8/S0.12/S0.13/S0.14, not decided identifiers. Logical state, actor/SHA fields, Postgres/Compose, TLS, fencing, rootless/user-namespaced and secret references are source-fixed mechanisms (H550/H441/H795/H818/H737), not invented technologies. S2.15's checkout-root rule is H356's policy, not a path. S4.11's reconciliation path is explicitly assigned to S0.2. The missing approved CLI-spec predecessor is F9, not an invented concrete choice in the revised cell.

#### 4. HOLD and gates

Script parsed H884–895's owner, subject and Blocks into separate fields and compared them byte-for-byte with V45–56: **12/12 owner, 12/12 subject, 12/12 blocker exact**. Direct AI-1 on every executable story: **105/105**; exceptions `[]`. V3, V82 and V333 keep the maintainer-controlled HOLD and also bind TEST/OPS; documentation may be drafted, not used to authorize execution. A re-verification APPROVE would not close an action item or authorize code.

| Gate | V / H lines | Source-field comparison | Direct story gates from script | Operational result |
|---|---|---|---|---|
| AI-1 | 45 / 884 | owner/subject/blocker exact | every CODE/TEST/OPS | HOLD retained, no executable exception. |
| AI-2 | 46 / 885 | all exact | S0.1, S0.6, S0.7 | Document approvals wait; runner work inherits; Risk 2 re-plans a changed baseline. |
| AI-3 | 47 / 886 | all exact | S4.6, S6.11, S6.12 | Default/pilot blocked; S4.14 is controlled-fixture-only. |
| AI-4 | 48 / 887 | all exact | S0.8, S1.17, S4.12 | Vendor selection and FR-6 owner escalation occur before first adapter code. |
| AI-5 | 49 / 888 | all exact | S0.10, S5.16, S5.17 | Survey owns host selection and exact pre-HLD-approval priority condition. |
| AI-6 | 50 / 889 | all exact | S6.12 | S0.7 itself records runner mode evidence; M6 requires benchmark PASS/report. No producer self-gate. |
| AI-7 | 51 / 890 | all exact | S5.11 | S0.9 specifies/closes default; purge waits. |
| AI-8 | 52 / 891 | all exact | S5.16, S5.17 | Mapping is checked in S0.10; conditional consumers wait. |
| AI-9 | 53 / 892 | all exact | none | Deferred runtime/host reassessments remain outside P0. |
| AI-10 | 54 / 893 | all exact | none | Optional backend not built. |
| AI-11 | 55 / 894 | all exact | S1.3; global every dependency addition | S1.1/S1.2 explicitly require recorded admission of any dependency they add. |
| AI-12 | 56 / 895 | all exact | S4.6, S6.11 | Default and pilot carry both owner decisions, not only the security review. |

#### 5. Dependency graph and specifying-document closure

Script parses Deps/Blocked, checks undefined references and DFS cycles, then computes transitive predecessors. Relevant-spec mapping covers all 86 CODE rows, assigning API/identity/CLI, state/audit, runner/provider/adapter, policy/host, install/upgrade or observability/heartbeat documents by behavior. Direct DOC edges are not required when the relevant DOC is reached transitively; unrelated stack reachability is insufficient.

| Probe | Script output | Result |
|---|---|---|
| Total | 125: DOC 20, CODE 86, TEST 14, OPS 5 | IDs unchanged. |
| M0 | DOC 17 = 17 | Design only. |
| M1 | CODE 22, TEST 1, OPS 3 = 26 | First executable milestone. |
| M2 | CODE 15, TEST 2 = 17 | Count matches. |
| M3 | CODE 19, TEST 3 = 22 | Count matches. |
| M4 | CODE 11, TEST 3 = 14 | Count matches. |
| M5 | CODE 14, TEST 3 = 17 | Two stories conditional. |
| M6 | CODE 5, TEST 2, OPS 2, DOC 3 = 12 | Count matches. |
| Structural | unknown story/gate `[]`; later milestone `[]`; later same-milestone row `[]`; cycles `[]` | Clean structural DAG. |
| DOC coverage | CODE with no DOC `[]`; direct M0 DOC **23/86** | Generic coverage passes, not the relevant-spec test below. |
| Relevant-spec coverage | map count 86; unmapped CODE `[]`; **85/86** complete; missing `[(S4.9, S0.5)]` | F9 remains PARTLY. A provisional overbroad S5.6→S0.5 assignment was rejected: it names no CLI behavior. |
| Original six pairs | S2.4→S0.7, S2.7→S0.8, S2.9→S0.7, S3.4→S0.6, S4.5→S0.5, S5.4→S0.5: all `True` | All originally listed subject gaps closed. |
| Explicit acceptance references | references absent in closure `[]` | No dangling specifying-ID reference. |
| S1.26 | S1.17/S1.24/S0.17/S0.4 `True`; S3.2/S3.11 `False`; later-milestone predecessors `[]` | Minimum fixture roles/current-SHA/audit are M1-owned; no later dependency is needed for them. Expired-upload refusal remains F2, not a dependency on M2 renewal. |
| S2.17 | S2.7/S2.9 both `False` | Sufficient alternative: V270 explicitly runs both story-test sets at M2 exit; the suite does not claim those cases. |
| S4.13 | S4.12/S1.23 both `True` | Both adapters to Candidate with API-read diff now have prerequisites. |
| S4.14 | S4.8/S3.17 `True`; S3.18 `False` | Isolation/staleness reached; restart-gate case is not this suite's acceptance. No unrelated edge needed. |
| S6.11 | S2.8/S3.19 both `True` | Full CLI comparison prerequisites reached. |
| Effective S4.14 gates | AI-1, AI-2, AI-4, AI-5, AI-11 | Sufficient for controlled fixture; not pilot eligibility evidence. |
| Effective S6.11 gates | AI-1, AI-2, AI-3, AI-4, AI-5, AI-11, AI-12 | Both pilot decisions inherited/direct. |
| Effective S6.12 gates | AI-1, AI-2, AI-3, AI-4, AI-5, AI-6, AI-11, AI-12 | Release checks all applicable gates and every P0 story at V74. |

#### Prose/acceptance preservation and extra changes

| Probe | Script output / evidence | Result |
|---|---|---|
| All acceptance cells that got shorter | Only S2.2: v1 P148 **123** bytes → V149 **114** bytes (−9). V149: “Terminal status carrying an older epoch is rejected, as events (S1.18) and commits (S1.21) already are; tests pass”. | Read in full. Earlier event/commit cases moved to S1.18/S1.21, second-claim refusal to S1.15; Deps now reaches S1.21. No v1 assertion is lost in this shortening. Post-expiry upload testing is the still-open F2 condition, not a removed v1 test. |
| Longer cells and prose trims | All 41 other changed acceptance cells grew; S6.12's growth replaced the representative-sample requirement. Only changed story gate: S6.11 `AI-1` → `AI-1, AI-3, AI-12`; no gate removed. | Cannot confirm the author's blanket preservation claim: F19 loses one NFR condition despite net growth. Sources' removed use column is redundant with stories; estimates retain LOW confidence, rate/rounding, conditional omission and HOLD/clock; review's removed LLD-stop sentence survives in strengthened Risk 7 (V339). Other condensed risk/scope prose removes no acceptance or gate. |
| Extra S0.12 | V104: “compiled-in P0 providers with out-of-process extensions only on runners or as separate services”; H764. | Faithful boundary clarification, no scope creep and no optional host implementation added. |
| Extra S2.14 | V161: “a user S0.4 authorises reads them back”; S1.18 already reaches S0.4. | Makes authority explicit without deciding a role; no scope creep or invented decision. |
| Extra Verification M5 | V273 adds running S5.1/S5.4 story tests, already in its Proved by cell. | Correct evidence wiring, not new behavior. |
| Extra fault-level citations | V260 cites S2.13, S2.17, S3.7, S3.15, S3.20. | Host outage, death/duplicate/restart, policy failure, webhook loss and audit failure now point to their actual owning stories; no scope expansion. |
| Plan-defined quoted terms | V41 defines “Closed”; V278 defines “Delivered by” and “Verified by”. | Acceptable: clearly definitions of this plan's controls/table labels, not assertions of literal HLD wording. |

#### 6–8. Changed-story quality, scope and milestone exits

| Probe | Output / reviewed population | Result |
|---|---|---|
| Changed acceptance tally | M0 10; M1 14; M2 4; M3 4; M4 4; M5 1; M6 5 = **42/42** | All changed cells read; no cell selects a concrete implementation reserved for its LLD. |
| Full population screen | M0 17; M1 26; M2 17; M3 22; M4 14; M5 17; M6 12 = **125/125** | All rows read in full; the v1 one-behavior/document/suite assessment remains unless changed. Missing `tests pass` on CODE/TEST `[]`; judgment-word scan `[]`. |
| Scope/conditional gates | V62, V204, V233–234, V48–49, V335 | No added story or P1 scope; first pair is vendor-gated, default is AI-3/AI-12 conditional, second host keeps R124's timing and AI-5/AI-8. Boundary fixes do not build the deferred runtime/host. |
| M1 §1 slice | Steps 1–5, 8, 10–11; step 9 only vacuous for an explicit no-required-check fixture policy | No later-milestone prerequisites. Full permission/cancel, nonempty checks, host status/merge and terminal cleanup remain later breadth. Roles/current-SHA/audit now present; lease-expired candidate upload is the remaining F2 invariant. |
| M2/M4 evidence | V270 explicitly runs cancel/timeout tests; S4.13 reaches diff-read; S4.14 reaches isolation and staleness | Evidence alternatives close the originally cited test gaps; controlled fixtures are not claimed as pilot evidence. |
| M5/M6 exits | V73 requires conditional S5.17; V74 requires NFR-1 PASS, NFR-2 report, every P0 and every applicable gate | Exit outcomes are observable and cannot substitute a report artifact for PASS. F19 still permits the wrong NFR-2 sample property. |

#### 9. Format and hygiene

| Probe | Command/script output | Result |
|---|---|---|
| README sections | V5 Source design docs; V58 Milestones; V76 Stories; V253 Verification strategy; V327 Risks; V343 Review log | Required sections in order; V39 Gates is supplemental. |
| Body limit | Required `LC_ALL=C awk` byte count: **59,958**; entire plan **67,078** bytes, **372** lines | PASS: 42 bytes headroom; Review log excluded as specified. |
| HTML comments | Delimiter scan exit **1**, stdout empty; Python matching lines `[]` | PASS. |
| Private-token scan | Required case-insensitive four-token `rg` scan exit **1**, stdout empty | PASS; exact supplied scan was used, not a weaker pattern. |
| Absolute-home scan | Required `/Users/` `rg` scan exit **1**, stdout empty | PASS. |
| Story and milestone names | duplicate definitions `[]`; undefined references `[]`; milestone name mismatches `[]`; count mismatches `[]` | PASS, including retained S2.2 ID and changed title. |
| States/tiers | near-state spellings `[]`; all 15 exact canonical names present; code-span tier token set `[trusted-local]`; alternate identifier spellings `[]` | PASS. Hardened container and reserved microVM are source-consistent prose labels, not invented identifiers. |
| Review log | 18 distinct finding dispositions, all `changed`, no rebuttal | Appropriate for v2; the v1 empty-log criterion does not apply to a revised plan. Dispositions are not closure proof. |
| Minimal-fix fit | In-memory exact-row simulation of F2/F9/F19 corrections plus safe trims: **59,948** body bytes, **52** bytes headroom | No plan write performed. F2 +17/−17; F9 +6/−6; F19 −10. No acceptance, gate or condition trimmed. |
| Recommended status fit | Exact verdict status replacement adds **4** bytes; all fixes and status yield **59,952** body bytes, **48** bytes headroom | Fits without another trim; no plan write performed. |

### What is strong in v2

- Minimum role/repo authority, same-SHA PR gating, atomic PR audit and TLS are now on M1's own path. Breadth is deferred without requiring M1 to implement all administration or CI execution.
- The bounded container envelope is explicit: affirmative eligibility, no fallback, hostile code refusal, controlled fixtures before the pilot decision, and both owner gates on the pilot.
- Upgrade freeze precedes the upgrade-specific backup; the drill tests rollback with the last pre-freeze audit/state. Previous-runner compatibility uses an unmodified prior release build.
- Gate source text is literal and separate from plan impacts; the four-column matrix labels specification/supplemental/conditional evidence clearly; measured release outcomes cannot be waived by merely producing an artifact.
- The revised graph is structurally clean, and all original six missing DOC links and five test prerequisite links were corrected. Remaining fixes are small and fit inside the byte cap without deleting safety or acceptance conditions.

### Learnings / mistakes

- I initially followed a compacted example using a quoted home-variable path; `view` treated it literally and failed. I used the resolved input path thereafter: built-in path parsing is not shell expansion.
- Conversation search snippets remained capped even when requesting a larger snippet. I recovered only this thread's two original task messages read-only from the local store; never infer omitted instructions or substitute the latest relay for the original source requirements.
- I drafted the missed CLI dependency as a new finding before the duplicate-free pass. It shares F9's cause, so I merged it into F9 and renumbered the sole revision-introduced finding F19; one cause, one finding.
- A byte calculation for a prose trim was initially off by three bytes. The UTF-8 calculation caught it; the recorded fixes now use measured equal trims and an exact targeted-row simulation, not character guesses.
- Audit scripts had a failed comprehension, a backtick regex crossing lines, and a final gate parser treating numbered HLD action items as table rows. I corrected each and reran the failed probe before relying on it; parser errors are not audit results.
- The first broad relevant-spec map over-assigned the CLI LLD to S5.6, which promises no CLI behavior. I rejected that pair on the acceptance text and kept only S4.9's actual CLI dependency; dependency audits must follow promised behavior, not proximity.
- Several read-only calls automatically promoted despite the no-background-task instruction, including one requested with a longer foreground timeout. I did not launch, poll or delegate background work; incidental tool promotion is not permission to scaffold a later iteration.
- The author's longer S6.12 cell removed a required sampling property. Net growth and checking only shorter cells are not preservation proofs; compare changed clauses against their source requirement too.
- A post-compaction recovery first queried the legacy conversation database and the wrong message type; empty results exposed both errors. I recovered the exact task text read-only from this thread in the current store before writing the verdict; absence from one store is not absence of instructions.
