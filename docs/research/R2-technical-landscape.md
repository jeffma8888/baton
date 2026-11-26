# R2 — Technical landscape for an open coding-agent conductor

**Status:** DRAFT v2 — revised against findings F1–F12 in `R2-technical-landscape.review.md`. **Observed:** 2026-11-25. Provider claims are not cross-provider benchmarks. `[Snn]` references primary URLs; architecture estimates are inferences. This builds on R1/R3.

## Facts

### 1. Workspaces: checkout separation is not execution isolation

- Git worktrees separate working directories, HEAD and indexes, but share refs, objects and default config. Linked-worktree submodule support is incomplete. Object reuse saves downloads, **not** process, credential or network isolation. [S01]
- A full `git clone` separates refs/objects, except default local hardlinks or `--shared`/`--reference` alternates; losing alternates can corrupt a shared clone. Time/disk depend on history, shallow/partial settings and transfer. A clone is not a security boundary. [S02]
- Docker containers use namespaces/cgroups/capabilities but share the kernel. Harden each workspace: rootless/user namespaces, dropped capabilities, seccomp, no daemon socket, separate mounts, quotas and egress policy. Image pulls, checkout and setup are additional to container launch time. [S03]
- Firecracker requires Linux KVM, a jailer, guest images and operator-managed volumes/network/secrets. Advertised **<125 ms boot** and **<5 MiB VMM footprint** exclude image preparation, checkout and agent startup. [S04]
**Provider contracts** (vendor startup claims, **not checkout/setup/agent-ready benchmarks**; “not published” means no number on the cited pages):

| Provider / runtime | Startup latency | Cost | Persistence | Network | Secrets |
|---|---|---|---|---|---|
| E2B / Linux VM [S05][S06][S79] | Resume ~1 s; cold number not published here | Pro $150/mo **plus** per-second usage; Hobby 1-h/Pro 24-h sessions | Pause filesystem + memory (~4 s/GiB); retained until kill | Internet default; allow/deny; domain rules have DNS/shared-endpoint caveats | HTTPS proxy injects stored keys outside VM; env vars readable by sandbox code |
| Daytona / default container; separate Linux/Windows VM [S07][S08] | Container **<90 ms** advertised, not VM/workspace-ready | $0.0504/vCPU-h + $0.0162/GiB-h RAM + $0.000108/GiB-h storage after 5 GiB free; per-second | VM pause/resume, fork, hot snapshots | Tier-dependent defaults; block/CIDR/domain controls | Outbound proxy substitutes secret references, not raw keys in sandbox |
| Modal / gVisor or VM [S10][S11][S12] | Number not published on cited pages; Ready follows initialization/probe | max(requested, actual) resources plus plan | Volumes/filesystem snapshots; 5-min default, up to 24 h, then new sandbox | Open default; block/CIDR; beta HTTPS/SNI domains | Inline secrets become environment; OIDC identity optional |
| Vercel / Firecracker [S13][S14] | “Milliseconds,” no comparable quantified ready-time | iad1 Pro $0.128/active-vCPU-h + **$0.0212/provisioned-GB-h idle too**; $0.60/M creations; storage/traffic extra | Persistent by default, auto-save, snapshots, Drives; Pro 24-h sessions resume | Outbound controls; Secure Compute/static egress | Project/team credential configuration |
| Fly / Machine VM [S15][S16][S80] | Create maybe low double-digit seconds; existing start usually <1 s | Per-second compute; stopped rootfs/volumes $0.15/GB-mo; egress extra | Ephemeral rootfs resets; attach volume | Deliberate app/firewall/private-network policy; static egress available | App vault injects env at boot on **all app Machines**; use per-trust app separation |
| Cloudflare / container microVM [S17]–[S19] | Number not published on cited pages | Active CPU; provisioned memory/disk even idle; Workers/DO extra | DO names session, not runtime files; snapshot files, not memory/processes; external storage needed | Worker mediates ingress/data; `enableInternet:false` blocks outbound | Keep credentials in mediating Worker; anything passed inside is exposed |
| Firecracker / self-operated KVM [S04] | <125 ms VM boot claim only | No hosted rate; operator compute/storage/network | Operator implements volumes/snapshots | Operator network/jailer configuration | Operator credential broker; no managed secret service |

- Conductor's consuming Vercel profile specifies **32 GB ephemeral NVMe**, not the provider's default persistence contract. [S62]
- Daytona's old `daytonaio/daytona` repository is archived, while the hosted provider docs remain live; do not assume the old OSS control plane is maintained. [S09]
- Cloudflare also offers **Dynamic Workers (JS/Python/Wasm)**, not Linux CLI sandboxes. Inactivity timeout can end an active-agent container; keep its Durable Object active or export state. [S17][S18]

**Isolation comparison — design inference from the preceding vendor contracts** ([S01]–[S19]):

