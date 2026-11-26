# R2 — Technical landscape for an open coding-agent conductor

**Observed:** 2026-11-25. Provider claims are not cross-provider benchmarks. `[Snn]` references primary URLs below; architecture estimates are inferences. This builds on R1's live-competitor correction and R3's self-hosted team-server wedge.

## Facts

### 1. Workspaces: checkout separation is not execution isolation

- Git worktrees keep separate working directories, HEAD and indexes while sharing the underlying repository, refs, objects and (by default) config. Git says linked-worktree submodule support is incomplete. Setup avoids a second object download, but a worktree does **not** isolate processes, credentials, ports or the host network. [S01]
- A full `git clone` owns independent refs and objects, unless the local clone uses default hardlinks or explicit `--shared`/`--reference`; Git warns an alternate repository disappearing can corrupt a shared clone. Clone time and disk depend on repository history, shallow/partial clone settings and transfer; neither clone nor worktree creates a security boundary. [S02]
- Docker containers use namespaces, cgroups and capabilities, but share the host kernel; the daemon and mounted socket are particularly sensitive privileges. A per-workspace container can add process, filesystem, resource and network isolation, but needs hardening: rootless/user namespaces, dropped capabilities, seccomp, no daemon socket, separate mounts, quotas and an egress policy. Build/image pulls and checkout/setup are not included in container launch time. [S03]
- Firecracker is a KVM microVM monitor with a jailer; its advertised **<125 ms boot** and **<5 MiB VMM memory footprint** do not include image preparation, workspace checkout, network setup or agent startup. It needs a Linux KVM host, guest images, volume/snapshot handling and an operator to implement lifecycle and secrets. [S04]
- E2B exposes isolated Linux-VM sandboxes, pause/resume and memory-plus-filesystem persistence; a resumed VM is claimed around **1 second**, while pause takes about **4 seconds per GiB RAM**. Its defaults include public-internet egress, with allow/deny controls; Hobby permits 1-hour continuous sessions, Pro 24 hours and $150/month **plus** per-second usage, enterprise multi-day persistence and BYOC. Preserve checkout state before kill; persisted paused instances need explicit cleanup. [S05][S06]
- Daytona's default Linux sandbox is a **container**, not a microVM; a separate Linux/Windows VM class offers pause/resume, fork and hot snapshots. It offers per-sandbox block/CIDR/domain network policy subject to account tier, and secrets mediated by an outbound proxy so their raw value stays out of the sandbox. Advertised milliseconds are **not** a coding-workspace-ready benchmark. Published base rates: $0.0504/vCPU-hour, $0.0162/GiB-hour RAM and $0.000108/GiB-hour storage after the first free 5 GiB, metered per second; the `daytonaio/daytona` GitHub repository is archived despite a currently live provider service, so avoid assuming its old OSS control plane is maintained. [S07][S08][S09]
- Modal Sandboxes have **gVisor** and **VM/KVM** runtime choices; they are not universally microVMs. Named images, volumes, filesystem snapshots and inline secrets are supported; default maximum lifetime is five minutes, configurable up to 24 hours, with snapshot-to-new-sandbox needed beyond that. Outbound is open by default, with block/CIDR and beta HTTPS/SNI-domain restrictions. Billing is on requested or actual resource usage, whichever is greater, plus applicable plan terms; no comparable end-to-end startup figure was established. [S10][S11][S12]
- Vercel Sandbox uses Firecracker microVMs and now documents default **persistent sandboxes** with auto-save, snapshots and Drives; access controls cover outbound network and project/team credentials. Published iad1 Pro rates include $0.128/active-vCPU-hour, **$0.0212/provisioned-GB-hour even while idle**, $0.60/million creations, snapshot and Drive storage and traffic; Pro sessions max out at 24 hours but persistent sessions may be resumed. Conductor's deployed configuration separately describes 32 GB **ephemeral** NVMe: provider capability must not be mistaken for each customer's persistence contract. [S13][S14][S62]
- Fly Machines expose API-managed VMs. Vendor docs distinguish *creation* (possibly low double-digit seconds including image/rootfs work) from restart of an existing stopped Machine (usually well below one second). Default ephemeral storage is lost on restart; attach a volume to retain workspace data. Started compute is per-second; a stopped rootfs and attached volumes still accrue published $0.15/GB-month storage charges, and network egress is a separate meter. Isolation and egress control require deliberate app/firewall/private-network configuration; a fast VM restart is not a fast first workspace. [S15][S16]
- Cloudflare's current Sandboxes offer **container microVMs** for full Linux and **Dynamic Workers** for restricted JS/Python/Wasm; they are not interchangeable execution environments for arbitrary CLI agents. A Worker mediates data, APIs and internet access; container `enableInternet: false` can deny outbound traffic. A Durable Object names the session but does **not** retain its running filesystem: stopped instances lose files/processes unless snapshot or external R2 is used. Snapshot saves filesystem, not process/memory; an idle container still bills provisioned memory/disk, alongside Workers/Durable Objects and active CPU. Inactivity timeout can stop even a running agent unless the application keeps the Durable Object active. [S17][S18][S19]

**Isolation comparison — design inference from the preceding vendor contracts** ([S01]–[S19]):

