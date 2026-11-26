# HLD-000 Platform overview HLD

> Status: DRAFT v1 (awaiting review)
> HLD = architecture decisions, system boundaries, options analysis, data flows.
> No class names, config keys, or test file names (those belong in the LLD).
> Every decision cites its source (research doc §, measurement, or "verified-by: <how>").

> Inputs: VISION APPROVED v2; PRD-000 APPROVED v2; R2 APPROVED v2; R3 APPROVED v4; R1 APPROVED v2. Scope: the first three-month (12-week) self-hosted alpha.
> Citation keys: "R2 S46" is source S46 in R2's source list; R1 and R2 number sources separately. R2 section keys are expanded under References.

## Background

Teams run several coding agents in parallel, each on its own copy of a repo, then review and merge; each vendor ships a surface for its own agent (VISION §The one-paragraph thesis; R1 §3a, §4). Platform and DevEx teams (payers) want that work inside their boundary, across vendors, with per-repo policy, budgets and audit; engineering teams (users) want shared, gated fleets instead of worktree scripts (VISION §Who it is for; PRD §Problem and users).

Prior attempts narrow the gap: Conductor is proprietary with no self-hosted control plane; GitHub Agent HQ is cross-vendor inside one host's cloud; OSS conductors ship servers and remote clients; Coder sells self-hosted governance in paid tiers; Stripe, Ramp and Spotify built their own harnesses (R1 §1, §4, Analysis; R2 §5; R3 §C). The target gap is a customer-hosted, cross-vendor control plane with declarative gates and audit in its open edition (R1 Analysis; MED).

Phase 0 changed three premises. Checkout separation is not isolation: worktrees share host network and credentials, containers share the kernel, microVMs are stronger but prove no zero exfiltration (R2 §1). ACP, MCP and A2A are complementary, none a job specification, and ACP tool permission is not change approval (R2 §2). Vendor policy can change in one announcement, so adapters cannot rely on consumer subscription login (R2 §2; R3 §C).

The approved stance is option B (team server, self-hosted runners, worktree and container executors, Postgres) inside option C's provider-neutral boundaries, shipped as an OSS self-hosted team server with a thin client (VISION §Strategy; R2 §Options; R3 §D). This HLD fixes platform boundaries, the PRD's handoff decisions (PRD §Handoffs to HLD-000) and the cross-cutting model for the 12-week alpha; component HLDs and LLDs follow (PHASE-1 §Open after Phase 1).

## Glossary

| Abbreviation | Full form | Description |
|---|---|---|
| A2A | Agent2Agent | Protocol between independently deployed, opaque agents (R2 §2). |
| ACP | Agent Client Protocol | Client-to-coding-agent JSON-RPC protocol; tool permission, not change approval (R2 §2). |
| Adapter | Agent adapter | Runner-side driver mapping one agent onto baton's agent contract, with declared capabilities. |
| Candidate SHA | Candidate commit SHA | Commit a gate decision binds to; any change stales it (VISION §Design principles). |
| Executor | Execution backend | Runner-side checkout and process boundary for one run (§3). |
| Fencing epoch | Lease generation number | Monotonic lease number; writes with an older one are rejected (R2 §Analysis/Durability). |
| Gate | Change gate | SHA-bound, fail-closed condition on a candidate (§6.1). |
| HLD / LLD | High- / low-level design | Boundaries here; schemas and configuration in the LLD (BRIEF rule 8). |
| KVM | Kernel-based Virtual Machine | Linux virtualization Firecracker needs (R2 S04). |
| Lease | Time-bounded claim | A runner's exclusive, renewable right to execute one run. |
| MCP | Model Context Protocol | Host-to-tools protocol; no supervision semantics (R2 §2). |
| OIDC | OpenID Connect | Core sign-in protocol for P0 users (PRD FR-34). |
| Outbox | Transactional outbox | Side-effect intents committed with the state change, executed later at least once. |
| P0 / P1 | Release priority | P0 must pass before the 12-week alpha; P1 is post-alpha or partner-dependent (PRD §Functional requirements). |
| Provider | External integration | Implementation behind a baton contract. |
| Runner | Customer-operated runner | Product binary in runner mode; claims runs outbound, hosts executors and adapters. |
| SCIM / SSO | System for Cross-domain Identity Management / single sign-on | Enterprise identity provisioning and federation (R3 §B). |
| trusted-local | Unsandboxed worktree tier | Worktree runs sharing the runner host's user, network and credentials; never called isolation (PRD FR-10). |

## P0 Scope

The first release is the PRD's 12-week self-hosted alpha for one organisation (PRD §P0 scope). It is the slice R2 judges credible for a small team: two agents, two executors, at most two Git hosts, one runner OS and architecture, Postgres, no multitenancy or compliance claim (R2 §Scope, LOW).

P0 includes:

- One product binary with a server mode and a runner mode (PRD §P0 scope).
- One Compose file that deploys the server, Postgres and one runner (PRD FR-31).
- Additional runners on other hosts using the same binary (PRD FR-12).
- An authenticated headless team API and a thin CLI (PRD FR-27, FR-28; option set 1 A).
- Two production-path agent adapters, at least one driven through its local CLI (PRD FR-4).
- Two executors: `trusted-local` worktree and hardened container (PRD FR-10, FR-11).
- One Git host with a tested PR path; a second is P1 (PRD FR-19, FR-20).
- Org and repo policy, SHA-bound fail-closed gates, budgets and append-only audit in the Apache-2.0 core (PRD FR-16 to FR-25).
- Single-org roles with OIDC sign-in (PRD FR-26, FR-34).
- Versioned extension boundaries (PRD FR-29).
- Opt-in heartbeat, off by default (PRD FR-32).
- Backup, restore and a documented schema-compatible upgrade (PRD NFR-8).

Why limited to this: the open core must run task -> run -> diff -> gate -> PR without a paid module (VISION §Design principles; R3 §B); more providers, tenancy, HA or per-agent parity need buyer evidence and a threat review first (R2 §Scope). The value is the gate, policy and audit layer, not a new UI (VISION §What it is NOT).

**Pre-HLD experiment record (PRD §Handoffs to HLD-000 item 9).** As of 2026-11-26 none of the seven R3 §G experiments has a recorded result (VISION §How we will know; PRD §Assumptions); dependent assumptions are marked UNRUN. VISION requires all seven before any HLD is approved, so review can proceed but approval waits (Action Item 1).

| # | R3 §G experiment | Status | Dependent assumptions |
|---|---|---|---|
| 1 | Payer kill test | UNRUN | 1, 2, 4, 9, 10 |
| 2 | Leaver/stayer interviews | UNRUN | 5, 6 |
| 3 | Segmented landing page | UNRUN | 5, 11, 14 |
| 4 | Audience 3b conversations | UNRUN | 11 |
| 5 | Fake-door pricing | UNRUN | 14 |
| 6 | Issue-tracker mining | UNRUN | 6 |
| 7 | Written vendor check | UNRUN | 3 |

## Requirements

PRD-000 requirements this HLD satisfies in P0, with the carrying Design section in brackets; each traces through PRD-000 to VISION and R1 to R3.

**Functional**

1. Team workflow and surfaces: FR-1, FR-2, FR-3, FR-27, FR-28 [§1, §2].
2. Agents and capabilities: FR-4, FR-5, FR-6 [§4]; the pair itself depends on Assumption 3.
3. Checkout, execution tiers and runners: FR-9, FR-10, FR-11, FR-12 [§3].
4. Run control and recovery: FR-13, FR-14 [§5].
5. Candidate, gates and PR path: FR-15, FR-16, FR-17, FR-18, FR-19 [§6, §7].
6. Policy, audit and budgets: FR-21, FR-22, FR-23, FR-24, FR-25 [§6].
7. Roles and identity: FR-26, FR-34 [§8].
8. Extension boundaries: FR-29 [§9].
9. Install and telemetry: FR-31, FR-32 [§10, §11].

**Non-functional**

10. Install time NFR-1 and latency evidence NFR-2: per-stage timings are recorded; both pass only when measured (Assumption 10) [§10, §11].
11. Recoverable availability NFR-3 [§5, §11].
12. Execution isolation NFR-4 [§3, §11].
13. Authorisation and confidentiality NFR-5 [§3, §7, §8, §11].
14. First-release tenancy NFR-6 [§8].
15. Bounded operating cost NFR-7 [§3, §11].
16. Upgradeability NFR-8 [§10].
17. Compatibility and licensing NFR-9 [§5, §9].
18. Audit integrity NFR-10, including the retention decision the PRD assigns here [§5, §6].

**Derived (this HLD)**

19. A shared durable queue with claims, leases and reconciliation; derived from the team-server requirement, not a VISION invariant (R2 §Options).
20. Pre-PR checks run on the candidate SHA in a fresh executor, not in the agent's live session [§6]; derived from FR-16 and NFR-4.
21. Git-host write credentials stay on the server; runners hold read-only, per-run credentials [§7]; derived from FR-17 and NFR-5.