| Model | Cold start / resource cost | Persistence | Network and secrets | Acceptable trust boundary |
|---|---|---|---|---|
| Linked worktree | Cheapest object reuse, checkout still required | Host disk until explicit cleanup | Inherits host network and user credentials | Cooperative same-user coding only |
| Independent clone | Fetch/copy cost and disk growth | Host disk | Same host privileges unless enclosed | Separate Git metadata, not security |
| Hardened container + checkout | Image pull/build and mount overhead; local CPU/RAM | Named volume or artifact export | Per-container network; inject scoped, short-lived secrets | Lower-cost controlled runners, not arbitrary hostile kernel code |
| KVM microVM + checkout | Provider bootstrap/VM/image/checkout; compute plus storage/egress | Explicit snapshot/Drive/volume semantics | Provider-specific egress and credential broker | Stronger untrusted-code boundary, still not a proof of zero exfiltration |

### 2. Agent driving contracts and three distinct protocols

- Claude Code offers `claude -p` with `json` or `stream-json`, a session ID for `--resume`, structured output, tool allowlists and permission modes. **Important:** non-interactive `-p` has no trust prompt and may load project hooks/MCP; `--bare` skips autodiscovery but requires API-key billing and omits hooks. The Python/TypeScript Claude Agent SDK drives the Claude Code binary and exposes structured events, permissions and hooks; the Claude *API* client SDK is not this coding-agent loop. [S20][S21]
- Claude's **self-hosted environments** are customer runners leasing an **Anthropic-hosted queue**, cloning repos and launching local Claude Code processes. Checkouts/artifacts remain customer-side, but event streams/transcripts still go to Anthropic; inference can route elsewhere without moving the control plane. This is not Cursor's hosted-loop/remote-tools split. [S75]
- Codex CLI has `codex exec` for scripted runs; the Codex TypeScript SDK manages local threads and `resumeThread`, and the current Python SDK controls a local Codex app-server over JSON-RPC, including sandbox presets and resumable threads. The app server is a richer client interface for streamed events/approvals; model selection and sandbox authorization remain explicit adapter configuration. [S23][S24]
- OpenCode has an embedded TypeScript SDK that owns a local server, sessions and event subscriptions. `opencode acp` runs a **private** server behind an ACP stdio agent; it does not attach to the shared background server. Its ACP implementation documents unsupported features, including MCP-over-ACP, so negotiate capabilities instead of assuming total parity. [S25][S26]
- Cursor CLI supports `agent -p` scripted print mode, CLI resume, `agent acp` JSON-RPC/stdin with permission requests and a programmable local/cloud SDK. Its Cloud Agent Self-Hosted Machines connect outbound while **Cursor still hosts the model loop, inference and planning**; workers execute tools and return relevant file contents, diffs and outputs. A worker is not a self-hosted control plane. [S27][S28][S29][S30]
- Gemini CLI supports headless prompts with `--output-format stream-json` and **native `gemini --acp`**, documented as JSON-RPC 2.0 over stdio with authentication, session load, prompt/cancel and approval-level configuration. [S31][S74]
- Kiro CLI supports `kiro-cli chat --v3 --no-interactive --output-format stream-json` with API key or saved login, explicit trusted tools, stable session IDs and optional local/cloud modes. Native `kiro-cli acp` provides initialize, session/new/load/prompt/cancel and notifications; ACP auth requires prior sign-in or a client-provided terminal login. Its sandbox setting may warn and continue when not required; request explicit fail-closed sandboxing for strict jobs. [S32][S33]
- Aider supports one-shot `aider --message`/`--message-file` and `--yes`; the project's own scripting guide calls its Python API **not officially supported**. Wrap the supported CLI; treat noninteractive `--yes` as unsafe unless the surrounding execution boundary is trustworthy. ACP/A2A implementation by Aider is **UNVERIFIED**. [S34]
- ACP (Agent Client Protocol, initiated by Zed) standardizes **editor/client ↔ coding-agent** session interaction: JSON-RPC, prompts, streamed messages/tool calls, permission requests and diff-oriented UX. Native: OpenCode, Cursor, Kiro and Gemini. Claude/Codex use *adapters* (`claude-agent-acp`, current `agentclientprotocol/codex-acp` built on Codex App Server); this does not establish native CLI ACP. Aider native ACP remains UNVERIFIED. ACP does not create sandbox isolation, durable task storage, model billing or repo gates. [S26][S28][S33][S35][S36][S37][S74]
- MCP (Model Context Protocol) standardizes **host/client ↔ external tools, resources and prompts**; it does not standardize coding-agent sessions or cross-agent supervision. A2A (Agent2Agent) standardizes **agent ↔ independently deployed opaque agent**, with Agent Cards, task states, messages, artifacts, polling/streaming and optional push. It does not grant a supervisor visibility into the agent's filesystem/process or guarantee branch-safe merging. All three are complementary, not competing wire formats. [S38][S39]
- Integrator-policy risk is concrete: on 2026-06-15 Conductor reported Anthropic's May 13 subscription-plan restriction for third-party Claude Agent SDK integrations had been **delayed indefinitely**, not cancelled permanently. This is the integrator's primary report, **not** independent verification of Anthropic's notice. An adapter must offer API-key/enterprise billing and not promise third-party consumer subscriptions will remain usable. [S22][S21]