| Model | Cold start / resource cost | Persistence | Network and secrets | Acceptable trust boundary |
|---|---|---|---|---|
| Linked worktree | Cheapest object reuse, checkout still required | Host disk until explicit cleanup | Inherits host network and user credentials | Cooperative same-user coding only |
| Independent clone | Fetch/copy cost and disk growth | Host disk | Same host privileges unless enclosed | Separate Git metadata, not security |
| Hardened container + checkout | Image pull/build and mount overhead; local CPU/RAM | Named volume or artifact export | Per-container network; inject scoped, short-lived secrets | Lower-cost controlled runners, not arbitrary hostile kernel code |
| KVM microVM + checkout | Provider bootstrap/VM/image/checkout; compute plus storage/egress | Explicit snapshot/Drive/volume semantics | Provider-specific egress and credential broker | Stronger untrusted-code boundary, still not a proof of zero exfiltration |

### 2. Agent driving contracts and three distinct protocols

- Claude Code offers `claude -p` with `json` or `stream-json`, a session ID for `--resume`, structured output, tool allowlists and permission modes. **Important:** non-interactive `-p` has no trust prompt and may load project hooks/MCP; `--bare` skips autodiscovery but requires API-key billing and omits hooks. The Python/TypeScript Claude Agent SDK drives the Claude Code binary and exposes structured events, permissions and hooks; the Claude *API* client SDK is not this coding-agent loop. [S20][S21]
- Codex CLI has `codex exec` for scripted runs; the Codex TypeScript SDK manages local threads and `resumeThread`, and the current Python SDK controls a local Codex app-server over JSON-RPC, including sandbox presets and resumable threads. The app server is a richer client interface for streamed events/approvals; model selection and sandbox authorization remain explicit adapter configuration. [S23][S24]
- OpenCode has an embedded TypeScript SDK that owns a local server, sessions/worktrees and event subscriptions. `opencode acp` runs a **private** server behind an ACP stdio agent; it does not attach to the shared background server. Its ACP implementation documents unsupported features, including MCP-over-ACP, so negotiate capabilities instead of assuming total parity. [S25][S26]
- Cursor CLI supports `agent -p` scripted print mode, CLI resume, `agent acp` JSON-RPC/stdin with permission requests and a programmable local/cloud SDK. Its Cloud Agent Self-Hosted Machines are workers making outbound connections while **Cursor still hosts the model loop, inference and planning**; customer workers execute tools and send relevant file contents, diffs and outputs back. Self-hosting that worker is not a self-hosted control plane. [S27][S28][S29][S30]
- Gemini CLI supports headless prompts and `--output-format stream-json` JSONL events. Do not claim Gemini offers native ACP based solely on a CLI docs index; **native support UNVERIFIED** in this run. A CLI adapter works without ACP. [S31]
- Kiro CLI supports `kiro-cli chat --v3 --no-interactive --output-format stream-json` with API key or saved login, explicit trusted tools, stable session IDs and optional local/cloud modes. Native `kiro-cli acp` provides initialize, session/new/load/prompt/cancel, permission exchanges and notifications; ACP auth requires prior sign-in or a client-provided terminal login. Its sandbox setting may warn and continue when not required; request explicit fail-closed sandboxing for strict jobs. [S32][S33]
- Aider supports one-shot `aider --message`/`--message-file` and `--yes`; the project's own scripting guide calls its Python API **not officially supported**. Wrap the supported CLI; treat noninteractive `--yes` as unsafe unless the surrounding execution boundary is trustworthy. ACP/A2A implementation by Aider is **UNVERIFIED**. [S34]
- ACP (Agent Client Protocol, initiated by Zed) standardizes **editor/client ↔ coding-agent** session interaction: JSON-RPC, prompts, streamed messages/tool calls, permission requests and diff-oriented UX. Native: OpenCode, Cursor and Kiro. Claude and Codex have maintained *adapters* (`claude-agent-acp`, `codex-acp`), not a basis to assert native CLI ACP. Gemini and Aider native ACP are UNVERIFIED. ACP does not create sandbox isolation, durable task storage, model billing or repo gates. [S26][S28][S33][S35][S36][S37]
- MCP (Model Context Protocol) standardizes **host/client ↔ external tools, resources and prompts**; it does not standardize coding-agent sessions or cross-agent supervision. A2A (Agent2Agent) standardizes **agent ↔ independently deployed opaque agent**, with Agent Cards, task states, messages, artifacts, polling/streaming and optional push. It does not grant a supervisor visibility into the agent's filesystem/process or guarantee branch-safe merging. All three are complementary, not competing wire formats. [S38][S39]
- Integrator-policy risk is concrete: on 2026-06-15 Conductor reported Anthropic's May 13 subscription-plan restriction for third-party Claude Agent SDK integrations had been **delayed indefinitely**, not cancelled permanently. This is the integrator's primary report, **not** independent verification of Anthropic's notice. An adapter must offer API-key/enterprise billing and not promise third-party consumer subscriptions will remain usable. [S22][S21]

### 3. Durability and orchestration for agent lifetimes