P1 requirements FR-7, FR-8, FR-20 and FR-30 get a seam, not a build (Out of Scope 6 to 9); FR-33 is withdrawn in the PRD. No P0 requirement is left unsatisfied by design.

## Out of Scope

1. Hosted edition: hosted PRD and HLD, after paying self-hosted customers (PRD Out of scope 1; R3 §B, §D).
2. Embeddable B2B2B or customer-tenant API: embedder PRD and tenancy HLD, only after two 3b letters of intent (PRD Out of scope 2; R3 §G item 4).
3. Secure multi-org, multi-customer tenancy: tenancy HLD; P0 keeps only the seam in §8 (PRD Out of scope 3).
4. Enterprise module contents (listed in §9): Enterprise PRD and HLD, contingent on Assumption 1 (PRD Out of scope 4; R3 §B).
5. MicroVM tier and external sandbox providers: execution/sandbox-adapter HLD; seam in §3; an optional provider spike is no shipped claim (PRD Out of scope 5; R2 §Scope).
6. FR-7, ACP ingestion (P1): agent-adapter HLD; the transport seam is in §4 (PRD FR-7).
7. FR-8, policy-scoped MCP tools (P1): agent-adapter HLD with the policy HLD; seam in §4 (PRD FR-8).
8. FR-20, second Git host (P1, P0 only if PRD option set 3 resolves to A before approval): Git-host component LLD; seam in §7 (PRD FR-20).
9. FR-30, operator branding (P1, Enterprise): Enterprise HLD; the presentation extension point is in §9 (PRD FR-30).
10. Public plugin marketplace and untrusted server-side plugins: extension-point LLD and security review (PRD Out of scope 6).
11. Desktop, web, mobile and IDE clients: later client requirements, if pilots select PRD option set 1 B or C (PRD Out of scope 7).
12. Merge automation: a later policy-and-gates HLD (PRD Out of scope 9).
13. Guaranteed dollar caps for opaque agents, compliance certifications and zero-exfiltration claims (PRD Out of scope 8).
14. Hostile-code workloads such as arbitrary external contributions: execution/sandbox-adapter HLD with a microVM tier; ineligible in P0 (§3.2; PRD Goal 6).
15. HA, zero-downtime upgrades and air-gap packaging: Enterprise HLD (PRD §P0 scope; NFR-8; R3 §B).
16. Tamper-evident audit chains, signing and export: policy-and-gates HLD and Enterprise HLD (R2 §Analysis/Execution; PRD §Handoffs item 6).
17. Import of other conductors' settings: later onboarding requirements (R1 §5).
18. More than one runner OS and architecture: later execution HLD (PRD §P0 scope).

## Assumptions

Each, if false, changes the section named. UNRUN marks dependence on an R3 §G experiment with no result (P0 Scope record).

1. **A payer exists for the enterprise split [UNRUN R3 §G item 1].** If the kill test fails, §9's Enterprise module is not built; the core stands (PRD Assumption 1).
2. **Pilot lower-trust work is cooperative or controlled, not hostile code [UNRUN R3 §G item 1; pilot threat review not run].** If pilots need hostile-code execution, the microVM tier moves into the alpha (PRD Assumption 5; R2 §Open questions).
3. **The first agent pair can run on permitted API-key or enterprise billing [UNRUN R3 §G item 7].** If a vendor rules an integration out, §4 swaps that adapter for an open harness (PRD Assumption 4, option set 2 B).
4. **One Git host, GitHub, is enough for the first partners, with a second as P1 [UNRUN R3 §G item 1; partner repo survey not run].** If a partner needs another host first, §7 swaps it; the contract stands (PRD Assumption 6).
5. **Teams use more than one agent [UNRUN R3 §G items 2 and 3].** If not, §4's contract stays but deeper multi-agent work is deferred (PRD Assumption 3).
6. **Shared gates and audit meet a need beyond existing OSS servers [UNRUN R3 §G items 2 and 6].** If not, §6's investment is revisited (PRD Assumption 2).
7. **Postgres-only orchestration is enough for agent run lifetimes in the alpha.** If cross-service timers, compensation and replay become central, §5 reassesses an embedded durable-workflow library or Temporal (R2 §Analysis/Durability, MED).
8. **The supported runner OS can run a rootless or user-namespaced container runtime.** If not, §3's container hardening and §10's install change; verified-by: runner LLD spike (R2 S03).
9. **Pilots accept model-provider egress from executors [UNRUN R3 §G item 1; threat review not run].** If strict no-egress is required, proprietary vendor CLIs are ineligible and §3, §4 need an open harness with a local model (PRD Assumption 5; R3 §F Q5).
10. **The NFR-1 prerequisites (host, container runtime, OIDC test client, credentials) are available for a one-hour first run [UNRUN R3 §G item 1; installation benchmark not run].** If setup dominates, §10's install target is redefined (PRD Assumption 7).
11. **The embedder is not the launch buyer [UNRUN R3 §G items 3 and 4].** If two 3b letters of intent arrive, a tenancy HLD activates §8's seam; P0 does not widen (PRD Assumption 9).
12. **The core stays Apache-2.0 and customers may serve their own customers.** If either invariant changes, §9 and the licence eliminations in §5 reopen (VISION §Strategy; R3 §B, Handoffs).
13. **Agent headless event formats stay stable enough for golden fixtures during the alpha.** If not, §4's adapters need a faster compatibility cadence (PRD option set 2, ADVISORY con).
14. **Opt-in heartbeat yields enough evidence to assess adoption [UNRUN R3 §G items 3 and 5].** If opt-in is sparse, §11 reports the rate without generalising (PRD Assumption 8).

## Design

Invariants used as BLOCKING criteria. Each is stated once and carries the same tag in every option it touches; a con shared by every option in a set is ADVISORY (LEARNINGS L3).

- **I1 Customer-hosted team control plane.** The shared server runs inside the customer's boundary (VISION §Strategy invariants; PRD §P0 scope).
- **I2 Licence boundary.** The core is Apache-2.0; it neither bundles nor hard-depends on a BSL, SSPL or other non-OSI server, and embeds no AGPL source (R3 §B; PRD NFR-9; R2 §Analysis/Self-host).
- **I3 Open core runs the defining workflow.** task -> run -> diff -> gate -> PR completes with the core's own worktree and container executors and two adapters (VISION §Design principles; PRD FR-3, FR-4, FR-11).
- **I4 Execution cannot approve or merge.** No runner or agent identity can approve, merge or write gate decisions (PRD FR-17, FR-26, NFR-5).
- **I5 Gate sequencing.** Gates bind to the candidate SHA and fail closed; a gate cannot depend on an event it precedes (PRD FR-16; LEARNINGS L36).
- **I6 Host authority.** The PR is created after the human gate and code-host branch protection is never bypassed (PRD FR-18, Out of scope 9).
- **I7 Single Compose, no workflow cluster.** Server, Postgres and runner install from one Compose file without a separate workflow-engine cluster (PRD FR-31, NFR-9).
- **I8 No untrusted code in the server.** Extension boundaries are versioned and an extension cannot gain server privileges by registering itself (PRD FR-29).
- **I9 Audit before privilege.** A privilege transition commits only with its audit event (PRD FR-22, NFR-10).
- **I10 Hard ceilings.** Time and concurrency ceilings hold at run and repo/org scope (PRD FR-24).
- **I11 No paid sandbox required.** A clean install completes the workflow without a paid hosted sandbox (PRD NFR-7).
- **I12 Lower-trust confinement.** A lower-trust run gets no host checkout, host agent credential or container daemon socket (PRD FR-11, NFR-4; VISION §Design principles).

### 1. End-to-end workflow

The defining workflow, task -> run -> diff -> gate -> PR, as it crosses the boundaries decided in §2 to §10 (VISION §Design principles; R2 §Analysis/Execution pipeline).

```mermaid
sequenceDiagram
    actor M as Member (CLI or API)
    participant S as Team server + Postgres
    participant R as Runner
    participant X as Executor + agent
    actor V as Reviewer
    participant G as Git host
    M->>S: create task, start run
    S->>S: evaluate policy, reserve budget, audit, enqueue
    R->>S: claim lease (outbound)
    S-->>R: run spec, fencing epoch, per-run credentials
    R->>X: prepare checkout and tier, start adapter
    X-->>R: agent events
    R->>S: ordered events, permission requests
    S-->>R: permission decisions or cancel
    X-->>R: agent finishes
    R->>S: candidate commits and diff
    S->>G: push candidate branch, record candidate SHA
    R->>X: pre-PR checks at candidate SHA (fresh executor)
    R->>S: check results bound to SHA
    V->>S: approve candidate SHA
    S->>S: gate passes, audit
    S->>G: open PR
    G-->>S: PR-triggered check status (webhook or poll)
    S->>S: merge eligibility at same SHA
    V->>G: merge under branch protection
    R->>X: dispose executor
```