### 3. Durability and orchestration for agent lifetimes

- Temporal has durable workflows and recovery, timers and activities; its single-binary development server is expressly **not** production self-hosting. Production needs Temporal Service, persistent store, namespaces, schema/upgrade care and workers; high capability and higher operations cost. [S40]
- Restate supplies durable execution and journaling; the **server** uses BSL 1.1 with a public-platform-service restriction and four-year conversion to Apache-2.0. The cited deployment guide covers Kubernetes operator/Helm, not a standalone production binary. Internal self-hosting is permitted, but bundling this server into baton's core is excluded by R3's approved licence decision (§Analysis below). [S41]
- Inngest offers a single-binary server with embedded Redis/SQLite defaults and external Redis/Postgres options; the **server** is SSPL with a three-year future Apache-2.0 grant, unlike permissive client SDKs. A single binary is not a replicated cluster. [S42][S43]
- Trigger.dev self-hosts as webapp and worker containers with Postgres/Redis; cloud-only features documented include checkpoints, warm starts and autoscaling. Docker Compose may be easy, but the additional services and cloud/self-host feature disparity matter. [S44]
- DBOS embeds durable workflows and Postgres-backed queues in the host app, with a permissive MIT TypeScript library; a Postgres server remains an operational dependency. Plain Postgres itself supports `FOR UPDATE SKIP LOCKED` for multiple consumers, but does not supply durable workflow replay, agent-step idempotency or sandbox supervision. [S45][S46]

### 4. Multi-tenant/self-host extension precedents

- Supabase self-hosting deploys **one project**: Studio's hosted multi-org/project management is absent; Postgres RLS can enforce row-level authorization but security-definer views/functions and `service_role` can bypass it. It is not per-run execution isolation. [S47][S48]
- Coolify runs self-hosted applications on operator-controlled servers, with teams, SSH hosts, REST/CLI/MCP, Docker networks, backups and an optional hosted service; its GitHub repository declares Apache-2.0. Its declarative deploy/one-command onboarding is an installation precedent, not an agent-specific governance engine. [S49]
- Plane offers Docker, Kubernetes and all-in-one self-host deployment with AGPL-3.0 Community source; deployment topology alone does not establish per-customer execution isolation (inference). [S50]
- GitLab CE is MIT and EE has a restricted licence; its official upgrade landing page confirms only an upgrade section, not package-dependency/licence-check details. Edition boundaries still require packaging decisions (inference). [S51][S52]
- Backstage has frontend/backend plugin extension points, but is a library-derived application rather than a drop-in white-label binary; upstream package bumps do **not** automatically merge customized generated templates or plugins. Avoid requiring customers to fork the UI to change branding. [S53][S54]
- Coder's self-hosted AGPL-3.0 server supports Terraform-defined workspaces on VMs/containers/Kubernetes; Community permits third-party agents, while AI audit/cost policy, multi-org RBAC and quotas live in Premium, and Coder Agents orchestration/API in AI Premium. This is a direct platform competitor as well as a possible workspace backend. [S55][S56]
- Gitpod's successor Ona supports execution in an existing customer's VPC, Dev Containers and background agent fleets, but current official docs say **new-customer signup is closed** while existing organisations remain; buyer fit and future availability should not be inferred from historical Gitpod positioning. [S57]

- A **full devbox** is an environment surface (shell/browser/editor), not a separate security primitive. Devin's brain always runs in Cognition cloud; even Customer Dedicated Deployment is a **Cognition-managed single-tenant VPC** linked to the customer, not a customer-operated instance. Contrast customer-operated Coder infrastructure and Ona's existing-customer VPC execution. [S55][S57][S78]
- Codespaces runs a GitHub-hosted **Docker dev container on a VM**, configurable through repo dev-container files and reachable through GitHub CLI. It is a possible optional workspace backend, not self-hosted compute. [S77]

### 5. Policy and live competitors