- Temporal has durable workflows and recovery, timers and activities; its single-binary development server is expressly **not** production self-hosting. Production needs Temporal Service, persistent store, namespaces, schema/upgrade care and workers; high capability and higher operations cost. [S40]
- Restate supplies durable execution, journaling and single-node/self-hosted deployment paths, but the Restate **server** source uses Business Source License 1.1 with a public-platform-service restriction and four-year conversion to Apache-2.0; a public third-party managed product needs license review even if internal installations are permitted. Its server deployment guidance includes single-node and Kubernetes operator/Helm routes. [S41]
- Inngest now offers a single-binary self-host server with embedded Redis/SQLite by default and external Redis/Postgres options for reliability/scale; the **server** is SSPL with a three-year future Apache-2.0 grant, a distinct constraint from permissive client SDKs. Single binary does not imply equivalent production durability to a replicated cluster. [S42][S43]
- Trigger.dev self-hosts as webapp and worker containers with Postgres/Redis; cloud-only features documented include checkpoints, warm starts and autoscaling. Docker Compose may be easy, but the additional services and cloud/self-host feature disparity matter. [S44]
- DBOS embeds durable workflows and Postgres-backed queues in the host app, with a permissive MIT TypeScript library; a Postgres server remains an operational dependency. Plain Postgres itself supports `FOR UPDATE SKIP LOCKED` for multiple consumers, but does not supply durable workflow replay, agent-step idempotency or sandbox supervision. [S45][S46]

### 4. Multi-tenant/self-host extension precedents

- Supabase self-hosting deploys **one project**: Studio's hosted multi-org/project management is absent; Postgres RLS can enforce row-level authorization but security-definer views/functions and `service_role` can bypass it. It is not per-run execution isolation. [S47][S48]
- Coolify runs self-hosted applications on operator-controlled servers, with teams, SSH hosts, REST/CLI/MCP, Docker networks, backups and an optional hosted service; its GitHub repository declares Apache-2.0. Its declarative deploy/one-command onboarding is an installation precedent, not an agent-specific governance engine. [S49]
- Plane offers Docker, Kubernetes and all-in-one self-host deployment; its AGPL-3.0 Community source and commercial/enterprise editions with workspace or instance licensing show that a workspace-scoped licence is not the same as an independently isolated customer tenant. [S50]
- GitLab CE's source licence is MIT and EE's is restricted/proprietary; Self-Managed's published upgrade path, package dependencies and licence checks illustrate how edition boundaries coexist with one product but add migration and packaging responsibilities. [S51][S52]
- Backstage has frontend/backend plugin extension points, but is a library-derived application rather than a drop-in white-label binary; upstream package bumps do **not** automatically merge customized generated templates or plugins. Avoid requiring customers to fork the UI to change branding. [S53][S54]
- Coder's self-hosted AGPL-3.0 server supports Terraform-defined workspaces on VMs/containers/Kubernetes; Community permits third-party agents, while AI audit/cost policy, multi-org RBAC and quotas live in Premium, and Coder Agents orchestration/API in AI Premium. This is a direct platform competitor as well as a possible workspace backend. [S55][S56]
- Gitpod's successor Ona supports execution in an existing customer's VPC, Dev Containers and background agent fleets, but current official docs say **new-customer signup is closed** while existing organisations remain; buyer fit and future availability should not be inferred from historical Gitpod positioning. [S57]

### 5. Policy and live competitors

- OPA separates Rego policy **decision** from application **enforcement** and has an embeddable Go library; Cedar models authorization as principal/action/resource/context and is especially suitable for RBAC/ABAC resource checks. Neither language enforces a policy if the conductor bypasses its own gate or the subprocess has uncontrolled side channels. [S58][S59]
- Conductor is **alive**: 0.94.2 explicitly reversed its shutdown. Its local agents run directly on host **without sandboxing**, while cloud workspaces use Vercel Sandbox microVMs with an 8-vCPU/16-GB/32-GB-ephemeral profile and GitHub-only cloud repos. Its beta `/v0` API creates workspaces, sends prompts, reads status/transcripts and triggers webhook routines. It announced joining Meta and its team says it continued development there; precise legal ownership structure is **UNVERIFIED**, so “Meta-owned” is shorthand for this publicly stated relationship, not a verified filing. [S60][S61][S62][S63][S64][S65]
- GitHub Agent HQ explicitly positions a multi-vendor mission control and policy/audit layer inside GitHub, with self-hosted runners available for GitHub Actions-based execution (not every cloud-sandbox mode). Its current third-party agents are Claude and Codex in **public preview**; the announced future partner list must not be misread as already generally available. The GitHub-hosted control plane is a real counterexample to “all conductors are single-vendor,” not a customer-hosted neutral control plane. [S66][S67]
- OSS competitors are **not merely offline desktop apps**: MIT T3 Code has a local server, remote/mobile clients and ACP agents; Apache-2.0 Paseo has daemon, Docker self-host, client SDK and plugin registry; MIT Orca has remote runtime/cloud VM and bots; Apache-2.0 Herdr has a Rust background server and socket API; MIT Paradev has worktrees, CLI/SDK/MCP plus paid remote/cloud features. Desktop UI alone cannot differentiate baton. [S68][S69][S70][S71][S72]

## Analysis

### Execution and agent contracts

**Inference, confidence HIGH:** one run must own exactly one checkout and one execution identity. The conductor stores a stable `(organization, repository, task, run, workspace)` relationship and a provider-neutral capability snapshot. Worktree paths are not arbitrary user input; choose a confined root, reject symlinks/path traversal, and retain a manifest for recovery. Do not mount the source checkout, Docker socket, host SSH agent or shared model credentials into a lower-trust runner. Git refs, process IDs, sandbox IDs and provider session IDs are *separate* identifiers; store all and do not use a path or CLI process lifetime as the canonical run record. Git and container limitations underpin the distinction. [S01][S03]