| # | Description |
|---|---|
| 1 | A member creates a task on a configured repo via CLI or API and starts a run with an agent and executor (FR-1, FR-2). |
| 2 | The server evaluates policy, reserves budget and writes run, audit event and queue entry in one transaction; a denied run is never queued (FR-21, FR-22, FR-25; §5, §6). |
| 3 | An eligible runner claims the run outbound under a fenced lease and receives per-run credentials: read-only repo access and the agent's API key (FR-12; §3.3, §7.2). |
| 4 | The runner creates a fresh checkout and branch in the run's tier: worktree, or inside a hardened container with no host mount (FR-9 to FR-11; §3). |
| 5 | The adapter starts the agent; events stream to the server and are stored in order (FR-2, FR-5; §4, §5). |
| 6 | Tool-permission requests are decided by policy or a member; they never approve a change (FR-5; R2 §2; §4). |
| 7 | Cancel and hard time limits stop new tool work; the run ends cancelled, timed out or needs-reconciliation (FR-13, FR-24; §5). |
| 8 | The runner commits outside the agent's control and transfers commits and diff; the server pushes the candidate branch with its own identity and records the SHA (FR-15; §7.2). |
| 9 | Pre-PR checks run at the candidate SHA in a fresh executor of the same tier; results bind to that SHA (FR-16; §6.1). |
| 10 | A distinct authorised reviewer sees files, diff, SHA and checks and approves or rejects that SHA; the runner cannot (FR-15, FR-17). |
| 11 | With approval and checks current at that SHA and audit committed, the server opens exactly one PR (FR-16, FR-18, FR-22). |
| 12 | The host runs PR-triggered checks; the server reads them by webhook and poll and computes merge eligibility at that SHA; a new commit stales every gate (FR-16, FR-18; §7). |
| 13 | A human merges on the host under branch protection; baton never merges (FR-18; PRD Out of scope 9). |
| 14 | The runner disposes the executor; audit reconstructs actor, action, decision, reason, policy version, SHA and order (FR-23). |

### 2. Control plane shape and the team-server boundary

**Question:** what is the control plane, what runs on the server, the runner and the thin client, and does baton build its runner plumbing (B inside C) or reuse an existing OSS daemon or platform (E)? (PRD §Handoffs item 1; VISION §Open questions item 5; R2 §Options.)

**Option 1: B inside C, a baton team server and baton runners behind provider-neutral contracts (Preferred)**

```mermaid
flowchart LR
    CLI[Thin CLI]
    subgraph Boundary["Customer boundary"]
        subgraph Server["Team server (server mode)"]
            API[Team API + OIDC]
            POL[Policy + gates]
            Q[Queue + scheduler + reconciler]
            AUD[Audit + budgets]
            GH[Git-host integration]
        end
        DB[(Postgres)]
        subgraph Runner["Runner (runner mode)"]
            EX[Executor: worktree or container]
            AD[Agent adapter]
        end
    end
    MP[Model providers]
    HOST[Git host + CI]
    CLI -->|HTTPS| API
    Server --- DB
    Runner -->|outbound only| Q
    AD --> MP
    GH --> HOST
    EX -->|read-only clone| HOST
```

| Component | Responsibilities | Runs where |
|---|---|---|
| Team server | API, OIDC, roles, policy, queue, leases, reconciler, gates, audit, budgets, Git-host writes and reads, extension registry, heartbeat | Customer host, Compose service |
| Postgres | System of record for every durable state in §5 | Customer host, Compose service |
| Runner | Outbound claims, checkout, executors, adapters, event streaming, local ceilings, commit capture, pre-PR checks, cleanup | Customer host or VM; one in Compose, more on other hosts |
| Executor | One checkout and process boundary per run (§3) | Inside the runner host |
| Thin CLI | Calls the team API; holds only a user token | Member's machine |
| Git host | Code, branch protection, PR-triggered CI, merge | Customer's existing host |

**Pros:**
1. baton owns the queue, gates and audit end to end, so I3 and I5 are under its control (R2 §Options B).
2. Smallest deployable team server: one binary plus Postgres in Compose, with outbound runners (R2 §Options B; R2 S46).
3. All code is Apache-2.0 and baton's, so I2 holds without dependency review of a reused server (R3 §B).
4. Provider-neutral contracts admit C's providers later without control-plane change (R2 §Options C, §Handoffs).

**Cons:**
1. [ADVISORY] Rebuilds runner plumbing that OSS daemons already ship (R2 §5, S68 to S72).
2. [ADVISORY] Fixed P0 executors postpone external sandbox and Coder integration (R2 §Options B).
3. [ADVISORY] Runners and Postgres add operations for the customer (R2 §Options B).
4. [ADVISORY] Agent neutrality and sandbox enforcement are implementation risks shared by every option (R2 §Options, cross-option requirements).

**Why is this option preferred?** No BLOCKING con; the smallest design keeping every invariant under baton's control, and R2's advisory preference (MED). Reversible toward Option 2 or 3 if the B-versus-E spike (Action Item 2) shows reuse is faster without weakening I2, I3 or I5.

**Option 2: E, reuse an Apache-2.0 OSS daemon (Paseo-class) as runner plumbing under a baton server**

**Pros:**
1. An Apache-2.0 daemon with SDK, Docker self-host and plugins exists (R2 S69).
2. The licence is compatible with the core (R2 S69; R3 §B).

**Cons:**
1. [ADVISORY] Forking risks upstream divergence (R2 §Options E).
2. [ADVISORY] Task, gate and merge durability still has to be built (R2 §Options E).
3. [ADVISORY] Lease, fencing and outbound-only behaviour undocumented in approved sources (UNVERIFIED).
4. [ADVISORY] Its upstream offers team features that compete with baton's (R3 §C).
5. [ADVISORY] Shared neutrality and sandbox risk, as Option 1 con 4.

**Why not preferred:** no BLOCKING con; advisory grounds only (R2 §Options E). Viable; the spike decides.

**Option 3: E, integrate customer-installed Coder through its API as the execution substrate**

**Pros:**
1. Customer workspaces on VMs, containers or Kubernetes through Terraform templates (R2 S55).
2. Out-of-process API use imports no AGPL source (R2 §Options E).
3. Matches R3's advice to integrate Coder as a provider (R3 §D, Recommendations).

**Cons:**
1. [ADVISORY] A second platform to install lengthens the NFR-1 path (inference).
2. [ADVISORY] Coder's paid governance overlaps baton's gates (R2 §Options E; R2 S56).
3. [ADVISORY] Shared neutrality and sandbox risk, as Option 1 con 4.

**Why not preferred:** no BLOCKING con while baton keeps its own executors beside it; on advisory grounds it fits as a later C provider (Out of Scope 5). A variant replacing baton's executors fails I3.

**Option 4: E-embedded, embed Coder's server or source in the core**

**Pros:**
1. One integrated workspace platform.

**Cons:**
1. [BLOCKING] Embeds AGPL source in an Apache-only core, violating I2 (R2 §Options E; R3 §B).
2. [ADVISORY] Adds a large dependency surface.

**Why not:** infeasible because of con 1 alone; con 2 is advisory and overruling it does not change this verdict.

**Option 5: A, local-first daemon and UI with no shared server**

**Pros:**
1. Simplest local setup with object reuse (R2 S01, S68, S69).

**Cons:**
1. [BLOCKING] No shared server, so no customer-hosted team control plane: violates I1 (R2 §Options A).
2. [ADVISORY] A crowded OSS slot (R1 Analysis).

**Why not:** infeasible because of con 1 alone; con 2 is advisory and overruling it does not change this verdict. Its daemon pattern survives as the runner.

**Option 6: D, vendor-hosted control plane with customer-side runners only**

**Pros:**
1. Little customer control-plane operation; Claude and Cursor show customer-side execution (R2 S30, S75).

**Cons:**
1. [BLOCKING] The central software is not customer-hosted, violating I1 (R2 §Options D).
2. [ADVISORY] Transcripts and model inputs still leave the customer (R2 S30, S75).

**Why not:** infeasible because of con 1 alone; con 2 is advisory and overruling it does not change this verdict. A later hosted edition reuses Option 1's boundaries.

**Comparison table**

| Criterion | 1 B in C | 2 OSS daemon | 3 Coder API | 4 Coder embedded | 5 A | 6 D |
|---|---|---|---|---|---|---|
| Customer-hosted team plane (I1) | Yes | Yes | Yes | Yes | No | No |
| Licence boundary (I2) | Yes | Yes | Yes | No | Yes | n/a |
| Gates and audit owned by baton | Yes | Yes | Yes | Yes | No | Vendor |
| Extra platform to operate | No | No | Coder | Coder | No | Vendor |
| BLOCKING | None | None | None | I2 | I1 | I1 |

Verdict from BLOCKING items alone: Options 4, 5 and 6 are eliminated; Options 1, 2 and 3 survive and BLOCKING items cannot rank them. Option 1 is preferred on advisory grounds, matching R2's verdict.

### 3. Execution model and trust tiers

**Question:** how do checkout separation and isolation relate, which tier serves lower-trust work by default (VISION §Open questions item 4; PRD Open question 5), and how do runners talk to the server (PRD §Handoffs item 3)?

#### 3.1 Tier model