- OPA separates Rego policy **decision** from application **enforcement** and has an embeddable Go library; Cedar models authorization as principal/action/resource/context and is especially suitable for RBAC/ABAC resource checks. Neither language enforces a policy if the conductor bypasses its own gate or the subprocess has uncontrolled side channels. [S58][S59]
- Conductor is **alive**: 0.94.2 explicitly reversed its shutdown. Its local agents run directly on host **without sandboxing**, while cloud workspaces use Vercel Sandbox microVMs with an 8-vCPU/16-GB/32-GB-ephemeral profile and GitHub-only cloud repos. Its beta `/v0` API creates workspaces, sends prompts and triggers webhook routines; read-only `POST /v0/sql` queries `session_transcripts_view` for accessible workspaces. It announced joining Meta and subsequently resumed development; the cited changelogs do not establish legal ownership or where that development occurs (**UNVERIFIED**). [S60][S61][S62][S63][S64][S65]
- GitHub Agent HQ explicitly positions a multi-vendor mission control and policy/audit layer inside GitHub, with self-hosted runners available for GitHub Actions-based execution (not every cloud-sandbox mode). Its current third-party agents are Claude and Codex in **public preview**; the announced future partner list must not be misread as already generally available. The GitHub-hosted control plane is a real counterexample to “all conductors are single-vendor,” not a customer-hosted neutral control plane. [S66][S67]
- OSS competitors are **not merely offline desktop apps**: MIT T3 Code has a local server, remote/mobile clients and ACP agents; Apache-2.0 Paseo has daemon, Docker self-host, client SDK and plugin registry; MIT Orca has remote runtime/cloud VM and bots; Apache-2.0 Herdr has a Rust background server and socket API; MIT Paradev has worktrees, CLI/SDK/MCP plus paid remote/cloud features. Desktop UI alone cannot differentiate baton. [S68][S69][S70][S71][S72]

- Emdash explicitly accepts own agents **through ACP** and advertises local/customer-server/cloud execution. Its open desktop Classic gets fixes; new development moved to 2.0, so do not equate 2.0's feature set with Classic's OSS distribution. [S76]

## Analysis

### Execution and agent contracts

**Inference, confidence HIGH — Git/container boundaries differ:** one run owns one checkout and execution identity. Store `(organization, repository, task, run, workspace)`, capabilities and separate Git/process/sandbox/session IDs. Confine checkout paths; reject symlinks/traversal; retain recovery manifests. Lower-trust runners must not mount host checkouts, Docker sockets, SSH agents or shared credentials. Paths and process lifetimes are not canonical run records. [S01][S03]

**Inference, confidence HIGH — interfaces have unequal capabilities:** normalize `start`, `events(cursor)`, `send`, `approve/deny`, `cancel`, `resume`, `usage`, `artifacts`, `capabilities`, `dispose`. Record structured events, steering, approvals, resume, usage, remote execution and ACP/MCP support. Stream CLI PTY/JSONL; without a permission callback, preconfigure tools or hard-gate the sandbox—never silently upgrade authority. Store scoped auth secrets, not copied home directories. ACP is an integration format, not a platform job specification. [S20][S23][S25][S28][S32][S34][S35]

**Inference, confidence HIGH — runtime and merge authority differ:** pipeline: `authorize → reserve → checkout/sandbox → agent/events → await/resume → capture SHA → checks → human review → PR → code-host merge → cleanup`. Bind checks/approvals to the **candidate SHA** and invalidate on change. Require branch protection/status checks; runners cannot self-approve or merge. Runtime tool permission and change approval are separate gates: ACP transports only the former. [S35][S60][S66][S73]

**Inference, confidence MED — metering may be incomplete:** separate monetary reservations per run/repo/org, runner-enforced time/CPU/RAM/storage ceilings and rate/concurrency quotas. Debit usage atomically, warn, stop new calls at supported caps and reconcile invoices. Opaque subscription billing/coarse metrics cannot guarantee an absolute USD hard cap; say so. Idle RAM and snapshots defeat token-only budgets. [S14][S16][S19][S20][S23]

**Inference, confidence HIGH — gates need reconstructable evidence:** append actor/tenant/run, sequence, policy hash, action/decision/reason, candidate SHA, time, provider IDs and redaction status. Optional prompt/tool payloads need separate encryption, access and retention controls: outputs can echo masked secrets. Export JSON; add tamper-evident chains/signatures later. A mutable activity feed is not an audit log. Deny privilege transitions if policy evaluation or audit commit fails. [S48][S58][S59][S62]

**Inference, confidence MED — compatibility demand is unknown:** start with tested YAML/JSON rules for repo/agent/sandbox, network, paths/tools, runtime/spend and SHA-pinned review/CI. Cedar suits RBAC; OPA arbitrary inputs. Choose an engine when customers need it. Cedar may skip erroneous policies: inspect diagnostics and deny critical gates on errors in the *application*, not merely trust the evaluator. [S58][S59]

### Durability: honest recovery boundaries

**Inference, confidence HIGH — database transactions coordinate claims:** atomically append event, update run state and enqueue the next stage. Claim with row locks/`SKIP LOCKED`; heartbeat leases with fencing epochs. A reconciler inspects recorded sandbox/process IDs and event cursors. Side effects remain **at least once**: use provider idempotency tokens or tagged lookup, and branch/PR keys. Ambiguous lost processes become `needs_reconciliation`, not a second checkout writer; webhooks alone are insufficient. [S46][S05][S14][S18]