**Inference, confidence HIGH:** normalize `start(prompt, cwd, identity, budget, permissions)`, `events(cursor)`, `send`, `approve/deny`, `cancel`, `resume`, `usage`, `artifacts`, `capabilities` and `dispose` across adapters. A capabilities record states whether an agent supports structured tool events, mid-turn steering, prompts needing approval, native resume, reliable token/cost usage, remote execution, ACP and MCP. For a CLI-only agent, stream the PTY or JSONL; if it lacks a permission callback, **do not** silently upgrade its authority—preconfigure allowed tools or run in a hard-gated sandbox. Treat vendor CLI auth tokens as user-supplied secrets and never copy their whole home directory. ACP is a useful first integration format, not a sufficient platform-level *job* specification. [S20][S23][S25][S28][S32][S34][S35]

**Inference, confidence HIGH:** the review/gate pipeline is `submit → authorize → reserve budget → create checkout → provision sandbox → start agent → append event → await input / resume → capture commit/diff → run checks → human review → publish PR → merge via code host → cleanup`. A check runs on the **immutable candidate SHA**; if the branch changes, invalidate approvals. Require code-host branch protection and status checks even if the local UI says approved, and do not let runner credentials merge or approve their own PR. Tool permission is a runtime action gate; review approval is a *change* gate; these must not share a Boolean. ACP permissions provide a transport for runtime approvals but do not enforce repository branch protection. [S35][S60][S66][S73]

**Inference, confidence MED:** budget limits have three independent quantities: a monetary reservation per run/repo/org (estimated token cost where the adapter reports it), a wall-clock/CPU/memory/storage ceiling enforced by the runner/provider, and rate/concurrency quotas. On incremental usage events, atomically debit or raise a soft warning, stop new tool/model calls at the hard cap, and reconcile against vendor/provider invoices later; external provider costs can arrive late and vendor CLI usage may be unavailable. An absolute USD hard guarantee is **not supported** when an adapter has only opaque subscription billing or coarse metrics. State this honestly in the UI. Per-sandbox cost tables show why idle RAM and snapshots can defeat a token-only cap. [S14][S16][S19][S20][S23]

**Inference, confidence HIGH:** a gate evaluation and its immutable inputs belong in an append-only audit trail: actor, tenant, run, event sequence, policy version/hash, action, decision, reason, resource SHA, timestamps, provider IDs and redaction status. Keep prompt/tool-output payloads optional, separately encrypted, access-controlled and retention-bounded; secrets can be echoed into chat/build logs even when a provider masks secret values in configuration. Ship JSON export and verifiable sequence/hash-chain or signed export later; a mutable UI activity feed alone is not an audit log. Fail closed when policy or audit commit fails before a privilege transition. [S48][S58][S59][S62]

**Inference, confidence MED:** ship a narrow declarative YAML/JSON rule set first: permitted repo/agent/sandbox combinations, network mode, path/tool allowlists, maximum runtime and spend, required human reviewer plus passing CI on the candidate SHA. Compile and test policy before activating it. Cedar suits principal-action-resource RBAC; OPA/Rego suits cross-cutting arbitrary input, but choose one only after a real customer requires policy-as-code compatibility. Cedar's evaluator can skip an individual policy on error, so inspect diagnostics and make the *application* deny critical gates on evaluation error; neither engine is an automatic fail-closed gateway. [S58][S59]

### Durability: honest recovery boundaries

**Inference, confidence HIGH:** the smallest defensible implementation is Postgres plus a transactional outbox/queue and periodic reconciler. In one transaction, insert the run event, enqueue the next stage and update state; workers claim rows using row locks/`SKIP LOCKED`, lease with a heartbeat and fencing epoch, and release or expire leases. Agent and provider side effects are **at least once** under crash/retry. Creation therefore uses an idempotency token if supported, otherwise a provider tag and reconciliation lookup; publishing a PR uses an idempotent branch/PR key. Before retrying a lost process, inspect its sandbox and event cursor; if ambiguous, mark `needs_reconciliation` rather than starting a second writer on the same checkout. The DB must record the active sandbox/process ID, not assume a webhook was delivered. [S46][S05][S14][S18]

**Inference, confidence MED:** a single binary *without* Postgres is possible as an explicitly single-user local mode using SQLite and one active writer, but a team server in Docker Compose should require a real Postgres service for concurrent claims, locks, backups and migration. That is a “single product binary plus database,” **not** literally an infrastructure-free single-binary deployment. Postgres-only is a workload-specific stepping stone; if many cross-service timers, human waits, compensation and replay become central, reassess Temporal or DBOS. Restate/Inngest licences make automatic embedding or a white-label managed workflow offering a legal review item, **not** proof that internal users may not self-host them. [S40]–[S46]

### Self-host, tenancy and customization boundaries

**Inference, confidence MED:** begin as a **single-organization instance with multiple teams/repos and users**, not untrusted-customer multitenancy. Store `org_id` on every business row now, enforce membership at API/query boundaries, add DB RLS as defense in depth only when tests cover service-role bypasses, and use separate per-tenant credentials/runner pools if a multi-customer host is eventually built. A team role is not automatically an isolation boundary: resources include Git credentials, agent auth, network reachability, filesystem volumes and event payloads. A tenant must not be able to forge `org_id` via an arbitrary adapter config or see cross-org audit events. Supabase and Daytona illustrate different layers of “tenant isolation.” [S07][S47][S48]