**Option 1: Checkout always separate; isolation a separate, labelled tier (Preferred)**

Every run gets its own checkout and branch (FR-9). Tier is a policy attribute shown before dispatch (FR-10): `trusted-local` (worktree from a runner-owned mirror, or a clone where submodules need it, R2 S01, S02), hardened container (checkout in the container's own volume), and a reserved microVM tier (Out of Scope 5). Checkout paths are confined; symlinks and traversal are rejected (R2 §Analysis/Execution).

**Pros:**
1. Matches R2's finding that checkout separation is not isolation (R2 §1).
2. Lets cheap cooperative work and lower-trust work coexist under one policy (R2 §1 isolation table).

**Cons:**
1. [ADVISORY] Two executor paths to build and test (R2 §Scope).

**Why is this option preferred?** No BLOCKING con; it is the only option satisfying I3 and FR-10 together.

**Option 2: Worktree only, all runs `trusted-local`**

**Pros:** 1. Fastest and cheapest (R2 §1).
**Cons:** 1. [BLOCKING] No container executor, so I3 fails (FR-11). 2. [ADVISORY] Host credentials exposed to every run (R2 S01).
**Why not:** infeasible because of con 1 alone; con 2 is advisory and overruling it does not change this verdict.

**Option 3: Container only, no `trusted-local` tier**

**Pros:** 1. One security posture for every run.
**Cons:** 1. [BLOCKING] No worktree executor, so I3 fails (FR-4 acceptance needs both). 2. [ADVISORY] Image and setup cost for cooperative work (R2 S03).
**Why not:** infeasible because of con 1 alone; con 2 is advisory and overruling it does not change this verdict.

**Option 4: Tier implied by checkout type (clone treated as isolated)**

**Pros:** 1. No separate tier attribute.
**Cons:** 1. [BLOCKING] A clone is not a security boundary (R2 S02), so the label would misstate isolation, violating FR-10. 2. [ADVISORY] Hides the choice from policy.
**Why not:** infeasible because of con 1 alone; con 2 is advisory and overruling it does not change this verdict.

#### 3.2 Default tier for lower-trust work

R2's provider contracts set the trade: self-operated Firecracker needs KVM, a jailer, guest images and operator-managed volumes, network and secrets (R2 S04); hosted providers bill per second, some for idle memory, and differ on persistence, egress and secret injection (R2 §1 provider table, S05 to S19, S79, S80); a hardened container (rootless, dropped capabilities, seccomp, no daemon socket, quotas, egress policy) runs on the customer's runner but shares the kernel (R2 S03).

**Option 1: Hardened container by default, with a bounded eligibility envelope (Preferred)**

Repos default to the container tier; `trusted-local` is a per-repo org-admin opt-in. The container tier is eligible for cooperative and controlled work on the org's own repos; hostile-code workloads are refused in P0, not downgraded (PRD Goal 6; Out of Scope 14). Egress is deny by default with allowlists for model providers and package sources (FR-21). The pilot tier stays UNRESOLVED until R3 §G item 1 and a pilot threat review run (Assumption 2; PRD §Handoffs item 3).

**Pros:**
1. Self-hostable with no paid account (I11; R2 S03).
2. Within the 12-week slice R2 scopes (R2 §Scope).
3. Fail-closed default: convenience requires an explicit admin choice.

**Cons:**
1. [ADVISORY] Shared kernel; not a hostile-code boundary (R2 §1 isolation table).
2. [ADVISORY] Agent code can read secrets injected into its environment, as with several providers (R2 S79); mitigated by per-run scoped keys and egress allowlists.

**Why is this option preferred?** No BLOCKING con; it meets I11 and I12 at the lowest operating cost, and its weakness is bounded by eligibility rather than hidden.

**Option 2: Self-operated microVM (Firecracker) by default**

**Pros:** 1. Stronger untrusted-code boundary (R2 §1 isolation table).
**Cons:** 1. [ADVISORY] KVM, jailer, images, volumes and a credential broker to build and operate (R2 S04). 2. [ADVISORY] Outside R2's 12-week slice (R2 §Scope). 3. [ADVISORY] Lengthens the NFR-1 path (inference).
**Why not preferred:** no BLOCKING con; not preferred on advisory grounds. It is the reserved tier when Assumption 2 fails.

**Option 3: Hosted microVM or sandbox provider by default (E2B, Vercel, Daytona VM, Fly, Cloudflare)**

**Pros:** 1. Managed boundary, snapshots and egress controls (R2 S05 to S19).
**Cons:** 1. [BLOCKING] A default needing a paid hosted sandbox fails I11 (NFR-7). 2. [ADVISORY] Code and checkouts leave the customer boundary (R2 §Handoffs). 3. [ADVISORY] Idle memory and storage charges defeat token-only budgets (R2 §Analysis/Execution).
**Why not:** infeasible as the default because of con 1 alone; cons 2 and 3 are advisory and overruling them does not change this verdict. Optional providers remain in scope later.

**Option 4: User-space-kernel container runtime by default (the runtime class Modal offers, R2 S10)**

**Pros:** 1. Narrows the shared-kernel exposure named in R2 S03 (inference).
**Cons:** 1. [ADVISORY] No approved source covers a self-hosted runtime of this class (UNVERIFIED). 2. [ADVISORY] Agent and tool compatibility untested.
**Why not preferred:** no BLOCKING con; not preferred on advisory grounds. Candidate hardening for the execution HLD.

**Option 5: No default; policy must name a tier before any run**

**Pros:** 1. Every tier choice is explicit.
**Cons:** 1. [ADVISORY] First-run friction against NFR-1. 2. [ADVISORY] Invites a convenience choice of `trusted-local` (inference).
**Why not preferred:** no BLOCKING con; Option 1 keeps the explicitness for `trusted-local` while giving a safe default.

**Comparison table**

| Criterion | 1 Container | 2 Own microVM | 3 Hosted | 4 User-space kernel | 5 No default |
|---|---|---|---|---|---|
| No paid account (I11) | Yes | Yes | No | Yes | Yes |
| Hostile code | No | Stronger | Stronger | Unverified | n/a |
| In 12-week slice | Yes | No | Spike only | Unverified | Yes |
| BLOCKING | None | None | I11 | None | None |

#### 3.3 Runner-to-server contract

**Option 1: Outbound-only runner with leased claims (Preferred)**

The runner authenticates, connects outbound, registers its platform, executors, adapters and labels, and claims eligible runs under leases with fencing epochs renewed by heartbeat. The same channel carries ordered events, artifacts, candidate commits and terminal status up, and cancels and permission decisions down. Per-run credentials expire with the lease; the runner credential can call only these operations (R2 §Options B, §Analysis/Durability).

**Pros:** 1. No inbound ports into runner networks. 2. Disconnected runners leave runs queued (FR-12). 3. Fencing rejects stale writers (FR-14).
**Cons:** 1. [ADVISORY] Cancellation latency is bounded by the channel or heartbeat interval.
**Why is this option preferred?** No BLOCKING con; it satisfies FR-12, FR-14 and I4 with the least network exposure.

**Option 2: Server pushes work to runners over inbound connections**

**Pros:** 1. Immediate dispatch and cancel.
**Cons:** 1. [ADVISORY] Needs inbound reachability into customer networks. 2. [ADVISORY] Server must track runner addresses.
**Why not preferred:** no BLOCKING con; not preferred on advisory grounds.

**Option 3: Dispatch runs as Git-host CI jobs on self-hosted CI runners**

**Pros:** 1. Reuses an existing runner fleet, as Agent HQ does (R2 S66).
**Cons:** 1. [ADVISORY] Ties execution to one host's CI, weakening FR-20 neutrality. 2. [ADVISORY] CI job identities often carry repo write scopes (inference).
**Why not preferred:** no BLOCKING con; not preferred on advisory grounds.

**Option 4: Runners read and write the database directly**

**Pros:** 1. No runner API to build.
**Cons:** 1. [BLOCKING] A database credential on the runner can write gate and approval records, violating I4. 2. [ADVISORY] Exposes Postgres to runner networks.
**Why not:** infeasible because of con 1 alone; con 2 is advisory and overruling it does not change this verdict.

**Comparison table**

| Sub-decision | Preferred | Eliminated by BLOCKING | Viable, not preferred |
|---|---|---|---|
| 3.1 | Separate tier | 2, 3 (I3), 4 (FR-10) | none |
| 3.2 | Container | 3 (I11) | 2, 4, 5 |
| 3.3 | Outbound leased | 4 (I4) | 2, 3 |

### 4. Agent adapter contract

**Question:** through which contract does an agent become first-class, and where are capability differences surfaced? (PRD §Handoffs item 2; VISION §Design principles.)

Capabilities the contract must carry (R2 §2; S-ids from R2):

| Agent | Headless structured events | Resume | Permission callback | ACP |
|---|---|---|---|---|
| Claude Code | Yes; skips trust prompt, may load project hooks and MCP (S20) | Session ID | Agent SDK hooks (S21) | Adapter (S36) |
| Codex | Exec; SDK and app-server stream events, approvals (S23, S24) | Threads | App-server | Adapter (S37) |
| OpenCode | SDK server events (S25) | Sessions | Via SDK | Native, documented gaps (S26) |
| Cursor CLI | Print mode (S27) | Yes | ACP (S28) | Native |
| Gemini CLI | Stream JSON (S31) | Session load | ACP (S74) | Native |
| Kiro CLI | Stream JSON, API key (S32) | Session IDs | ACP (S33) | Native |
| Aider | One-shot message mode only (S34) | No | No | UNVERIFIED |

**Option 1: baton-owned normalized contract with per-agent transports (Preferred)**

One contract covers start, ordered events from a cursor, input, permission answers, cancel, resume, usage, artifacts, capability declaration and dispose (R2 §Analysis/Execution). Each adapter uses its agent's best transport: headless CLI stream, vendor SDK or ACP. Adapters start the agent inside the run's executor with per-run scoped API keys, never a copied home directory or consumer login (FR-6). Capabilities are declared and versioned at registration; the server marks unsupported features unavailable before dispatch and denies and audits undeclared requests at run time (FR-5). An agent with no permission callback runs with preconfigured tools under its tier as the hard gate; authority is never silently upgraded (R2 §Analysis/Execution). The agent's own sandbox options are set explicitly; baton's tier stays the boundary (R2 §2).

**Pros:**
1. Every agent in the table fits, including CLI-only Aider.
2. Usage, artifacts and capability flags, which ACP lacks, have a home (R2 §2).
3. ACP (FR-7) and MCP tools (FR-8) become transports and tools, not new contracts.

**Cons:**
1. [ADVISORY] baton maintains a contract and golden event fixtures per adapter (R2 §Scope; Assumption 13).

**Why is this option preferred?** No BLOCKING con; it is the only option that admits every listed agent while keeping FR-5 enforceable.

**Option 2: ACP as the sole adapter contract**

**Pros:** 1. A standard with permission requests and native support in four agents (R2 S35, S74). 2. Adapters exist for Claude and Codex (R2 S36, S37).
**Cons:** 1. [ADVISORY] Claude and Codex depend on third-party adapters, not native CLI ACP (R2 §2). 2. [ADVISORY] No usage, billing, durable task or gate semantics, so baton adds them anyway (R2 §2). 3. [ADVISORY] Agents without ACP are excluded (R2 S34).
**Why not preferred:** no BLOCKING con; not preferred on advisory grounds. Kept as a transport inside Option 1.

**Option 3: MCP as the adapter contract**

**Pros:** 1. Broad tool ecosystem (R2 S38).
**Cons:** 1. [BLOCKING] MCP defines no agent session, cancellation or supervision semantics (R2 §2), so FR-13 cancellation and FR-5 enforcement have no carrier. 2. [ADVISORY] Extending it yields a private protocol.
**Why not:** infeasible because of con 1 alone; con 2 is advisory and overruling it does not change this verdict.

**Option 4: A2A as the adapter contract**

**Pros:** 1. Task states, artifacts and streaming between agents (R2 S39).
**Cons:** 1. [BLOCKING] The agent is opaque and runs in its own environment, so baton cannot place it in a tier or own its checkout, failing I3 and FR-10 (R2 §2). 2. [ADVISORY] None of the first-pair agents is documented as an A2A agent in R2.
**Why not:** infeasible because of con 1 alone; con 2 is advisory and overruling it does not change this verdict.

**Option 5: Headless CLI wrapping only**

**Pros:** 1. Universal across the table (R2 §2).
**Cons:** 1. [ADVISORY] Loses richer approvals and streams from SDKs and app-servers (R2 S21, S24). 2. [ADVISORY] CLI output formats change (Assumption 13).
**Why not preferred:** no BLOCKING con; it is a subset of Option 1.

**Option 6: Vendor SDKs only**

**Pros:** 1. Structured events and approvals (R2 S21, S24, S25).
**Cons:** 1. [ADVISORY] Unofficial for some agents (R2 S34). 2. [ADVISORY] SDK languages and stability vary.
**Why not preferred:** no BLOCKING con; also a subset of Option 1.

**Comparison table**

| Criterion | 1 Normalized | 2 ACP | 3 MCP | 4 A2A | 5 CLI | 6 SDK |
|---|---|---|---|---|---|---|
| Admits all listed agents | Yes | No | No | No | Yes | No |
| Cancel and capability enforcement | Yes | Yes | No | Partial | Partial | Yes |
| Tier placement by baton | Yes | Yes | Yes | No | Yes | Yes |
| BLOCKING | None | None | FR-13, FR-5 | I3 | None | None |

**First pair.** The contract supports both PRD pairs: Claude Code CLI with Codex CLI (preferred), or one permitted proprietary CLI with an open harness such as OpenCode; the written vendor check selects (PRD option set 2; Assumption 3, UNRUN).

### 5. Durable state and the run lifecycle

**Question:** where do runs, events, gate results and audit live, how is work queued, claimed and recovered, and what is retained? (PRD §Handoffs item 4; NFR-10.)

**Option 1: Postgres as system of record with queue, outbox, leases and a reconciler (Preferred)**

Postgres holds org, teams, members, repos, policy versions, tasks, runs, leases, ordered run events, candidates, checks, approvals, budgets, audit, the outbox and the runner registry. One transaction appends the event, changes run state, writes audit and enqueues the next stage or side effect (R2 §Analysis/Durability). The server grants claims using row locks with skip-locked selection (R2 S46); runner-side sequence numbers make event appends idempotent. Outbox workers perform external effects (push, PR creation, status reads) at least once, deduplicated by branch name, tagged PR lookup and provider idempotency keys; exactly-once is not claimed (R2 §Analysis/Durability). Logs and transcripts sit in a server-managed artifact store referenced from Postgres; backup covers both (NFR-8).

```mermaid
stateDiagram-v2
    [*] --> Queued: policy allows
    Queued --> Claimed: lease granted
    Claimed --> Preparing
    Preparing --> Running
    Running --> AwaitingInput: permission or steering
    AwaitingInput --> Running
    Running --> Candidate: agent done, SHA recorded
    Candidate --> Checks: pre-PR checks
    Checks --> Review: checks pass
    Review --> PROpen: SHA approved
    PROpen --> MergeEligible: host checks pass at SHA
    PROpen --> Candidate: new commit, gates stale
    MergeEligible --> Candidate: new commit, gates stale
    MergeEligible --> [*]: merged on host
    Claimed --> NeedsReconciliation: lease lost, state ambiguous
    Preparing --> NeedsReconciliation: lease lost, state ambiguous
    Running --> NeedsReconciliation: lease lost, state ambiguous
    NeedsReconciliation --> Queued: operator confirms no writer
    NeedsReconciliation --> Failed: operator closes
    Running --> Cancelled: cancel or time limit
    Checks --> Failed: check fails
    Review --> Rejected: reviewer denies
```

Recovery rules: a lost lease whose process and executor the reconciler confirms dead requeues with a new epoch; an ambiguous one enters needs-reconciliation and never gets a second writer on the same checkout (FR-14; R2 §Analysis/Durability). Restart loses nothing committed; gates stay ineligible until re-evaluated (NFR-3). A periodic poll backs up webhooks (R2 §Analysis/Durability).

Retention (PRD NFR-10 assigns it here): the product never deletes basic audit events in P0; run events and optional payloads (prompts, tool I/O, transcripts) have operator-set finite retention, default fixed in the LLD, and each purge is audited. Payloads are stored apart from audit with separate access control (R2 §Analysis/Execution).

**Pros:**
1. One dependency already in Compose (I7; R2 §Analysis/Durability).
2. State change, audit and queue commit atomically, which I9 needs.
3. Honest recovery boundary: at-least-once effects, explicit ambiguity.

**Cons:**
1. [ADVISORY] Timers, human waits and replay are hand-built; reassess if they grow (Assumption 7).
2. [ADVISORY] Queue load and the database share one server (inference).

**Why is this option preferred?** No BLOCKING con; it meets I7, I9 and FR-14 with the fewest moving parts (R2 Recommendations, MED).

**Option 2: Embedded durable-workflow library on Postgres (DBOS-class)**

**Pros:** 1. Durable workflows and Postgres-backed queues in-process; MIT library (R2 S45).
**Cons:** 1. [ADVISORY] Its execution model shapes the codebase. 2. [ADVISORY] The approved sources cite only its TypeScript library (R2 S45). 3. [ADVISORY] External effects stay at least once anyway.
**Why not preferred:** no BLOCKING con; it is the first reassessment path under Assumption 7.

**Option 3: Temporal as a required orchestration backend**

**Pros:** 1. Durable workflows, timers, activities and recovery (R2 S40).
**Cons:** 1. [BLOCKING] Production needs a Temporal Service, persistent store and workers, a separate workflow cluster that violates I7; the single-binary dev server is not for production (R2 S40). 2. [ADVISORY] Higher operations cost (R2 §3).
**Why not:** infeasible as a P0 requirement because of con 1 alone; con 2 is advisory and overruling it does not change this verdict. A later optional backend is not excluded.

**Option 4: Bundled source-available durable-execution server (Restate BSL 1.1, Inngest SSPL)**

**Pros:** 1. Journaling, durable steps; Inngest ships a single binary (R2 S41, S42).
**Cons:** 1. [BLOCKING] Bundling a BSL or SSPL server in the core violates I2 (R2 S41, S43; R3 §B). 2. [ADVISORY] Restate's cited deployment route is Kubernetes (R2 S41).
**Why not:** infeasible because of con 1 alone; con 2 is advisory and overruling it does not change this verdict.

**Option 5: Trigger.dev self-hosted**

**Pros:** 1. Compose deployment exists (R2 S44).
**Cons:** 1. [BLOCKING] Adds webapp and worker services plus Redis, a separate workflow-engine deployment that violates I7 (R2 S44). 2. [ADVISORY] Checkpoints, warm starts and autoscaling are cloud-only (R2 S44). 3. [ADVISORY] Server licence not stated in the approved sources (UNVERIFIED).
**Why not:** infeasible because of con 1 alone; cons 2 and 3 are advisory and overruling them does not change this verdict.

**Option 6: Separate message broker beside Postgres**

**Pros:** 1. Push delivery and fan-out.
**Cons:** 1. [ADVISORY] A third Compose service. 2. [ADVISORY] Dual writes still need an outbox (inference).
**Why not preferred:** no BLOCKING con; not preferred on advisory grounds.

**Option 7: Embedded single-writer database, no Postgres**

**Pros:** 1. Infrastructure-free binary (R2 §Analysis/Durability).
**Cons:** 1. [ADVISORY] One active writer suits a single-user local mode, not concurrent team claims, locks and backups (R2 §Analysis/Durability, MED).
**Why not preferred:** no BLOCKING con; not preferred on advisory grounds. A later single-user mode may use it.

**Comparison table**

| Criterion | 1 Postgres | 2 Library | 3 Temporal | 4 BSL/SSPL | 5 Trigger.dev | 6 Broker | 7 Embedded |
|---|---|---|---|---|---|---|---|
| Single Compose, no cluster (I7) | Yes | Yes | No | Yes | No | Yes | Yes |
| Licence boundary (I2) | Yes | Yes | Yes | No | Unverified | Yes | Yes |
| Atomic state + audit | Yes | Yes | Partial | Partial | Partial | Outbox | Yes |
| BLOCKING | None | None | I7 | I2 | I7 | None | None |

### 6. Gates, policy, audit and budgets

**Question:** where gates sit relative to the Git host's PR-triggered CI, how policy is represented, how audit is written, and where budgets are enforced (PRD §Handoffs items 5 and 6).

#### 6.1 Gate placement

**Option 1: Two-phase gate (Preferred).** Pre-PR gate in baton: policy, pre-PR checks at the candidate SHA in a fresh executor, and a distinct human's approval of that SHA; baton then opens the PR; merge eligibility needs the host's PR-triggered checks passing at the same SHA, and a human merges under branch protection (FR-16, FR-18; R2 S73). A new commit stales both phases; an evaluation error, missing check or missing audit commit means ineligible.
**Pros:** 1. Executable order: no gate waits for an event it precedes (I5). 2. Keeps host protection and merge authority (I6). 3. Fresh-executor checks cannot be altered by leftover agent processes.
**Cons:** 1. [ADVISORY] A second checkout per candidate costs time (NFR-2 measures it).
**Why is this option preferred?** No BLOCKING con; the only option meeting I5 and I6 together.

**Option 2: Host-only gate, draft PR first.** **Pros:** 1. Reuses mature host enforcement (R2 S73). **Cons:** 1. [BLOCKING] Opens the PR before the human gate, violating I6 (FR-18). 2. [ADVISORY] Gate rules become host-specific, against FR-20. **Why not:** infeasible because of con 1 alone; con 2 is advisory and overruling it does not change this verdict.

**Option 3: baton is the only gate and merges.** **Pros:** 1. One place to reason about. **Cons:** 1. [BLOCKING] Merge automation bypasses host authority, violating I6 (PRD Out of scope 9). 2. [ADVISORY] Duplicates host protection. **Why not:** infeasible because of con 1 alone; con 2 is advisory and overruling it does not change this verdict.

**Option 4: Single pre-PR gate that includes host CI.** **Pros:** 1. One gate. **Cons:** 1. [BLOCKING] PR-triggered checks cannot run before the PR exists, violating I5. 2. [ADVISORY] Duplicates CI on the runner. **Why not:** infeasible because of con 1 alone; con 2 is advisory and overruling it does not change this verdict.

#### 6.2 Policy representation

**Option 1: Bounded declarative document per org with per-repo overrides, evaluated in-process (Preferred).** Covers agents, tiers, tools, network, paths, time and concurrency, money, required approvals and checks (FR-21; R2 §Analysis/Execution, MED). Repo overrides may tighten freely; loosening needs an org admin; deny wins. Each version is hashed and recorded with every decision (FR-23). Decision is separate from enforcement so OPA or Cedar can later sit behind the same seam (R2 S58, S59).
**Pros:** 1. Testable, readable rules for the bounded alpha. 2. Errors deny in the application (FR-22).
**Cons:** 1. [ADVISORY] Less expressive than a policy language.
**Why is this option preferred?** No BLOCKING con; smallest surface that meets FR-21 and FR-22; R2 advises choosing an engine when customers need one.

**Option 2: Embedded OPA/Rego.** **Pros:** 1. Arbitrary inputs; embeddable library (R2 S58). **Cons:** 1. [ADVISORY] Operator learning curve. 2. [ADVISORY] Compatibility demand unknown (R2 §Analysis/Execution). **Why not preferred:** no BLOCKING con; later evaluator.

**Option 3: Cedar.** **Pros:** 1. Principal, action, resource, context; default deny (R2 S59). **Cons:** 1. [ADVISORY] May skip erroneous policies, so the application must still deny on diagnostics (R2 §Analysis/Execution). 2. [ADVISORY] Fits authorization better than budgets and egress. **Why not preferred:** no BLOCKING con; later evaluator for roles.

**Option 4: Repo-resident policy read from the protected default branch.** **Pros:** 1. Policy reviewed like code. **Cons:** 1. [ADVISORY] Policy authority moves to host merge rights, not baton's admin role (FR-26). 2. [ADVISORY] Per-host semantics. **Why not preferred:** no BLOCKING con; a variant reading policy from the run's own branch fails I4 (self-authorization) and is not this option.

#### 6.3 Audit write path

**Option 1: Append-only audit events in the same transaction as the decision (Preferred).** Each event records actor, org, repo, run, sequence, action, decision, reason, policy hash, candidate SHA, time, provider IDs and redaction status (R2 §Analysis/Execution). Secrets issued to the run are scrubbed before write (NFR-5). The application's database role cannot update or delete audit rows (verified-by: LLD privilege test). Tamper-evident chains and export come later (Out of Scope 16).
**Pros:** 1. I9 holds by construction. **Cons:** 1. [ADVISORY] Audit volume shares the primary database.
**Why is this option preferred?** No BLOCKING con; the only option meeting I9.

**Option 2: Asynchronous audit stream to a separate sink.** **Pros:** 1. Offloads volume. **Cons:** 1. [BLOCKING] A decision can commit before its audit write, violating I9. **Why not:** infeasible because of con 1 alone.

**Option 3: Treat the run activity feed as audit.** **Pros:** 1. No extra store. **Cons:** 1. [BLOCKING] A mutable feed is not append-only audit (R2 §Analysis/Execution; FR-23). **Why not:** infeasible because of con 1 alone.

#### 6.4 Budget enforcement point

**Option 1: Server admission plus runner hard stop (Preferred).** The server enforces concurrency at claim time and monetary reservations per run, repo and org; the runner enforces time, CPU, memory and storage ceilings locally, even when disconnected, and stops new agent calls at a supported cap. Usage from adapters debits atomically; opaque charges show unknown and never claim a dollar cap (FR-24, FR-25; R2 §Analysis/Execution).
**Pros:** 1. Holds under partition and across runners (I10). **Cons:** 1. [ADVISORY] Two enforcement points to test.
**Why is this option preferred?** No BLOCKING con; the only option meeting I10 at both scopes.

**Option 2: Server only.** **Pros:** 1. One enforcement point. **Cons:** 1. [BLOCKING] A partitioned runner runs past its time limit, violating I10. **Why not:** infeasible because of con 1 alone.

**Option 3: Runner only.** **Pros:** 1. Works offline. **Cons:** 1. [BLOCKING] No runner sees org-wide concurrency, violating I10. **Why not:** infeasible because of con 1 alone.

**Comparison table**

| Sub-decision | Preferred | Eliminated by BLOCKING | Viable, not preferred |
|---|---|---|---|
| 6.1 | Two-phase | 2 (I6), 3 (I6), 4 (I5) | none |
| 6.2 | Declarative | branch-read variant (I4) | 2, 3, 4 |
| 6.3 | Same transaction | 2 (I9), 3 (FR-23) | none |
| 6.4 | Split | 2, 3 (I10) | none |

### 7. Git-host integration boundary

**Question:** how many hosts in P0, and who writes the candidate branch? (PRD FR-18 to FR-20, option set 3.)

#### 7.1 Host breadth

**Option 1: Host contract with one host in P0 (GitHub) and a second (GitLab) as P1 (Preferred).** The contract covers read-only clone credentials, candidate-branch push, PR creation, PR-triggered status and protection reads, and webhooks with polling as the floor. GitHub goes first because its protection, stale-review and status-check semantics are in the approved sources (R2 S73); GitLab's are not (UNVERIFIED, Git-host LLD). **Pros:** 1. Meets FR-19 in the slice (R2 §Scope). 2. FR-20 needs no gate change. **Cons:** 1. [ADVISORY] Host neutrality stays untested until P1. **Why is this option preferred?** No BLOCKING con; PRD default, swappable under Assumption 4.

**Option 2: Two hosts in P0.** **Pros:** 1. Proves a non-GitHub PR path. **Cons:** 1. [ADVISORY] Second mapping costs alpha time. **Why not preferred:** no BLOCKING con; becomes preferred if PRD option set 3 resolves to A before approval.

**Option 3: GitHub only, no host contract.** **Pros:** 1. Least code. **Cons:** 1. [ADVISORY] FR-20 later needs gate rework. **Why not preferred:** no BLOCKING con; advisory.

**Option 4: Plain Git push, humans open PRs.** **Pros:** 1. Any host. **Cons:** 1. [BLOCKING] No PR creation or merge-status surface, failing FR-18. **Why not:** infeasible because of con 1 alone.

#### 7.2 Candidate branch writer

**Option 1: Server pushes commits the runner transfers (Preferred).** The runner commits outside the agent's control and uploads the new commits; the server pushes with its Git-host identity and records the SHA. Runners hold only read-only, per-run clone credentials. **Pros:** 1. Write credentials never reach runners or executors (NFR-5). 2. The server is the SHA authority (FR-15). 3. Same path for later hosted providers (R2 §Handoffs). **Cons:** 1. [ADVISORY] Large diffs travel through the server. **Why is this option preferred?** No BLOCKING con; least privilege.

**Option 2: Runner pushes with a per-run token.** **Pros:** 1. No transfer via server. **Cons:** 1. [ADVISORY] On `trusted-local` the agent shares the runner's user and can read the token (R2 S01). **Why not preferred:** no BLOCKING con; advisory.

**Option 3: Agent pushes from inside the executor.** **Pros:** 1. Simplest. **Cons:** 1. [ADVISORY] Write token inside the lower-trust tier. 2. [ADVISORY] Pushes after capture stale the gate. **Why not preferred:** no BLOCKING con; advisory.

**Comparison table**

| Sub-decision | Preferred | Eliminated by BLOCKING | Viable, not preferred |
|---|---|---|---|
| 7.1 | Contract, one host | 4 (FR-18) | 2, 3 |
| 7.2 | Server push | none | 2, 3 |

### 8. Identity, roles and tenancy

**Question:** which identities and roles exist in P0, and what seam will later multi-org tenancy use? (PRD §Handoffs item 7; FR-26, FR-34, NFR-6.)

Identities: human users sign in through the core OIDC path (FR-34); CLI calls carry a user-delegated token; each runner has its own credential limited to §3.3 operations; the server holds the Git-host identity; agent API keys are stored as secret references per org or repo (NFR-5). The first admin binds through a one-time install token. Roles: org admin, member (submitter), reviewer; teams group members and repos; repo membership scopes every read, including audit (NFR-6). The execution identity has no role (I4).

**Option 1: One org per instance, every record carries an org identifier (Preferred).** Every query is scoped by org and repo membership in the API layer; P0 has one org row. Later tenancy adds org rows plus separate runner pools, credentials, networks, volumes and payload access (R2 §Analysis/Self-host). **Pros:** 1. Cheap seam now. **Cons:** 1. [ADVISORY] Scoping is tested, not enforced by the database. **Why is this option preferred?** No BLOCKING con; cheapest seam.

**Option 2: No org dimension in P0.** **Pros:** 1. Simplest schema. **Cons:** 1. [ADVISORY] Multi-org later touches every record and query. **Why not preferred:** no BLOCKING con; advisory.

**Option 3: Multi-org with row-level security now.** **Pros:** 1. Database-enforced scoping. **Cons:** 1. [ADVISORY] Security-definer and service-role bypasses need testing (R2 S48). 2. [ADVISORY] Beyond P0 (PRD Out of scope 3). **Why not preferred:** no BLOCKING con; advisory.

**Option 4: One instance per org as the multi-org model.** **Pros:** 1. Strongest separation, no shared tables. **Cons:** 1. [ADVISORY] Per-org operations; cross-org Enterprise RBAC harder (R3 §B). **Why not preferred:** no BLOCKING con; stays available as a deployment pattern.

Enterprise SSO/SCIM attaches at the identity boundary and multi-org RBAC at the org dimension and role model, both outside the core (R3 §B).

**Comparison table**

| Criterion | 1 Org id | 2 None | 3 RLS | 4 Per instance |
|---|---|---|---|---|
| Later tenancy cost | Low | High | Built | Per-org ops |
| BLOCKING | None | None | None | None |

### 9. Extension points and the open-core boundary

**Question:** how do adapters, executors, hosts, gates, roles and branding extend the core, and where is the open-core line? (FR-29, FR-30; R3 §B.)

Core (Apache-2.0): server, runner, CLI, API, all agent and executor adapters, Git-host providers, policy, gates, budgets, basic audit, single-org roles, OIDC, opt-in heartbeat, the presentation extension point, backup and upgrade tools. Enterprise (source-available): SSO/SCIM, multi-org RBAC, audit export, policy packs with approvals, HA, air-gap bundles, branding controls, cross-vendor cost accounting, SLA (R3 §B). Contributions use DCO without a CLA, and a dependency-licence check guards I2 (R3 Handoffs; R2 §Open questions).

#### 9.1 Extension mechanism

**Option 1: Compiled-in providers behind versioned contracts, plus out-of-process extensions (Preferred).** Contracts are versioned; the server rejects unsupported versions; out-of-process extensions run on runners or as separate services and never in the server; registration needs an admin grant (R2 §Analysis/Self-host). **Pros:** 1. Meets I8. **Cons:** 1. [ADVISORY] Out-of-process calls add latency. **Why is this option preferred?** No BLOCKING con.

**Option 2: In-process dynamic plugins.** **Pros:** 1. Fast, flexible. **Cons:** 1. [BLOCKING] Loaded code holds every server privilege, violating I8. **Why not:** infeasible because of con 1 alone.

**Option 3: Sandboxed in-server plugins (Wasm-class).** **Pros:** 1. In-process with confinement (inference). **Cons:** 1. [ADVISORY] No approved source on the runtime (UNVERIFIED). 2. [ADVISORY] Git and network needs widen the host interface. **Why not preferred:** no BLOCKING con; later candidate.

**Option 4: Fork to customize.** **Pros:** 1. Unlimited change. **Cons:** 1. [BLOCKING] No versioned boundary, failing FR-29 (I8). 2. [ADVISORY] Upgrade drift (R2 S53, S54). **Why not:** infeasible because of con 1 alone; con 2 is advisory and overruling it does not change this verdict.

#### 9.2 Edition packaging

**Option 1: Enterprise module in the same repository, separate build (Preferred).** The core build excludes it; it depends only on the core's public contracts; the core never imports it (R3 §B; R2 S51, S55). **Pros:** 1. Apache-only core artifact. **Cons:** 1. [ADVISORY] Build discipline needed. **Why is this option preferred?** No BLOCKING con; the R3 packaging.

**Option 2: One binary with licence-key flags.** **Pros:** 1. One artifact. **Cons:** 1. [BLOCKING] No Apache-only core artifact exists, violating I2 (NFR-9). **Why not:** infeasible because of con 1 alone.

**Option 3: Separate repository.** **Pros:** 1. Clean licence split. **Cons:** 1. [ADVISORY] Version skew. **Why not preferred:** no BLOCKING con; advisory.

**Option 4: Enterprise as a separate service.** **Pros:** 1. Strongest separation. **Cons:** 1. [ADVISORY] SSO and HA integrate poorly out of process (inference). **Why not preferred:** no BLOCKING con; suits audit export later.

**Comparison table**

| Sub-decision | Preferred | Eliminated by BLOCKING | Viable, not preferred |
|---|---|---|---|
| 9.1 | Compiled + out-of-process | 2, 4 (I8) | 3 |
| 9.2 | Same repo, separate build | 2 (I2) | 3, 4 |

### 10. Install and upgrade

**Question:** how is the alpha installed and upgraded, and how are versions kept compatible? (FR-31, NFR-8; R2 §Analysis/Self-host.)

#### 10.1 Install

**Option 1: One Compose file: server, Postgres, one runner; more runners from the same image (Preferred).** The runner is the one privileged component: it drives a rootless or user-namespaced container runtime, and no executor gets the daemon socket (I12; R2 S03; Assumption 8). **Pros:** 1. Meets I7. **Cons:** 1. [ADVISORY] A database plus binary, not an infrastructure-free install (R2 §Analysis/Durability). **Why is this option preferred?** No BLOCKING con.

**Option 2: Kubernetes chart instead of Compose.** **Pros:** 1. Production scheduling. **Cons:** 1. [BLOCKING] No Compose install, failing FR-31 (I7). **Why not:** infeasible because of con 1 alone; a chart in addition is later.

**Option 3: OS packages.** **Pros:** 1. Native service management. **Cons:** 1. [ADVISORY] Per-OS packaging. **Why not preferred:** no BLOCKING con.

#### 10.2 Upgrade and schema versioning

**Option 1: Forward-only migrations with dry-run, mandatory backup, restore as rollback (Preferred).** The database records its schema version; a binary refuses a newer schema and migrates an older one only by an explicit operator step. Policy documents, extension contracts, runner protocol and event formats carry versions; the server accepts the previous runner version; old runs stay readable (NFR-8; R2 §Analysis/Self-host). **Pros:** 1. Testable rollback boundary. **Cons:** 1. [ADVISORY] Rollback loses writes made after upgrade. **Why is this option preferred?** No BLOCKING con.

**Option 2: Reversible down-migrations.** **Pros:** 1. In-place rollback. **Cons:** 1. [ADVISORY] Doubles the test matrix; data loss remains. **Why not preferred:** no BLOCKING con.

**Option 3: Expand-and-contract zero-downtime.** **Pros:** 1. No outage. **Cons:** 1. [ADVISORY] Not promised (NFR-8); HA is Enterprise. **Why not preferred:** no BLOCKING con.

**Comparison table**

| Sub-decision | Preferred | Eliminated by BLOCKING | Viable, not preferred |
|---|---|---|---|
| 10.1 | Compose | 2 (I7) | 3 |
| 10.2 | Forward-only | none | 2, 3 |

### 11. Cross-cutting

**Security and isolation model.** Trust boundaries: client to server (OIDC, TLS); runner to server (runner credential, outbound only); runner to executor; executor to model providers and the internet; server to Git host (R2 §Analysis/Execution; NFR-5).

| Tier | Boundary | Network and secrets | Eligible work |
|---|---|---|---|
| `trusted-local` | Separate worktree; host user, network, credentials | Inherits host | Admin opt-in, cooperative same-team work (R2 §1) |
| Hardened container (default) | Own checkout volume; §3.2 hardening; no host mount or daemon socket | Default-deny egress, allowlist; scoped keys readable by agent code | Cooperative or controlled work; not hostile code (R2 S03) |
| microVM (reserved) | KVM guest via provider contract | Provider-specific broker | Hostile code, later HLD (R2 S04) |

Vendor CLIs send prompts and tool output to their model providers; baton does not claim an air gap or zero exfiltration (R2 §2; PRD Out of scope 8). Fail-closed rules: policy or audit failure denies (I9); unknown tier or executor denies (I8).

**Multi-tenancy.** Single org, multiple teams and repos; seam and options in §8.

**Failure modes and recovery.**

| Failure | Behaviour | Recovery |
|---|---|---|
| Runner dies or partitions | Runner stops the run at its time limit; lease expires | Reconciler requeues if confirmed dead, else needs-reconciliation (FR-14) |
| Duplicate dispatch | Second claim fails; stale epoch writes rejected | None needed |
| Server restart | Committed state survives; gates ineligible until re-evaluated | Runners reconnect (NFR-3) |
| Postgres down | API and claims fail closed; runners keep local ceilings | Restore from backup (NFR-8) |
| Policy or audit write fails | Transition denied, operator sees error | Retry after fix (FR-22) |
| Git host unreachable or webhook lost | Outbox retries; poll reconciles | Tagged PR lookup avoids duplicates |
| Agent hangs | Time ceiling kills it | Run ends timed out (FR-24) |
| Executor orphaned | Reconciler matches recorded executor IDs | Dispose and record |

**Observability.** Structured logs and metrics for queue depth, lease expiries, reconciliation counts, gate outcomes and per-stage durations (cold and warm checkout, container ready, ready-to-prompt) feed the NFR-2 report; provider boot claims are not used (R2 §1). The opt-in heartbeat is separate, off by default and field-documented (FR-32; Assumption 14).

**Cost model.** Server and Postgres are small fixed costs; runner compute per concurrent run and model inference on the customer's keys dominate (inference). The self-hosted product is never usage-metered (R3 §B). Hosted providers later add per-second compute, idle memory and storage, so budgets count more than tokens (R2 §1 provider table).

**Migration and rollout.** Internal use, then one pilot repo, then design partners, each behind the §G record (Action Item 1). No import tooling in P0 (Out of Scope 17). Upgrades follow §10.2. A later hosted edition reuses these boundaries with the vendor as operator (R2 §Options D mitigation).

## Appendix

**A. BLOCKING eliminations by criterion.** Each criterion carries the same tag wherever it appears (Design intro).

| Criterion | Eliminated options |
|---|---|
| I1 | §2 Options 5, 6 |
| I2 | §2 Option 4; §5 Option 4; §9.2 Option 2 |
| I3 | §3.1 Options 2, 3; §4 Option 4 |
| I4 | §3.3 Option 4; §6.2 branch-read variant |
| I5 | §6.1 Option 4 |
| I6 | §6.1 Options 2, 3 |
| I7 | §5 Options 3, 5; §10.1 Option 2 |
| I8 | §9.1 Options 2, 4 |
| I9 | §6.3 Option 2 |
| I10 | §6.4 Options 2, 3 |
| I11 | §3.2 Option 3 |
| PRD FR | §3.1 Option 4 (FR-10); §4 Option 3 (FR-13, FR-5); §6.3 Option 3 (FR-23); §7.1 Option 4 (FR-18) |

I12 eliminates no option; it shapes §3.2 and §10.1.

## References

Primary-source URLs live in the research source lists and are cited here by S-id, which meets BRIEF rule 1; this HLD adds no new source.

1. VISION.md, APPROVED v2.
2. docs/design/PRD-000-platform.md, APPROVED v2.
3. docs/research/R1-market-landscape.md, APPROVED v2.
4. docs/research/R2-technical-landscape.md, APPROVED v2; sources S01 to S80 cited as "R2 Snn".
5. docs/research/R3-business-case.md, APPROVED v4.
6. docs/process/BRIEF.md, PHASE-1.md and LEARNINGS.md.
7. docs/design/TEMPLATE-HLD.md.

R2 section keys: §1 to §5 are its numbered Facts sections; §Analysis/Execution = "Execution and agent contracts"; §Analysis/Durability = "Durability: honest recovery boundaries"; §Analysis/Self-host = "Self-host, tenancy and customization boundaries"; §Options = "Reference architectures and options analysis"; §Scope = "Three-month scope for a small agent-heavy team".

## Action Items / Open Questions

1. Run and record all seven R3 §G experiments. Owner: maintainer. Blocks: HLD-000 approval (VISION §How we will know) and every UNRUN assumption.
2. B-versus-E spike: a Paseo-class daemon and the Coder API against leases, fencing, outbound-only runners and I2. Owner: runner LLD author. Blocks: §2 build-or-reuse of runner plumbing.
3. Pilot threat review with R3 §G item 1, setting the pilot tier (UNRESOLVED). Owner: security reviewer. Blocks: §3.2 container eligibility for pilots and whether the microVM tier enters the alpha (Assumption 2).
4. Written vendor check (R3 §G item 7). Owner: maintainer. Blocks: §4 first agent pair (PRD option set 2).
5. Partner repo survey. Owner: maintainer. Blocks: §7.1 first host and FR-20 priority (PRD option set 3).
6. NFR-1 install and NFR-2 latency benchmarks on the reference runner, including Assumption 8's rootless runtime. Owner: runner LLD author. Blocks: alpha release and §10.1's install target.
7. Default retention values for run events and payloads. Owner: policy-and-gates LLD author. Blocks: NFR-10 acceptance.
8. Primary-source check of GitLab protection, stale-approval and status semantics. Owner: Git-host LLD author. Blocks: §7.1 second host (FR-20).
9. Sources for a self-hosted user-space-kernel runtime and a Wasm-class plugin runtime. Owner: execution HLD author. Blocks: reassessing §3.2 Option 4 and §9.1 Option 3.
10. Trigger.dev server licence (UNVERIFIED). Owner: maintainer. Blocks: only the §5 table cell; the verdict rests on I7.
11. Dependency-licence check and DCO in the build. Owner: maintainer. Blocks: first core release (I2, NFR-9).
12. PRD Open question 5 still frames container-default eligibility; §3.2 now decides the default tier. Owner: PRD owner. Blocks: closing PRD Open question 5 and VISION Open question 4.

## Review log
| Date | Reviewer (model) | Verdict | Summary of changes requested |
|---|---|---|---|