**Inference, confidence MED:** a single binary *without* Postgres is possible as an explicitly single-user local mode using SQLite and one active writer, but a team server in Docker Compose should require a real Postgres service for concurrent claims, locks, backups and migration. That is a “single product binary plus database,” **not** literally an infrastructure-free single-binary deployment. Postgres-only is a workload-specific stepping stone; if many cross-service timers, human waits, compensation and replay become central, reassess Temporal or DBOS. Under R3's governing decision, bundling Restate's BSL server or Inngest's SSPL server in the Apache-2.0 core is **[BLOCKING]** for that bundled variant; neither may be a hard dependency. At most, support optional operator-installed backends, without denying their permitted internal self-hosting. See [R3 §B](R3-business-case.md#b-open-core--self-hosted-playbook). [S40]–[S46]

### Self-host, tenancy and customization boundaries

**Inference, confidence MED — trust spans multiple layers:** start **single-org, multiple teams/repos/users**, not hostile-customer multitenancy. Store `org_id`; enforce API/query membership and test RLS/service-role bypasses. Later customer tenancy requires separate credentials/runner pools, network, volumes and payload access. Adapter config must not forge tenants or expose cross-org events. Team roles alone are not isolation. [S07][S47][S48]

**Inference, confidence MED — generated-app upgrades are costly:** version schemas for branding, adapters, providers, gates and signed webhooks. Later operator-installed trusted plugins are distinct from compiled-in adapters; untrusted tenant plugins must not run inside the server. SemVer extension APIs, offer migration dry-runs, pin releases, back up DB/secret references and publish rollback compatibility. Backstage warns against requiring UI forks. [S53][S54]