**Inference, confidence MED:** customization should mean versioned configuration schemas for theme/brand strings, adapter manifests, sandbox-provider manifests, policy gates and webhook integrations; trusted Go/TypeScript plugin packages can be a later operator-installed extension. An untrusted tenant-provided plugin running in the server process violates the tenant boundary. Distinguish a *process plugin*, an out-of-process webhook with signed payloads, and a compiled-in adapter. SemVer the plugin API; expose a migration/dry-run command, pin image and binary versions, back up DB and secret references, and publish an upgrade/rollback matrix. Backstage's generated-app upgrade friction is a warning against “just fork our UI.” [S53][S54]

**Inference, confidence MED:** the open core must cover end-to-end `task → run → diff → gate → PR` plus at least two adapters, one local and one container backend, single-org roles and auditable events. Otherwise a claimed open conductor cannot perform its own defining workflow without a paid module. R3's Apache-2.0 core/commercial Enterprise suggestion is a **licensing decision pending legal review**, not implied by Coder/Plane/GitLab's differing AGPL/MIT splits. Avoid shipping proprietary Coder components or copyleft workflow-engine sources inside the open binary without an explicit compatibility assessment. [S50][S51][S55][S56][S41][S43]

### Reference architectures and options analysis

Scope of comparison: **one small team's first three-month release**, against VISION's hard commitments: an open, self-hostable core; a shared multi-user server with a persistent queue; agent-neutral contracts; and a real isolation choice. A `[BLOCKING]` con below means the *option as defined* violates one of these commitments, not that the technology can never be evolved into a different option. Infrastructure advice and time estimates are overrulable `[ADVISORY]` judgments.

**A. Local-first daemon + web UI; host worktrees, no optional server.**

- Pros: lowest time to first local run; worktree reuse; a daemon/CLI/mobile client path proven by T3 Code and Paseo. Local filesystem work remains inspectable and cheap. [S01][S68][S69]
- Cons: [BLOCKING] as defined, a per-user daemon with no shared server cannot provide VISION's self-hosted **team control plane** or shared durable approval queue; [BLOCKING] if host-only worktrees are the sole backend, it cannot safely advertise sandbox isolation for untrusted execution; [ADVISORY] competing OSS tools already dominate this UX. [S01][S68]–[S72]
- Mitigation: make the daemon a **runner client of B or C**, and a worktree an explicitly “trusted-local” mode; this changes the option, rather than removing the blocking defect in A.

**B. Team server + self-hosted runner agents, fixed local-container backend; Postgres.**

- Pros: single-org self-hosting, queues and shared gates have clear ownership; a runner can claim a job on outbound connection and create a constrained container next to its checkout; Docker Compose is tractable in a quarter. [S03][S46]
- Cons: [BLOCKING] **if** the only agent interface remains hardcoded to one vendor or the only checkout remains unsandboxed, this violates agent-neutrality/isolation choice; otherwise **none** inherent to this option. [ADVISORY] a fixed container executor delays VM/cloud-provider and customer-owned Coder integration; [ADVISORY] runner and database operations cost more than a single daemon. [S03][S55]
- Mitigation: specify agent and sandbox *interfaces* now, implement container and trusted worktree first, and phase in remote providers. Under this definition B is viable; the conditional blocking errors are requirements the implementation must avoid.

**C. Neutral control plane + independently pluggable agent and sandbox providers, with self-hosted server and runner.**

- Pros: clean ownership separation; B can be its initial implementation; enterprises can attach Coder, VM providers or own runners while retaining one task/gate/audit model. Neutrality is meaningful when egress/secrets and permissions are provider capabilities, not vague marketing labels. [S05]–[S19][S55]
- Cons: no inherent BLOCKING con under the stated commitments. [ADVISORY] every provider is a lifecycle, snapshot and billing edge-case; [ADVISORY] fully general plugin ABI and security certification exceed one quarter; [ADVISORY] some agent vendors still operate their own hosted model loops, so customer-run **control plane** does not imply local inference. [S07][S10][S13][S17][S30]
- Mitigation: define interfaces, build **one** real team-server/runner deployment and two agents; treat cloud provider plug-ins as an interface test or narrow optional pilot, not a promised eight-provider GA matrix.

**D. Hosted managed control plane with customer-side runner only.**

- Pros: deployment friction is low for a hosted buyer; customer-side execution is a proven pattern with Cursor. [S30]
- Cons: [BLOCKING] the central control plane is not self-hostable, contrary to VISION's initial hard commitment; [ADVISORY] customer still exports tool outputs to the vendor and cannot guarantee local data residency merely by supplying compute. [S30]
- Mitigation: offer hosted mode only **after** C ships as deployable customer software.

| Criterion | A: daemon only | B: fixed server+runner | C: neutral control plane | D: hosted+runner |
|---|---|---|---|---|
| Customer-hosted shared control plane | **No** | Yes | Yes | **No** |
| Durable shared review queue | No | Yes | Yes | Vendor-hosted |
| Isolation choice | No (host only) | Local container/worktree | Contract + multiple backends | Customer runner backend |
| Cross-agent viability | Could have | Yes if designed | Yes by contract | Vendor dependent |
| Three-month implementation risk | Low | Medium | Medium with B-first slice; high if all integrations | Medium plus hosting |
| Blocking under VISION as defined | Yes, two | None if requirements met | None | Yes, one |

**Verdict from BLOCKING items alone:** eliminate **A as a standalone product** and **D as first product**; **B and C remain viable**. The blocking analysis **does not distinguish B from C**, so it cannot prove C superior. **Advisory preference (MED, overrulable):** implement B as the first deployable *vertical slice* inside C's provider-neutral boundaries. Select C as a bounded reference architecture, not a mandate to ship every cloud sandbox and protocol in three months. This is consistent with R1's policy/audit gap and R3's open team-server-first recommendation; the cloud B2B2B case remains unproven. [S55][S66][S68]–[S72]

### Three-month scope for a small agent-heavy team

**Estimate, confidence LOW:** with **two experienced engineers plus one part-time designer/security reviewer** and agents assisting implementation, an honest 12-week **alpha** is plausible if prerequisites in the repo's design process (approved HLD, LLD and plan) finish before implementation. This is an engineering estimate, **not** a vendor-published benchmark; security review, vendor terms and enterprise procurement remain critical path. Assume one supported OS/architecture for runners, one Postgres instance, two code hosts at most, no untrusted public multitenancy, and no regulated compliance claims. [S03][S46][S55]

- **Weeks 1–2:** approved HLD/LLD/plan and threat model; design-partner interviews per R3; benchmark cold checkout and sandbox-ready latency, gather repo sizes and secrets-network requirements. Design review must precede code per VISION; the estimate slips if it does not finish. [S01]–[S19]
- **Weeks 3–5:** single-org Compose install with server/Postgres, one outbound runner, worktree `trusted-local` and hardened container backend, run records, crash recovery, cancellation and ordered event streaming. Verify runner death, duplicate dispatch and orphaned sandbox recovery with forced-failure tests. [S01][S03][S46]
- **Weeks 6–8:** two production-path adapters (Claude CLI with API key and Codex CLI/API key, or one open harness plus one proprietary agent if policy review stalls); provider capability flags and golden JSONL/event fixtures; human permission UI; hard time/concurrency quotas and best-effort monetary tracking. [S20]–[S26]
- **Weeks 9–12:** diff/commit/PR flow, SHA-pinned human gate and CI status, one small auditable policy schema, API/CLI, docs, recovery tests, backup/restore and one real pilot. First external sandbox-provider spike (e.g. Vercel or E2B) is **optional** and does not become a GA claim merely because one demo works. [S05][S13][S14][S46]

**Not a credible 12-week promise (MED, judgment):** complete SSO/SCIM, secure multi-org external tenancy, eight certified sandbox providers, every CLI's deep interactive parity, a stable public plugin marketplace, white-label B2B2B metering, formal audit/compliance claims or guaranteed dollar caps for subscription agents. Those are later milestones that require buyer evidence and explicit threat reviews. [S20]–[S39][S47]–[S57]

## Recommendations

- **HIGH —** Use a self-hosted team control plane with a runner and Postgres; A/D alone violate the declared self-host/team commitments. [S46][S68][S69]
- **HIGH —** Name checkout separation and sandbox isolation as different capabilities; worktrees expose host processes and credentials. [S01][S03][S60]
- **HIGH —** Make gate checks SHA-bound and fail closed before merge, with execution identities unable to self-approve; ACP permissions are not merge policy. [S35][S66][S73]
- **HIGH —** Do not depend on consumer subscriptions as the sole auth path; the reported third-party Claude SDK restriction was delayed, not permanently rescinded. [S21][S22]
- **MED —** Introduce provider-neutral agent/sandbox interfaces but implement only a bounded two-agent/two-executor slice in 90 days; eight vendor lifecycles exceed the credible alpha. [S05]–[S39]
- **MED —** Prefer Postgres queue/outbox/reconciler over a new workflow cluster initially; Temporal's dev server is not a production single-binary substitute. [S40][S46]
- **MED —** Ship single-org roles, declarative policy and structured audit in the open core; this is the R1/R3 differentiation against paid governance and highly competitive OSS clients. [S55][S56][S68]–[S72]
- **LOW —** Add customer-tenant white-label extension and per-tenant billing only after buyer validation; the current B2B2B demand evidence is weak and Supabase shows hosted/self-hosted tenancy need not match. [S47][S50]

## Open questions

- What workload mix and trust tier do pilot users need: cooperative team work, hostile untrusted PR code, private registry access, or air-gapped local models? Their answer chooses hardened-container versus VM default. [S03][S04][S10]
- What precise external key/consumer-plan terms will Anthropic, OpenAI, Cursor and Kiro allow for third-party self-hosted orchestration after the delayed subscription-policy change? Written vendor confirmation is required. [S22][S30][S32]
- Which two Git hosts and two agents must the first design partners actually support? Conductor's cloud GitHub-only limit is a candidate gap, not proof buyers will switch. [S61][S66]
- Does “onboard in an hour” include Docker daemon privileges, Postgres provisioning, IdP and secrets setup or only a first local run? Benchmark warm and cold ready-to-prompt separately. [S03][S40][S46]
- Which enterprise data must **never** leave the customer network, given vendor CLIs and customer-side workers still export inference inputs or tool results? [S21][S30]
- Is there any signed partner asking to embed the full platform for **its own end customers**? Until then keep B2B2B tenant isolation and white-label design latent, not shipped. [S47][S57]
- What source licence and contribution terms will the maintainer adopt, given Apache/MIT/AGPL precedents and Restate/Inngest workflow-server limitations? Legal review is necessary before bundling. [S41][S43][S49]–[S56]

## Handoffs to other docs