**Inference, confidence MED:** the open core must cover end-to-end `task → run → diff → gate → PR` plus at least two adapters, one local and one container backend, single-org roles and auditable events. Otherwise a claimed open conductor cannot perform its own defining workflow without a paid module. **Governing decision: [R3 §B](R3-business-case.md#b-open-core--self-hosted-playbook), approved v4, selects Apache-2.0 core plus a commercial, source-available `enterprise/` module. FSL/BSL/ELv2/SSPL are [BLOCKING] for the core**, not reopened here. Dependency checks must preserve that decision; separate API integration with operator-installed Coder is not embedding its AGPL code in the core. [S50][S51][S55][S56][S41][S43]

### Reference architectures and options analysis

Compare the **first three-month release** against [VISION](../../VISION.md): “expose it to your teams” (L21), “team-shared, policy-gated agent fleets” (L27–28), “agent-agnostic by contract” (L41) and “self-host first, hosted later” (L47). A shared durable queue is a **derived design requirement**, not VISION's explicit invariant. VISION L43–44 makes the sandbox optional; checkout separation alone cannot promise safe untrusted execution.

**Cross-option requirements:** every option must preserve agent-neutral interfaces and distinguish checkout separation from sandbox isolation; label unsandboxed worktrees `trusted-local` and offer a sandboxed backend for lower-trust work. Failures are **[ADVISORY] shared implementation risks for A–E**, never a selective BLOCKING con; adding a sandbox to A changes its defined host-only scope. R3 §B rules out any **[BLOCKING] bundled Restate-BSL/Inngest-SSPL** core variant; A–E do not bundle those servers. [S01][S03][S41][S43]

**A. Local-first daemon + UI; host worktrees, no shared server.**

- Pros: simplest local setup; object reuse; T3 Code/Paseo demonstrate daemon clients. [S01][S68][S69]
- Cons: [BLOCKING] no shared server cannot supply the selected team control plane; [ADVISORY] lower-trust sandbox and agent neutrality are shared implementation risks, unsatisfied in A as defined; [ADVISORY] crowded OSS UX. [S01][S68]–[S72]
- Mitigation: retain the daemon as a B/C runner client; adding a team server changes A's definition.
- **Why not preferred:** eliminated by its team-server BLOCKING con; local client ergonomics remain useful.

**B. Team server + self-hosted runners; local container/worktree executors; Postgres.**

- Pros: owns shared queue/gates; outbound runners and Compose deployment; bounded first slice. [S03][S46]
- Cons: [ADVISORY] fixed executors postpone external sandbox/Coder integration; [ADVISORY] neutrality and sandbox enforcement remain shared implementation risks; [ADVISORY] runners and database add operations. **No inherent BLOCKING con.** [S03][S55]
- Mitigation: define neutral interfaces now, implement container and trusted worktree first.
- **Why preferred (MED):** smallest deployable team server; sequencing judgment, not a blocking elimination of C/E.

**C. Neutral control plane + independently pluggable agent/sandbox providers; customer-hosted server and runner.**

- Pros: B fits inside this boundary; customer-owned Coder, GitHub-hosted Codespaces or VM backends retain one gate/audit model. [S05]–[S19][S55][S77]
- Cons: [ADVISORY] neutrality and sandbox enforcement remain shared implementation risks; [ADVISORY] provider lifecycle/billing edge cases; [ADVISORY] general plug-in ABI and certified provider matrix exceed a quarter; [ADVISORY] vendor model calls may still leave customer infrastructure. **No inherent BLOCKING con.** [S07][S10][S30]
- Mitigation: ship two agents/two executors, not every provider.
- **Why preferred (MED):** desired long-term boundaries without requiring the entire integration matrix initially.

**D. Vendor-hosted control plane; customer-side runner only.**

- Pros: little customer control-plane operation; Cursor/Claude demonstrate customer-side execution. [S30][S75]
- Cons: [BLOCKING] central software is not customer-hostable, violating “self-host first”; [ADVISORY] customer compute does not keep transcripts/model inputs local; [ADVISORY] neutrality and sandbox enforcement remain shared implementation risks, not structurally impossible. [S30][S75]
- Mitigation: offer hosted mode after the deployable core, not instead of it.
- **Why not preferred:** eliminated for the first release; later hosted mode can reuse C.

**E. Extend an existing OSS server/platform rather than build all runner plumbing.**

- Pros: Paseo's Apache daemon/SDK/plugins offer reuse; an **out-of-process Coder API integration** can use customer workspaces without importing its AGPL source. [S55][S69]
- Cons: [ADVISORY] neutrality and sandbox enforcement remain shared implementation risks; [ADVISORY] forking Paseo risks upstream divergence; [ADVISORY] task/gate/merge durability still needs building; [ADVISORY] Coder APIs and paid governance overlap require boundary discipline. The distinct **E-embedded-Coder** variant has [BLOCKING] embedding AGPL source in an Apache-only core; separate API integration survives. [R3 §B](R3-business-case.md#b-open-core--self-hosted-playbook). [S55][S56][S69]
- Mitigation: test SDK/API integration without a fork or bundled Coder server; keep baton-owned gates and audit.
- **Why not preferred initially (MED):** integration dependencies could dominate the bounded alpha; E remains viable and should be spiked against B during HLD, not eliminated by judgment.

| Criterion | A: daemon | B: bounded server | C: neutral platform | D: hosted+runner | E: extend/API |
|---|---|---|---|---|---|
| Customer-hosted team plane | No | Yes | Yes | No | Yes |
| Shared durable gate queue | No | Postgres | Backend contract | Vendor-hosted | Add baton layer |
| Execution | Host only | Container/worktree | Provider contract | Customer runner | Existing platform |
| Agent neutrality | Shared requirement | Shared requirement | Shared requirement | Vendor-dependent, advisory | Adapter-dependent, advisory |
| Quarter risk (judgment) | Low | Medium | Medium B-first; high full matrix | Medium plus hosting | Integration-dependent |
| BLOCKING, as defined | Team plane | None | None | Self-host first | None; embedded-AGPL variant excluded |

**Verdict from BLOCKING items alone:** eliminate **A standalone** and **D first**; **B, C and E remain viable**. This leaves the original A–D verdict unchanged. E-embedded-Coder and bundled-BSL/SSPL variants fail separately. Blocking items cannot rank survivors. **Advisory preference (MED): B first, within C's boundaries** to bound the alpha; compare E in HLD. Audience 1 pays for the control plane, Audience 2 uses it, and Audience 3 embedding remains unproven. [R3 §§A/B/D](R3-business-case.md#analysis).

### Three-month scope for a small agent-heavy team

**Estimate, confidence LOW — author inference, no source:** **two experienced engineers plus a part-time designer/security reviewer**, assisted by agents, could ship a 12-week **alpha** after approved HLD/LLD/plan. This is not a benchmark; security, vendor terms and procurement remain critical path. Assume one runner OS/architecture, Postgres, at most two Git hosts, no public multitenancy or compliance claims.

- **Weeks 1–2:** approved HLD/LLD/plan and threat model; design-partner interviews per R3; benchmark cold checkout and sandbox-ready latency, gather repo sizes and secrets-network requirements. Design review must precede code per VISION; the estimate slips if it does not finish. [S01]–[S19]
- **Weeks 3–5:** single-org Compose install with server/Postgres, one outbound runner, worktree `trusted-local` and hardened container backend, run records, crash recovery, cancellation and ordered event streaming. Verify runner death, duplicate dispatch and orphaned sandbox recovery with forced-failure tests. [S01][S03][S46]
- **Weeks 6–8:** two production-path adapters (Claude CLI with API key and Codex CLI/API key, or one open harness plus one proprietary agent if policy review stalls); provider capability flags and golden JSONL/event fixtures; human permission UI; hard time/concurrency quotas and best-effort monetary tracking. [S20]–[S26]
- **Weeks 9–12:** diff/commit/PR flow, SHA-pinned human gate and CI status, one small auditable policy schema, API/CLI, docs, recovery tests, backup/restore and one real pilot. First external sandbox-provider spike (e.g. Vercel or E2B) is **optional** and does not become a GA claim merely because one demo works. [S05][S13][S14][S46]

**Not a credible 12-week promise (MED, judgment):** complete SSO/SCIM, secure multi-org external tenancy, eight certified sandbox providers, every CLI's deep interactive parity, a stable public plugin marketplace, white-label B2B2B metering, formal audit/compliance claims or guaranteed dollar caps for subscription agents. Those are later milestones that require buyer evidence and explicit threat reviews. [S20]–[S39][S47]–[S57]

## Recommendations

- **HIGH —** Give **Audience 1 (platform/DevEx)** a self-hosted team control plane serving **Audience 2 (engineering teams)**; A/D alone fail the team/self-host requirements. Postgres is the MED substrate preference below. [S46][S68][S69]
- **HIGH —** Name checkout separation and sandbox isolation as different capabilities; worktrees expose host processes and credentials. [S01][S03][S60]
- **HIGH —** Make gate checks SHA-bound and fail closed before merge, with execution identities unable to self-approve; ACP permissions are not merge policy. [S35][S66][S73]
- **HIGH —** Do not depend on consumer subscriptions as the sole auth path; the reported third-party Claude SDK restriction was delayed, not permanently rescinded. [S21][S22]
- **MED —** Introduce provider-neutral agent/sandbox interfaces but implement only a bounded two-agent/two-executor slice in 90 days; eight vendor lifecycles exceed the credible alpha. [S05]–[S39]
- **MED —** Prefer Postgres queue/outbox/reconciler over a new workflow cluster initially; Temporal's dev server is not a production single-binary substitute. [S40][S46]
- **MED —** For **Audience 1**, ship single-org roles, declarative policy and structured audit in the open core; this is the R1/R3 differentiation against paid governance and highly competitive OSS clients. [S55][S56][S68]–[S72]
- **LOW —** Defer **Audience 3 (B2B2B)** tenancy, white-label and billing until buyer validation: R3 §A finds weak demand; hosted/self-hosted tenancy differs. [R3 §A](R3-business-case.md#analysis). [S47]

## Open questions

- What workload mix and trust tier do pilot users need: cooperative team work, hostile untrusted PR code, private registry access, or air-gapped local models? Their answer chooses hardened-container versus VM default. [S03][S04][S10]
- What precise external key/consumer-plan terms will Anthropic, OpenAI, Cursor and Kiro allow for third-party self-hosted orchestration after the delayed subscription-policy change? Written vendor confirmation is required. [S22][S30][S32]
- Which two Git hosts and two agents must the first design partners actually support? Conductor's cloud GitHub-only limit is a candidate gap, not proof buyers will switch. [S61][S66]
- Does “onboard in an hour” include Docker daemon privileges, Postgres provisioning, IdP and secrets setup or only a first local run? Benchmark warm and cold ready-to-prompt separately. [S03][S40][S46]
- Which enterprise data must **never** leave the customer network, given vendor CLIs and customer-side workers still export inference inputs or tool results? [S21][S30]
- Is there any signed partner asking to embed the full platform for **its own end customers**? Until then keep **Audience 3 (B2B2B)** tenant isolation and white-label design latent, not shipped. [S47][S57]
- How will the dependency compatibility check and `enterprise/` packaging preserve R3's approved Apache-2.0 core and source-available module split? Apply R3's DCO-without-CLA handoff; this is implementation, not a new licence choice. [R3 §B and Handoffs](R3-business-case.md#handoffs-to-other-docs). [S41][S43]

## Handoffs to other docs

- **R1:** update OSS-desktop wording where it implies desktop-only: T3 Code, Paseo, Orca, Herdr and Paradev now explicitly document server/remote/mobile or programmatic control. Preserve Conductor's active status. [S64][S68]–[S72]
- **R3:** model *idle provisioned RAM and stopped storage* as well as token charges; Daytona defaults to containers not microVMs; its old GitHub repo is archived while the hosted provider docs remain live. Avoid equating open-source workflow **SDK** licences with Restate/Inngest **server** licences. [S07][S09][S13][S14][S16][S41][S43]
- **HLD-000 provider contract:** include Coder, Codespaces, E2B, Daytona and Vercel behind one lifecycle/artifact/egress/secrets capability interface; keep hosted-only compute optional and core task/gate/audit customer-hostable. Claude/Cursor runners are execution patterns, not neutral control-plane substitutes. [S05][S07][S13][S30][S55][S75][S77]
- **HLD-000:** decide C's provider-neutral boundaries versus B's bounded implementation; define explicit threat tiers, tenant/runner credential boundaries, app-provider reconciliation, event and gate states, policy failure modes, source-host merge authority, migration/rollback and dependency-licence compatibility checks under R3's decision. Keep options analysis BLOCKING verdict separate from advisory schedule preference. [S01][S03][S35][S40]–[S46][S58][S59]

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
- [S37] Current Codex App Server ACP adapter — https://github.com/agentclientprotocol/codex-acp
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

- [S74] Gemini CLI, native ACP mode — https://geminicli.com/docs/cli/acp-mode/
- [S75] Claude Code, self-hosted runner versus hosted control plane — https://code.claude.com/docs/en/self-hosted-environments
- [S76] Emdash, ACP and Classic/2.0 boundaries — https://emdash.sh
- [S77] GitHub Codespaces, Docker-on-VM and CLI access — https://docs.github.com/en/codespaces/overview
- [S78] Devin, vendor brain and dedicated Devbox deployment — https://docs.devin.ai/enterprise/vpc/overview
- [S79] E2B, proxy secrets versus readable environment — https://docs.e2b.dev/secrets.md ; https://docs.e2b.dev/sandbox/environment-variables.md
- [S80] Fly, app-wide vault secrets injected into Machines — https://fly.io/docs/apps/secrets/

## Review log

2026-11-25: different-model review of DRAFT v1 returned **REVISE**; DRAFT v2 revisions below await re-review. Findings refer to `R2-technical-landscape.review.md`.

| Finding | Tag | Change / disposition | Section touched |
|---|---|---|---|
| F1 | BLOCKING | R3's approved licence split governs; excluded bundled BSL/SSPL servers and hard dependencies; narrowed open question to packaging. | Durability; customization; questions; HLD handoff |
| F2 | BLOCKING | Added a why-preferred/not-preferred line per architecture. | Options A–E |
| F3 | BLOCKING | Tagged shared neutrality/sandbox enforcement risks ADVISORY for A–E; removed B's conditional blocker and recalculated verdict: A/D out, B/C/E viable. | Options; matrix; verdict |
| F4 | ADVISORY | Quoted VISION; called queue derived, not explicit; downgraded A's sandbox deficit. | Option scope; A |
| F5 | ADVISORY | Added E: Paseo reuse or separate Coder API integration; distinguished excluded embedded-AGPL variant. | Options; verdict |
| F6 | ADVISORY | Read named agent directory and Gemini's ACP docs; replaced native-ACP uncertainty. | Agents; Sources S74 |
| F7 | ADVISORY | Read stale repo migration notice and current repo; replaced adapter citation. | ACP; Sources S37 |
| F8 | ADVISORY | Removed Kiro permission claim; limited Restate to Kubernetes operator/Helm; removed Plane edition-licensing assertion; softened Conductor location and GitLab upgrade detail. | Agent, durability, precedent and competitor Facts |
| F9 | ADVISORY | Added Claude runner boundary, Emdash ACP, SQL transcript view, full-devbox topology and Codespaces abstraction. | Facts; provider handoff |
| F10 | ADVISORY | Replaced provider prose with seven rows covering latency, cost, persistence, network and secrets; missing latency numbers labelled. | Workspace Facts |
| F11 | ADVISORY | Applied VISION numbering: 1 platform/DevEx, 2 teams, 3 B2B2B. | Verdict; recommendations; questions |
| F12 | ADVISORY | Removed vendor citations from staffing/timing estimate; labelled author inference without source. | Three-month estimate |

## Learnings / mistakes

- Daytona default is a container, not a microVM; caught in its runtime docs. Rule: classify the actual default. [S07]
- Modal offers gVisor and VM; caught in runtime docs. Rule: compare runtime classes. [S10]
- Vercel defaults to persistence but Conductor configures ephemeral NVMe; caught by comparing both docs. Rule: separate provider contracts from consuming profiles. [S13][S62]
- Ona signup is closed to new customers; caught in current docs. Rule: check availability. [S57]
- Stale Restate/DBOS/Cedar/Codex doc routes returned 404; replaced with fetched pages. Rule: never cite a plausible but unfetched route. [S23][S41][S45][S59]
- Restate/Inngest server terms differ from SDK terms; caught in server LICENSE files. Rule: inspect packaged component licences. [S41][S43]
- General Copilot page did not support third-party availability; replaced with dedicated preview page. Rule: cite the precise availability claim. [S67]
- Revision: I first retained a Restate single-node wording unsupported by the cited deployment page; reviewer evidence prompted the Kubernetes-only correction. Rule: do not embellish deployment topology. [S41]
- Revision: a bulk source-fetch script called `AKI_BIN web_fetch` rather than `AKI_BIN ext web_fetch`; all calls rejected, then corrected and re-run. Rule: test the CLI route on one URL before batching.