- **R1:** update OSS-desktop wording where it implies desktop-only: T3 Code, Paseo, Orca, Herdr and Paradev now explicitly document server/remote/mobile or programmatic control. Preserve Conductor's active status. [S64][S68]–[S72]
- **R3:** model *idle provisioned RAM and stopped storage* as well as token charges; Daytona defaults to containers not microVMs; its old GitHub repo is archived while the hosted provider docs remain live. Avoid equating open-source workflow **SDK** licences with Restate/Inngest **server** licences. [S07][S09][S13][S14][S16][S41][S43]
- **HLD-000:** decide C's provider-neutral boundaries versus B's bounded implementation; define explicit threat tiers, tenant/runner credential boundaries, app-provider reconciliation, event and gate states, policy failure modes, source-host merge authority, migration/rollback and licence review. Keep options analysis BLOCKING verdict separate from advisory schedule preference. [S01][S03][S35][S40]–[S46][S58][S59]

## Sources

- [S01] Git, linked worktrees — https://git-scm.com/docs/git-worktree
- [S02] Git, clone (local hardlinks and alternates) — https://git-scm.com/docs/git-clone
- [S03] Docker, engine security — https://docs.docker.com/engine/security/
- [S04] Firecracker project, jailer, boot and VMM footprint — https://firecracker-microvm.github.io/
- [S05] E2B, sandbox persistence and network — https://docs.e2b.dev/sandbox/persistence.md ; https://docs.e2b.dev/network/internet-access.md
- [S06] E2B, plans and per-second pricing — https://e2b.dev/pricing
- [S07] Daytona, runtime classes and isolation/secrets — https://www.daytona.io/docs/en/sandboxes.md ; https://www.daytona.io/docs/en/isolation.md
- [S08] Daytona, billing — https://www.daytona.io/pricing
- [S09] Daytona, archived repository metadata — https://api.github.com/repos/daytonaio/daytona
- [S10] Modal, Sandbox lifecycle, runtimes and storage — https://modal.com/docs/guide/sandbox
- [S11] Modal, network policy — https://modal.com/docs/guide/sandbox-networking
- [S12] Modal, pricing and plans — https://modal.com/pricing
- [S13] Vercel, Sandbox capabilities — https://vercel.com/docs/vercel-sandbox
- [S14] Vercel, Sandbox active CPU/idle memory/storage pricing — https://vercel.com/docs/vercel-sandbox/pricing
- [S15] Fly, Machines creation versus restart and volumes — https://fly.io/docs/machines/overview/
- [S16] Fly, pricing — https://fly.io/docs/about/pricing/
- [S17] Cloudflare, two Sandbox execution environments — https://developers.cloudflare.com/sandbox/
- [S18] Cloudflare, lifetimes, snapshots and idle risks — https://developers.cloudflare.com/sandbox/concepts/lifetime/
- [S19] Cloudflare, container pricing — https://developers.cloudflare.com/containers/pricing/
- [S20] Claude Code, `-p`/JSON/permissions/resume — https://code.claude.com/docs/en/headless
- [S21] Anthropic, Claude Agent SDK overview and terms — https://platform.claude.com/docs/en/agent-sdk/overview
- [S22] Conductor's first-party account of Anthropic's delayed third-party plan update — https://www.conductor.build/blog/claude-subscription-update
- [S23] OpenAI, Codex noninteractive execution — https://developers.openai.com/codex/noninteractive
- [S24] OpenAI, Codex TypeScript/Python SDK and app-server — https://developers.openai.com/codex/sdk
- [S25] OpenCode, SDK and session API — https://opencode.ai/docs/sdk/
- [S26] OpenCode, ACP integration and limits — https://opencode.ai/docs/acp/
- [S27] Cursor, CLI print mode/resume — https://cursor.com/docs/cli/overview.md
- [S28] Cursor, native CLI ACP — https://cursor.com/docs/cli/acp.md
- [S29] Cursor, TypeScript SDK local/cloud `Agent.create`, `Run` — https://cursor.com/docs/sdk/typescript.md
- [S30] Cursor, customer-side workers versus hosted model loop — https://cursor.com/docs/cloud-agent/self-hosted
- [S31] Gemini CLI, headless output — https://geminicli.com/docs/cli/headless/
- [S32] Kiro CLI, headless/API-key/stream-json — https://kiro.dev/docs/cli/headless/
- [S33] Kiro CLI, ACP — https://kiro.dev/docs/cli/acp/
- [S34] Aider, documented CLI/Python scripting caveat — https://aider.chat/docs/scripting.html
- [S35] Agent Client Protocol, editor/agent interface — https://agentclientprotocol.com/overview/introduction
- [S36] ACP adapter for Claude Agent SDK — https://github.com/agentclientprotocol/claude-agent-acp
- [S37] ACP adapter for Codex — https://github.com/zed-industries/codex-acp
- [S38] Model Context Protocol specification — https://modelcontextprotocol.io/specification/2025-11-25
- [S39] Agent2Agent specification — https://a2a-protocol.org/latest/specification/
- [S40] Temporal, production self-hosted guide versus development server — https://docs.temporal.io/self-hosted-guide
- [S41] Restate, current server BSL and deploy routes — https://raw.githubusercontent.com/restatedev/restate/main/LICENSE ; https://docs.restate.dev/server/deploy
- [S42] Inngest, self-host architecture and single-binary defaults — https://www.inngest.com/docs/self-hosting
- [S43] Inngest, server licence and future grant — https://raw.githubusercontent.com/inngest/inngest/main/LICENSE.md
- [S44] Trigger.dev, Compose/worker and Cloud/self-host gaps — https://trigger.dev/docs/self-hosting/overview
- [S45] DBOS, embedded durable queues and library licence — https://docs.dbos.dev/ ; https://raw.githubusercontent.com/dbos-inc/dbos-transact-ts/main/LICENSE
- [S46] PostgreSQL, locking/`SKIP LOCKED` queue updates — https://www.postgresql.org/docs/current/sql-update.html ; https://www.postgresql.org/docs/current/sql-select.html
- [S47] Supabase, self-hosted one-project limitation — https://supabase.com/docs/guides/self-hosting
- [S48] Supabase, row-level security bypass caveats — https://supabase.com/docs/guides/database/postgres/row-level-security
- [S49] Coolify, teams/deploy/API and Apache metadata — https://github.com/coollabsio/coolify ; https://api.github.com/repos/coollabsio/coolify
- [S50] Plane, Community AGPL and installation — https://github.com/makeplane/plane ; https://developers.plane.so/self-hosting/
- [S51] GitLab, CE MIT versus EE restrictions — https://docs.gitlab.com/development/licensing/
- [S52] GitLab, upgrading Self-Managed instances — https://docs.gitlab.com/update/
- [S53] Backstage, new/legacy plugin architecture — https://backstage.io/docs/plugins/
- [S54] Backstage, upgrades and generated-template drift — https://backstage.io/docs/getting-started/keeping-backstage-updated/
- [S55] Coder, self-hosted workspaces and control plane — https://github.com/coder/coder ; https://api.github.com/repos/coder/coder
- [S56] Coder, Community/Premium/AI Premium feature matrix — https://coder.com/pricing
- [S57] Ona, customer-VPC execution and signup status — https://ona.com/docs
- [S58] Open Policy Agent/Rego decisions and enforcement — https://www.openpolicyagent.org/docs/latest/
- [S59] Cedar, PARC and default-deny/forbid semantics — https://docs.cedarpolicy.com/auth/authorization.html
- [S60] Conductor, local agents not sandboxed — https://www.conductor.build/docs/faq
- [S61] Conductor, Vercel Sandbox GitHub-only cloud — https://www.conductor.build/docs/cloud/faq
- [S62] Conductor, configured microVM resources and secrets warning — https://www.conductor.build/docs/cloud/cloud-computer
- [S63] Conductor, beta Cloud API `/v0` — https://docs.conductor.build/api
- [S64] Conductor 0.94.2, reversed shutdown — https://www.conductor.build/changelog/0.94.2-conductor-is-back
- [S65] Conductor 0.93.0, joining Meta; ownership structure UNVERIFIED — https://www.conductor.build/changelog/0.93.0-conductor-is-joining-meta
- [S66] GitHub, Agent HQ announcement — https://github.blog/news-insights/company-news/welcome-home-agents/
- [S67] GitHub, currently documented third-party agents (public preview) — https://docs.github.com/en/copilot/concepts/agents/about-third-party-agents
- [S68] T3 Code, remote server/ACP and MIT licence — https://github.com/pingdotgg/t3code
- [S69] Paseo, daemon/SDK/plugins/Apache-2.0 — https://github.com/getpaseo/paseo
- [S70] Orca, remote/runtime/plugins/MIT — https://github.com/stablyai/orca
- [S71] Herdr, Rust daemon/socket/Apache-2.0 — https://github.com/herdrdev/herdr
- [S72] Paradev, worktrees/CLI/cloud/MIT — https://github.com/ParadevAI/Paradev
- [S73] GitHub, protected branches, required reviews, stale approvals and status checks — https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/managing-protected-branches/about-protected-branches

## Review log

| Date | Reviewer | Verdict | Changes |
|---|---|---|---|
| Pending | Different-model reviewer | Pending | Initial draft awaiting independent review |

## Learnings / mistakes

- Initially grouped Daytona under the requested “microVM sandboxes”; its current docs say its **default is a container**, with VM as a separate class. Rule: classify the default execution boundary from the provider's actual runtime docs. [S07]
- Initially treated Modal Sandbox as a microVM-only option; its docs also offer gVisor. Rule: compare products at runtime-class granularity, not by category label. [S10]
- Initially treated Vercel/Conductor's “ephemeral storage” as one contract; Vercel now documents default persistence while Conductor's own profile says ephemeral NVMe. Rule: distinguish provider capability from a consuming application's configuration. [S13][S62]
- Initially assumed Gitpod/Ona remained a normal signup option; Ona's current docs say new-customer signup is closed. Rule: check current availability before recommending a precedent as a deployable alternative. [S57]
- Initially tried stale documentation URLs for Restate deployment, DBOS internals, Cedar authorization and Codex noninteractive CLI (404); replaced claims with verified live primary URLs. Rule: never cite an un-fetched route merely because it looks plausible. [S23][S41][S45][S59]
- Initially presumed server-licence terms from permissive SDK narratives; verified Restate server BSL and Inngest server SSPL/future grant. Rule: inspect the exact packaged server's LICENSE before architecture or white-label recommendations. [S41][S43]
- Initially used GitHub's general Copilot page as the citation for current third-party availability; checked and replaced it with the dedicated third-party-agent page, which states Claude/Codex public preview. Rule: cite the page that directly supports a current availability claim. [S67]
