# Review — R2 technical landscape (DRAFT v1)

status: COMPLETE

- Target: `docs/research/R2-technical-landscape.md` (46,756 B, 264 lines, ~5,970 words, 73 source IDs S01–S73 / 81 URLs).
- Reviewer: Opus · review R2 (cross-model; R2 was written by a different model). Date: 2026-11-25.
- Inputs read in full: `docs/process/BRIEF.md`, `VISION.md`, R2; consistency targets R1 (`R1-market-landscape.md`) and APPROVED R3 v4 (`R3-business-case.md`); original R2 task statement.
- Source integrity: **73/73 source IDs fetched (81/81 URLs)**; 72 readable, 1 stub (S52). Result: 63 supports, 9 partly, 0 mismatch, 1 unreadable.

## Verdict

**REVISE**: 3 BLOCKING findings. R2 reopens R3's approved licence conclusion (F1). No option has a "Why preferred" line (F2). A conditional con that applies to every option is tagged BLOCKING on one option only, and the agent-neutrality invariant is tagged inconsistently (F3). All three are bounded text fixes. None of them changes R2's BLOCKING-derived architecture verdict (eliminate A and D; B and C remain viable; B is built first, inside C's boundaries), and the source base is sound.

## Findings

**F1 [BLOCKING] — Licence conclusion contradicts approved R3.**
- Where: Analysis › Self-host, tenancy and customization, L95; Durability, L87; Open questions, L166; Handoffs › HLD-000, L172.
- Evidence: R3 v4 (APPROVED) eliminates FSL and BSL-with-competing-use, plus ELv2, under the customer invariant, and SSPL under the open invariant (R3 L96–L97, L100). It recommends an **Apache-2.0 core plus a commercial, source-available `enterprise/` module** (R3 L100, L193). R2 conflicts with this in three places. L95 calls it "R3's Apache-2.0 core/commercial Enterprise *suggestion*", "a licensing decision pending legal review". L166 asks again "What source licence … will the maintainer adopt, given Apache/MIT/AGPL precedents". L87 treats bundling the BSL-1.1 Restate server or the SSPL Inngest server as "a legal review item". R2 never says that FSL/BSL/ELv2/SSPL are BLOCKING for the core.
- Fix:
  - (a) L95: replace the sentence with "Per R3 (approved): Apache-2.0 core plus a source-available `enterprise/` module; FSL/BSL/ELv2/SSPL are BLOCKING for the core."
  - (b) L87: add "Consequence: the BSL-1.1 Restate server and the SSPL Inngest server must not be bundled into the Apache-2.0 core or be a hard dependency of it. They can at most be optional, operator-installed backends behind the durability interface, which fits the Postgres-first choice."
  - (c) L166: narrow the question to contribution terms (DCO vs CLA) and the `enterprise/` boundary, and drop the licence re-litigation.
  - (d) L172: change "licence review" to "dependency-licence compatibility check".

**F2 [BLOCKING] — No "Why preferred" for any option.**
- Where: Reference architectures A–D, L101–L123. Each option has only Pros / Cons / Mitigation.
- Evidence: the review standard requires Pros, tagged Cons and "Why preferred" for every option. R2's preference reasoning appears only once, in the verdict paragraph (L134).
- Fix: add one "Why preferred / why not" line to each option, about 4 lines in total:
  - A: not preferred; eliminated by BLOCKING; kept only as a runner client.
  - B: preferred as the first deployable slice; it is the smallest team server.
  - C: preferred as the boundary; it carries provider neutrality.
  - D: not preferred; eliminated; possible later as a hosted mode.

**F3 [BLOCKING] — Conditional BLOCKING con, and inconsistent invariant tags.**
- Where:
  - B cons, L110: "[BLOCKING] **if** the only agent interface remains hardcoded to one vendor or the only checkout remains unsandboxed … otherwise **none**".
  - Table L132, B: "None if requirements met".
  - D cons, L122, compared with table L130, D: "Vendor dependent".
- Evidence: BRIEF rule 4 defines BLOCKING as "cannot work — a violated invariant". B's con depends on an implementation mistake that any of A, C or D could equally make. It is therefore shared by all options, so it cannot be BLOCKING, yet R2 tags it on B alone. The same agent-neutrality invariant is conditional-BLOCKING in B, untagged in A, and missing from D's cons, even though the table marks D "Vendor dependent".
- Fix:
  - Move B's conditional into a single cross-option line: "Requirements for every option: agent interface not hardcoded; a sandboxed backend available".
  - Keep only B's two [ADVISORY] cons.
  - Set table L132 for B to "None".
  - Add an agent-neutrality con to D with an explicit tag (ADVISORY unless R2 can show D cannot be agent-neutral).
  - State that the verdict is unchanged.

**F4 [ADVISORY] — The scope line overstates VISION's "hard commitments".**
- Where: L99, which lists "a shared multi-user server with a persistent queue" and "a real isolation choice".
- Evidence: VISION has no "hard commitments" list and does not mention a persistent queue. The nearest text:
  - L20: "bring your … sandboxes"
  - L21: "expose it to your teams"
  - L27–28: "team-shared, policy-gated agent fleets"
  - L43–44: "one workspace = one isolated checkout (+ optional sandbox)"
  - L47: "a single-binary / single-compose install is the product"

  A's first BLOCKING con (no shared team server, L104) still holds on L21 and L27–28. A's second BLOCKING con (host-only worktrees) is weaker, because VISION L43 makes the sandbox *optional*.
- Fix: quote the VISION lines in L99. Recast "persistent queue" as a requirement derived from those lines, not an invariant. Ground A's second con in VISION L20 or downgrade it to ADVISORY; A stays eliminated by the first con either way.

**F5 [ADVISORY] — The option set never evaluates "extend an existing OSS server or platform".**
- Evidence: BRIEF rule 4 says "list ALL viable options". R2 itself shows viable substrates: Paseo (Apache-2.0 daemon, Docker self-host, SDK, plugins; L65, S69) and Coder (AGPL server, Terraform workspaces; L57, S55). It also presents GitHub Agent HQ with self-hosted runners as the governed alternative (L64). No option covers building baton's gate/audit/policy layer on top of one of these.
- Fix: add option E with tagged cons. For example: embedding Coder's AGPL code in the core is BLOCKING under R3's Apache core; forking Paseo is ADVISORY (upstream divergence). Alternatively, state in one line why this option is out of scope.

**F6 [ADVISORY] — Gemini native ACP is left UNVERIFIED although one fetch settles it.**
- Where: L35, L38.
- Evidence: the ACP agents page (https://agentclientprotocol.com/overview/agents) lists "Gemini CLI" (link: github.com/google-gemini/gemini-cli), alongside Codex CLI, Cursor, Kiro CLI, OpenCode, Claude Agent and GitHub Copilot. Aider is not listed.
- Fix: cite Gemini CLI's own ACP documentation as the primary source and update L35/L38. Keep Aider UNVERIFIED.

**F7 [ADVISORY] — S37 cites a stale repository.**
- Where: L38, L212.
- Evidence: https://github.com/zed-industries/codex-acp says "Development has moved to agentclientprotocol/codex-acp" (the adapter is built on the Codex App Server).
- Fix: cite https://github.com/agentclientprotocol/codex-acp.

**F8 [ADVISORY] — Five claims are not on the exact page cited.** None is an outright mismatch.
- (a) L36, Kiro "permission exchanges": the S33 page lists initialize, session/new, load, prompt, cancel, session/notification and `_kiro.dev/*` extensions. The word "permission" does not appear on it.
- (b) L45, Restate "single-node … routes": the S41 deploy page shows only the Kubernetes operator and Helm.
- (c) L54, Plane "commercial/enterprise editions with workspace or instance licensing": neither S50 page says this. They show only AGPL-3.0, deploy methods and instance admin.
- (d) L63, "its team says it continued development there": S64 says only "Conductor isn't shutting down. It's back, and we're building again", and S65 says "Conductor is joining Meta".
- (e) L55, GitLab "published upgrade path, package dependencies and licence checks": S52 returned only the stub "Upgrade a GitLab instance to a later version", twice.
- Fix: cite the page that actually states each claim, or soften the claim to what the cited page says.

**F9 [ADVISORY] — Some R1/R3 handoff items are not addressed.**
- R1 L274–278 asks R2 for four items that are missing or thin:
  - Claude runners as a customer-side execution pattern: only Cursor is covered (L34).
  - Emdash ACP adoption: absent.
  - Conductor's SQL-over-transcripts design: S63 documents read-only `POST /v0/sql` over `session_transcripts_view`, but L63 says only "reads status/transcripts".
  - Full-devbox isolation (Devin/Coder/Ona): covered only in part.
- R3 L211 asks R2 to abstract Coder, Codespaces, E2B, Daytona and Vercel Sandbox as providers; Codespaces is absent.
- Fix: add one sentence per item, or list them as explicitly deferred under Handoffs.

**F10 [ADVISORY] — The per-provider isolation matrix is incomplete against the R2 task.**
- Evidence: the task asks for latency, cost, persistence, network and secrets for each of the seven providers. The table at L22–27 is per isolation *model*, not per provider. Gaps in the prose:
  - secrets: E2B (L13) and Fly (L17);
  - startup latency: Vercel (L16), Cloudflare (L18) and Modal (L15, "no comparable figure").
- Fix: add a 7-row provider table, with "not published" where the vendor gives no number.

**F11 [ADVISORY] — Audience numbering is not used.**
- Evidence: R2 never names Audience 1 (platform/DevEx) or Audience 3 (B2B2B). It says "single-organization instance", "customer tenants" and "B2B2B" (L91, L134, L156, L165). This does not contradict R3 (L81, L83).
- Fix: label L149/L155 as Audience 1 and L156/L165 as Audience 3, for traceability.

**F12 [ADVISORY] — The estimate cites sources that cannot support it.**
- Where: L138 cites [S03][S46][S55] for a team-size / 12-week estimate. Those pages cannot support it, and the text already calls it an "engineering estimate".
- Fix: drop the citations or mark the line "inference, no source".

**Checks that passed:**
- Consistency with R1 and R3:
  - Conductor-is-alive (L63; S64 confirmed) matches R1 L219 and R3.
  - Agent HQ facts (L64: multi-vendor mission control announced 2025-10-28; Claude/Codex third-party agents in public preview; self-hosted runners for Actions-based execution; GitHub-hosted control plane) match R1 L154 and L176, and R3 L58 and L111.
  - B2B2B kept latent (L156, L165) matches R3's §A critique.
  - The single-org open core (L91, L95) is a subset of R3 L102.
- Completeness:
  - All seven task coverage areas are present: isolation (7 providers plus worktree/clone/container/Firecracker), drivers (7 agents plus ACP/MCP/A2A plus vendor-policy risk), durability (6 options), precedents (7), policy, and live competitors (8).
  - 4 reference architectures (≥3 required), a comparison table, a verdict explicitly stated "from BLOCKING items alone" (L134), and a 12-week estimate.
  - Recommendations, Open questions, Handoffs (R1/R3/HLD-000), Sources (73, all cited in the body), Review log stub and Learnings are all present.
  - About 5,970 words, inside the 3,000–6,000 target.
- Rule 5 provenance scan: **PASS**.
  - No private paths (`/Users/`, `~/`, `/home/`, `Library/`).
  - No `@` characters at all, so no email addresses or handles.
  - No personal names: every capitalised word pair is a product or term name.
  - No employer reference. GitHub owners in source URLs are organisations.

## Source spot-check table

| # | Source id | URL | Claim in R2 | What the page says | Verdict |
|---|---|---|---|---|---|
| 1 | S01 | https://git-scm.com/docs/git-worktree | Worktrees share repo/refs/objects/config; submodule support incomplete; no process isolation | Shared repository and config by default; "support for submodules is incomplete"; move/remove caveats | supports |
| 2 | S02 | https://git-scm.com/docs/git-clone | Local clone hardlinks; --shared/--reference; alternate loss can corrupt | Hardlinks by default for local clones; --shared alternates; corruption warning | supports |
| 3 | S03 | https://docs.docker.com/engine/security/ | Namespaces, cgroups, capabilities; shared kernel; daemon/socket sensitive | Namespaces/cgroups/capabilities; daemon needs root unless rootless; socket risk | supports |
| 4 | S04 | https://firecracker-microvm.github.io/ | KVM microVM, jailer, <125 ms boot, <5 MiB footprint | <125 ms, <5 MiB, KVM, jailer | supports |
| 5 | S05 | https://docs.e2b.dev/sandbox/persistence.md ; https://docs.e2b.dev/network/internet-access.md | Pause ~4 s/GiB, resume ~1 s; paused kept until kill; internet on by default with allow/deny | Same figures; paused sandboxes kept indefinitely until killed; outbound internet default; allow/deny lists | supports |
| 6 | S06 | https://e2b.dev/pricing | Hobby 1 h; Pro 24 h, $150/mo plus per-second usage; Enterprise multi-day, BYOC | Hobby 1 h; Pro $150/mo + usage per second, 24 h; Enterprise multi-day persistent, BYOC | supports |
| 7 | S07 | https://www.daytona.io/docs/en/sandboxes.md ; https://www.daytona.io/docs/en/isolation.md | Default is a container; VM class with fork/pause/snapshots; tiered network policy; proxy-mediated secrets | Containers by default; Linux/Windows VM class with fork, pause/resume, memory snapshots; per-sandbox block/CIDR/domain by tier; secrets via outbound proxy placeholder | supports |
| 8 | S08 | https://www.daytona.io/pricing | $0.0504 vCPU-h, $0.0162 GiB-h, $0.000108 GiB-h storage after 5 GiB free, per second | Same rates and free tier, per-second metering | supports |
| 9 | S09 | https://api.github.com/repos/daytonaio/daytona | Repository archived | `archived: true`, last push 2026-07-24 | supports |
| 10 | S10 | https://modal.com/docs/guide/sandbox | gVisor or VM runtime; 5 min default, up to 24 h; snapshot beyond; inline secrets | gVisor or VM runtime; default 5 min, max 24 h, snapshot to continue; secrets supported | supports |
| 11 | S11 | https://modal.com/docs/guide/sandbox-networking | Outbound open by default; block/CIDR; beta domain allowlist | `block_network`, CIDR allowlist, domain allowlist (Beta); open by default | supports |
| 12 | S12 | https://modal.com/pricing | Billed on requested or actual usage, whichever is higher | "charged based on whichever is higher: the amount you request or the amount you actually use" | supports |
| 13 | S13 | https://vercel.com/docs/vercel-sandbox | Firecracker; persistent by default; snapshots, Drives; network/credential controls | Firecracker microVM; persistence default; Drives; snapshots; network and credentials controls | supports |
| 14 | S14 | https://vercel.com/docs/vercel-sandbox/pricing | iad1 $0.128 active vCPU-h; $0.0212 provisioned GB-h; $0.60/1M creations; Pro max 24 h, resumable | Same rates (iad1); Pro max session 24 h; limit resets on resume | supports |
| 15 | S15 | https://fly.io/docs/machines/overview/ | Creation possibly low double-digit seconds vs restart well under 1 s; ephemeral storage unless volume | "maybe low double digit seconds" create; start "usually well under a second"; ephemeral storage, attach a volume | supports |
| 16 | S16 | https://fly.io/docs/about/pricing/ | Per-second compute; stopped rootfs and volumes $0.15/GB-month; egress separate | Billed by the second; "Stopped or suspended Machine $0.15/GB/month of rootfs"; Volume $0.15/GB/month; outbound data metered | supports |
| 17 | S17 | https://developers.cloudflare.com/sandbox/ | Container microVMs vs Dynamic Workers (JS/Python/Wasm); `enableInternet: false` | Containers run in a microVM with their own kernel; Dynamic Workers for JS/Python/Wasm; `enableInternet:false` | supports |
| 18 | S18 | https://developers.cloudflare.com/sandbox/concepts/lifetime/ | DO keeps identity not files/processes; inactivity timeout; snapshots filesystem only | Storage kept but instance files/processes end; inactivity timeout up to 6 h; code inside does not count as activity; bills memory/disk while running | supports |
| 19 | S19 | https://developers.cloudflare.com/containers/pricing/ | Idle container bills provisioned memory/disk; CPU on active | Memory/disk on provisioned; CPU on active usage | supports |
| 20 | S20 | https://code.claude.com/docs/en/headless | `-p` json/stream-json, resume, no trust prompt, loads hooks/MCP; `--bare` needs API key, no hooks | `-p` skips trust dialog, loads project hooks and .mcp.json; `--bare` skips discovery, requires ANTHROPIC_API_KEY, hooks do not run | supports |
| 21 | S21 | https://platform.claude.com/docs/en/agent-sdk/overview | Agent SDK drives Claude Code binary; events/permissions/hooks; terms | SDK runs the Claude Code binary; tools, permissions, sessions, hooks; Commercial Terms | supports |
| 22 | S22 | https://www.conductor.build/blog/claude-subscription-update | 2026-06-15: May 13 restriction "delayed indefinitely"; integrator's own account | Dated June 15 2026; restriction "delayed indefinitely"; first-party Conductor post | supports |
| 23 | S23 | https://developers.openai.com/codex/noninteractive | `codex exec` for scripted runs | Documents `codex exec` non-interactive mode | supports |
| 24 | S24 | https://developers.openai.com/codex/sdk | TS SDK threads/resumeThread; Python SDK controls app-server via JSON-RPC; app server for clients | TS `startThread`/`resumeThread` with `sandboxMode`; "Python SDK controls the local Codex app-server over JSON-RPC"; app server for custom clients/approvals | supports |
| 25 | S25 | https://opencode.ai/docs/sdk/ | Embedded TS SDK owns a local server, sessions/worktrees, events | SDK "hosts OpenCode directly inside your application", assembles the server; `sessions` and `events` aliases; worktrees not seen | partly |
| 26 | S26 | https://opencode.ai/docs/acp/ | `opencode acp` private server, not the shared one; MCP-over-ACP unsupported | Starts a private server, not the shared background service; "doesn't support MCP over ACP" | supports |
| 27 | S27 | https://cursor.com/docs/cli/overview.md | `agent -p` print mode; resume | Print mode `agent -p`, `--output-format`; resume previous conversations | supports |
| 28 | S28 | https://cursor.com/docs/cli/acp.md | `agent acp` JSON-RPC stdio with permission requests | `agent acp`, stdio JSON-RPC 2.0, `session/request_permission` | supports |
| 29 | S29 | https://cursor.com/docs/sdk/typescript.md | Programmable local/cloud SDK | `@cursor/sdk`; local or cloud runtime picked in `Agent.create()`; same API key | supports |
| 30 | S30 | https://cursor.com/docs/cloud-agent/self-hosted | Cursor hosts loop/inference/planning; workers outbound, send files/diffs/outputs | "Cursor runs the agent loop, inference, and planning"; worker sends file contents, terminal output, diffs; outbound-only | supports |
| 31 | S31 | https://geminicli.com/docs/cli/headless/ | Headless prompts, `--output-format stream-json` JSONL; ACP UNVERIFIED | Headless `-p`, stream-json JSONL events; page does not mention ACP (but see F6) | supports |
| 32 | S32 | https://kiro.dev/docs/cli/headless/ | `--v3 --no-interactive` stream-json; API key or login; trusted tools; sandbox warns and continues | KIRO_API_KEY or stored login; `--trust-tools`; stream-json; `--cloud`; sandbox "warns and continues" unless `--sandbox` | supports |
| 33 | S33 | https://kiro.dev/docs/cli/acp/ | initialize, session/new/load/prompt/cancel, **permission exchanges**, notifications; prior sign-in | Methods, notifications and sign-in or terminal auth confirmed; no "permission" text on the page | partly |
| 34 | S34 | https://aider.chat/docs/scripting.html | `--message`/`--message-file`, `--yes`; Python API not officially supported | Same flags; "The python scripting API is not officially supported or documented" | supports |
| 35 | S35 | https://agentclientprotocol.com/overview/introduction | ACP standardizes editor/client to coding-agent interaction, JSON-RPC, diffs | Standardizes editors/IDEs and coding agents; JSON-RPC over stdio; diff UX | supports |
| 36 | S36 | https://github.com/agentclientprotocol/claude-agent-acp | Maintained Claude adapter, not native CLI ACP | ACP adapter built on the Claude Agent SDK | supports |
| 37 | S37 | https://github.com/zed-industries/codex-acp | Maintained Codex adapter | "Development has moved to agentclientprotocol/codex-acp"; built on the Codex App Server | partly (stale repo, F7) |
| 38 | S38 | https://modelcontextprotocol.io/specification/2025-11-25 | MCP = host/client to tools/resources/prompts | Hosts, clients, servers expose tools/resources/prompts; LSP analogy | supports |
| 39 | S39 | https://a2a-protocol.org/latest/specification/ | Opaque agents; Agent Cards, tasks, artifacts, streaming, push | Same concepts in the spec | supports |
| 40 | S40 | https://docs.temporal.io/self-hosted-guide | Dev server single binary not production; production needs Service, store, namespaces, upgrades | Dev server "single binary with no external dependencies" for local dev; production checklist, namespaces, upgrades | supports |
| 41 | S41 | https://raw.githubusercontent.com/restatedev/restate/main/LICENSE ; https://docs.restate.dev/server/deploy | BSL 1.1, public-platform-service restriction, 4-year Apache-2.0 change; single-node and K8s/Helm routes | BSL 1.1 with the Restate Platform Service restriction, internal use permitted, Change Date 4 years to Apache-2.0; deploy page shows K8s operator/Helm only | partly (F8b) |
| 42 | S42 | https://www.inngest.com/docs/self-hosting | Single binary; embedded Redis/SQLite default; external Redis/Postgres | Single binary, in-memory Redis, SQLite default, external Redis/Postgres; SQLite not beyond single node | supports |
| 43 | S43 | https://raw.githubusercontent.com/inngest/inngest/main/LICENSE.md | Server SSPL with three-year future Apache-2.0 grant | SSPL v1 plus Apache 2.0 Future License effective on the third anniversary | supports |
| 44 | S44 | https://trigger.dev/docs/self-hosting/overview | Webapp + worker, Postgres/Redis; checkpoints, warm starts, autoscaling cloud-only | Same architecture; those features Cloud-only | supports |
| 45 | S45 | https://docs.dbos.dev/ ; https://raw.githubusercontent.com/dbos-inc/dbos-transact-ts/main/LICENSE | Embedded durable workflows, Postgres-backed queues; MIT TS library | "Reliable Queues … backed by Postgres"; LICENSE is MIT | supports |
| 46 | S46 | https://www.postgresql.org/docs/current/sql-update.html ; https://www.postgresql.org/docs/current/sql-select.html | `FOR UPDATE SKIP LOCKED` for multiple consumers | Row locking with FOR UPDATE confirmed; the SKIP LOCKED text was not present in my extracted page text (extraction gap, not evidence against) | partly |
| 47 | S47 | https://supabase.com/docs/guides/self-hosting | Self-host is one project; no multi-org Studio | "runs as a single project … Studio doesn't support multiple organizations or projects" | supports |
| 48 | S48 | https://supabase.com/docs/guides/database/postgres/row-level-security | RLS bypass via security-definer views/functions, service_role | Views bypass RLS by default (security definer); service_role bypasses | supports |
| 49 | S49 | https://github.com/coollabsio/coolify ; https://api.github.com/repos/coollabsio/coolify | Teams, SSH hosts, Docker networks, backups, optional cloud; Apache-2.0 | Teams and integrations, SSH servers, isolated Docker networks, scheduled backups, Coolify Cloud; licence Apache-2.0 | supports |
| 50 | S50 | https://github.com/makeplane/plane ; https://developers.plane.so/self-hosting/ | Docker/K8s/AIO; AGPL-3.0 Community; commercial editions with workspace or instance licensing | AGPL-3.0 and self-host deploy confirmed; edition/licensing model not stated on either page | partly (F8c) |
| 51 | S51 | https://docs.gitlab.com/development/licensing/ | CE MIT; EE restricted | CE MIT, EE under a restricted licence | supports |
| 52 | S52 | https://docs.gitlab.com/update/ | Published upgrade path, package dependencies, licence checks | Fetched twice: only "Upgrade GitLab / Upgrade a GitLab instance to a later version." | unreadable (F8e) |
| 53 | S53 | https://backstage.io/docs/plugins/ | Frontend/backend plugin extension points; library-derived app, not a white-label binary | Legacy plugin intro pointing to the new frontend/backend systems; SPA built from plugins. The white-label point is an inference | partly |
| 54 | S54 | https://backstage.io/docs/getting-started/keeping-backstage-updated/ | Package bumps do not merge customised generated templates | `backstage-cli versions:bump`; separate "Following create-app template changes" section; explicit "not automatic" wording not captured | partly |
| 55 | S55 | https://github.com/coder/coder ; https://api.github.com/repos/coder/coder | AGPL-3.0 self-hosted server; Terraform workspaces on VMs/containers/K8s | Terraform templates on EC2/K8s/Docker; Coder Agents; licence AGPL-3.0 | supports |
| 56 | S56 | https://coder.com/pricing | Community allows third-party agents; Premium audit/RBAC/quotas; AI Premium Coder Agents/API | Same tier split | supports |
| 57 | S57 | https://ona.com/docs | Customer-VPC execution; new-customer signup closed | "New customer signup is closed"; existing orgs can run in own VPC on AWS/GCP | supports |
| 58 | S58 | https://www.openpolicyagent.org/docs/latest/ | Decision decoupled from enforcement; embeddable Go library | "decouples policy decision-making from policy enforcement"; "can be embedded inside Go programs as a library" | supports |
| 59 | S59 | https://docs.cedarpolicy.com/auth/authorization.html | PARC, default deny, forbid overrides, skip on error | Same, including policies skipped on evaluation error | supports |
| 60 | S60 | https://www.conductor.build/docs/faq | Local agents run without sandboxing | "Agents run directly on your system without sandboxing" | supports |
| 61 | S61 | https://www.conductor.build/docs/cloud/faq | Vercel Sandbox; GitHub-only cloud repos | Vercel sandbox; 8 vCPU/16 GB/32 GB ephemeral NVMe; GitHub only; chats stored on Conductor servers | supports |
| 62 | S62 | https://www.conductor.build/docs/cloud/cloud-computer | 8/16/32 GB ephemeral microVM; secrets warning | Isolated microVM 8/16/32 ephemeral; secrets exposed as env vars, "don't print" | supports |
| 63 | S63 | https://docs.conductor.build/api | Beta `/v0`: workspaces, prompts, status/transcripts, webhook routines | Beta `/v0` workspaces, sessions/messages, status, `POST /v0/sql` over `session_transcripts_view`, routines (cloud workspaces) | supports |
| 64 | S64 | https://www.conductor.build/changelog/0.94.2-conductor-is-back | 0.94.2 reversed the shutdown | "Conductor isn't shutting down. It's back, and we're building again." | supports |
| 65 | S65 | https://www.conductor.build/changelog/0.93.0-conductor-is-joining-meta | Announced joining Meta; "team says it continued development there" | "Conductor is joining Meta"; wind-down text; no statement about continuing development at Meta | partly (F8d) |
| 66 | S66 | https://github.blog/news-insights/company-news/welcome-home-agents/ | Multi-vendor mission control, policy/audit control plane, self-hosted runners | 2025-10-28: Anthropic, OpenAI, Google, Cognition, xAI "over the coming months"; mission control; control plane with policies/audit; Actions or self-hosted runners | supports |
| 67 | S67 | https://docs.github.com/en/copilot/concepts/agents/about-third-party-agents | Third-party agents: Claude and Codex, public preview | Public preview; Claude and Codex | supports |
| 68 | S68 | https://github.com/pingdotgg/t3code | MIT; local server; remote/mobile; ACP agents | MIT; one server, remote, mobile, any ACP agent | supports |
| 69 | S69 | https://github.com/getpaseo/paseo | Apache-2.0; daemon, Docker self-host, SDK, plugins | LICENSE is Apache-2.0 (GitHub API reports NOASSERTION); daemon, Docker self-host, TS SDK, plugins | supports |
| 70 | S70 | https://github.com/stablyai/orca | MIT; remote runtime/cloud VM; bots | MIT; remote server, cloud VM per worktree, bots | supports |
| 71 | S71 | https://github.com/herdrdev/herdr | Apache-2.0; Rust background server; socket API | Apache-2.0; single Rust binary, background server, socket API | supports |
| 72 | S72 | https://github.com/ParadevAI/Paradev | MIT; worktrees; CLI/SDK/MCP; paid remote/cloud | MIT; worktrees "do not sandbox processes"; CLI/MCP; Pro automations; cloud workspaces | supports |
| 73 | S73 | https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/managing-protected-branches/about-protected-branches | Required reviews, status checks, dismiss stale approvals | Same; administrators bypass by default unless restricted | supports |

Additional evidence (not an R2 source): https://agentclientprotocol.com/overview/agents lists Gemini CLI, Kiro CLI, OpenCode, Cursor, Codex CLI, Claude Agent and GitHub Copilot, and not Aider (F6).

## What is strong

- **Runtime-class precision.** R2 separates checkout from isolation and container from microVM (L7–L27). It also checks each vendor's *default* runtime: Daytona is a container by default and Modal offers gVisor *or* VM. All of these claims verified.
- **Cost realism.** It points out that idle provisioned RAM, stopped rootfs and snapshots can defeat a token-only budget cap (L77, L171). The Vercel, Fly, Cloudflare and Daytona figures all matched the live pricing pages.
- **Gate design.** Approvals are bound to a SHA and invalidated when the branch changes. Runtime tool permission is separated from the change gate. Merge authority stays with the code host's branch protection (L75, L151), which S73 supports.
- **Honest uncertainty.** UNVERIFIED labels are used where R2 lacked evidence: Conductor's ownership, Aider ACP, the vendor-policy notice. The Anthropic policy item is correctly framed as the integrator's own account (L40).
- **Licences checked against the actual licence files.** Restate's BSL, Inngest's SSPL plus future grant, DBOS MIT, Coolify, Plane, Coder and Paseo were all read from LICENSE files rather than marketing pages. Every licence claim in the table verified.
- **Accurate current state.** Conductor is alive (0.94.2). Agent HQ is described as public preview with Claude/Codex only, and its announced partner list is kept apart from what is GA. Ona's signup is closed. Daytona's repo is archived. All match R1 and R3.
- **The verdict discipline is mostly right.** The text says explicitly that the BLOCKING analysis does not distinguish B from C, and it marks the B-inside-C preference as ADVISORY (L134).
- **Durability recovery is stated honestly.** At-least-once side effects, idempotency keys, fencing epochs and a `needs_reconciliation` state are spelled out (L85).

## Learnings / mistakes

1. **I did not fill the review as I went.** I created the skeleton early, but wrote no spot-check rows or findings into it before the context was compacted. Had the run died then, the partial file would have held only headings. Rule: append the table rows and findings to the review file every ~10 sources, not only at the end.
2. **I wrote grep syntax that the search tool does not accept.** Once I used GNU-style `a\|b`; the search tool is ripgrep and needs `a|b`. Rule: write ripgrep alternation from the start.
3. **Long outputs were cut even with raw output on.** Long multi-section command outputs were still filtered, which hid lines (S64 text, the Gemini check). Rule: send multi-check output to a file, then `view` it.
4. **One scan nearly gave a false PASS.** My first check for personal names used `rg -n -o` with `awk` field 3. That field was always empty, so the scan printed nothing. I noticed and re-ran it correctly. Rule: before trusting a scan that returns nothing, confirm it can return something.
5. **I recorded a stub page as unreadable instead of substituting a neighbour.** S52 returned a 201-byte stub twice. I logged it as unreadable rather than swapping in a nearby GitLab page, to respect "read the exact page cited".
6. **An extraction gap is not a mismatch.** The S46 extracted text did not contain "SKIP LOCKED", even though row locking with FOR UPDATE was present. I graded it "partly, extraction gap", not "mismatch". Rule: a phrase missing from extracted text lowers confidence; it does not contradict the claim.

## Re-verification (v2)

status: COMPLETE. Verdict: APPROVE (F1–F3 BLOCKING closed; F4–F12 ADVISORY closed; new F13–F15 ADVISORY)

- Target: `docs/research/R2-technical-landscape.md` DRAFT v2 (48,137 B, 316 lines; 80 source IDs, 89 URLs). Re-verifier: Opus · review R2. Date: 2026-11-25. R2 was not edited.

### Per-finding status

| Finding | Tag | Status | Evidence |
|---|---|---|---|
| F1 Licence vs R3 | BLOCKING | closed | L110: "**Governing decision:** R3 §B, approved v4, selects Apache-2.0 core plus a commercial, source-available `enterprise/` module. FSL/BSL/ELv2/SSPL are [BLOCKING] for the core, not reopened here." L102: bundling Restate's BSL server or Inngest's SSPL server in the core is "[BLOCKING] for that bundled variant; neither may be a hard dependency". L116 says the same once for all options ("A–E do not bundle those servers"). L55 repeats it in Facts. Open question L194 now asks only about packaging and the dependency check ("not a new licence choice") and applies R3's DCO-without-CLA handoff, which R3 L212 does contain. L201 uses "dependency-licence compatibility checks under R3's decision". The R3 anchors (`#b-open-core--self-hosted-playbook`, `#analysis`, `#handoffs-to-other-docs`) match R3's headings (R3 L89, L73, L208). |
| F2 Why preferred | BLOCKING | closed | Every option now has the line: A L123 "Why not preferred"; B L130 "Why preferred (MED)"; C L137 "Why preferred (MED)"; D L144 "Why not preferred"; E L151 "Why not preferred initially (MED)". |
| F3 Conditional BLOCKING / tag consistency | BLOCKING | closed | The cross-option requirements are stated once (L116): agent-neutral interfaces and checkout-vs-sandbox separation. Failures of either are "[ADVISORY] shared implementation risks for A–E, never a selective BLOCKING con". Each option carries the same tag: A L121, B L128, C L135, D L142 and E L149 all say "[ADVISORY] … shared implementation risks". B's conditional blocker is gone ("No inherent BLOCKING con", L128). Table L160 for B reads "None". Agent-neutrality row L158: A–C "Shared requirement", D "Vendor-dependent, advisory", E "Adapter-dependent, advisory", so no BLOCKING anywhere. Verdict L162, recomputed from BLOCKING items alone: eliminate A (team plane) and D (self-host first); B, C and E viable; "B first, within C's boundaries" as an ADVISORY preference; E to be compared in the HLD. This is the expected result. |
| F4 VISION invariants | ADVISORY | closed | L114 quotes VISION L21 ("expose it to your teams"), L27–28 ("team-shared, policy-gated agent fleets"), L41 ("agent-agnostic by contract") and L47 ("self-host first, hosted later"). It calls the shared durable queue "a **derived design requirement**, not VISION's explicit invariant" and cites VISION L43–44 for the optional sandbox. A's sandbox con is now [ADVISORY] (L121). |
| F5 Missing option (extend existing platform) | ADVISORY | closed | Option E added (L146–151) with pros, cons, mitigation and a "Why not preferred initially" line; E column in the table (L153–160); verdict L162 keeps E viable for an HLD spike. New sub-issues are F13 and F14 below (both ADVISORY). |
| F6 Gemini ACP | ADVISORY | closed | L45: "**native `gemini --acp`**, documented as JSON-RPC 2.0 over stdio …" [S31][S74]. L48 lists Gemini among the native ACP agents. S74 supports this (spot-check row 75). |
| F7 Codex ACP repo | ADVISORY | closed | L48: "current `agentclientprotocol/codex-acp` built on Codex App Server; this does not establish native CLI ACP". S37 URL is now `https://github.com/agentclientprotocol/codex-acp` (L241), which supports it (row 74). |
| F8 Over-claims (Kiro, Restate, Plane, Conductor, GitLab) | ADVISORY | closed | (a) L46: the Kiro "permission exchanges" claim is gone. (b) L55: Restate is limited to "Kubernetes operator/Helm, not a standalone production binary". (c) L64: the Plane edition-licensing assertion is removed; per-customer isolation is labelled "(inference)". (d) L76: Conductor "subsequently resumed development; … where that development occurs" is marked UNVERIFIED, consistent with S64's "we're building again". (e) L65: GitLab now says the upgrade page "confirms only an upgrade section, not package-dependency/licence-check details". |
| F9 Missing precedents | ADVISORY | closed | Claude self-hosted runners L41 [S75]; Emdash ACP L80 [S76]; Conductor `POST /v0/sql` `session_transcripts_view` L76; full-devbox/Devin VPC L70 [S78]; Codespaces L71 [S77], which also appears in option C's pros (L134) and the HLD-000 provider handoff (L200). Rows 74–81 support each new source. |
| F10 Provider facts | ADVISORY | closed | New 7-row provider table (L15–23) gives startup latency, cost, persistence, network and secrets for E2B, Daytona, Modal, Vercel, Fly, Cloudflare and Firecracker. L13 says "vendor startup claims, **not checkout/setup/agent-ready benchmarks**" and that "not published" means no number on the cited pages. Changed claims re-checked in rows 82–87. |
| F11 Audience naming | ADVISORY | closed | "Audience 1 (platform/DevEx)", "Audience 2 (engineering teams)" and "Audience 3 (B2B2B)" are used at L162, L177, L183, L184 and L193. They match VISION's three audiences (VISION L25, L27, L30). |
| F12 Team estimate sourcing | ADVISORY | closed | L166: "**Estimate, confidence LOW — author inference, no source:**". The misleading citations are removed. |

### Spot-check extension (new or changed sources)

The rows continue the numbering of the v1 table. Every page was fetched in this run.

| # | Source id | URL | Claim in R2 v2 | What the page says | Verdict |
|---|---|---|---|---|---|
| 74 | S37 (changed URL) | https://github.com/agentclientprotocol/codex-acp | L48: the current Codex adapter is `agentclientprotocol/codex-acp`, built on the Codex App Server; adapter, not native CLI ACP | "ACP adapter for Codex CLI … a stdio ACP agent server. It starts the Codex App Server, translates ACP requests into Codex operations" | supports |
| 75 | S74 | https://geminicli.com/docs/cli/acp-mode/ | L45/L48: native `gemini --acp`, JSON-RPC 2.0 over stdio, authentication, session load, prompt/cancel, approval-level configuration | "`gemini --acp`"; "stdio using the JSON-RPC 2.0 protocol"; methods initialize, authenticate, newSession, loadSession, prompt, cancel; "setSessionMode: … changing the approval level for tool calls" | supports |
| 76 | S75 | https://code.claude.com/docs/en/self-hosted-environments | L41: customer runners lease an Anthropic-hosted queue, clone repos, launch local Claude Code; checkouts/artifacts stay customer-side; event streams and transcripts go to Anthropic; inference can route elsewhere | "Anthropic's control plane places the session on the environment's queue"; "A runner … claims the session and holds a lease"; "clones the repository … spawns a child Claude Code process"; "repository checkouts and build artifacts stay on infrastructure you control. Session content still goes to api.anthropic.com"; inference can optionally route to third-party cloud model services (paraphrased; the page names two) | supports |
| 77 | S76 | https://emdash.sh | L80: own agents "through ACP"; local, customer-server or cloud execution; Classic open desktop gets fixes, new development moved to 2.0 | "Use Claude Code, Codex, Cursor, OpenCode or your own agents through ACP"; "Run locally, on your servers or on Emdash cloud"; "Emdash Classic: The open-source desktop app … still gets fixes, but new development has moved to Emdash 2.0" | supports |
| 78 | S77 | https://docs.github.com/en/codespaces/overview | L71: GitHub-hosted Docker dev container on a VM; repo dev-container configuration; GitHub CLI access | "hosted by GitHub in a Docker container, running on a virtual machine"; dev container configurations; "connect … by using GitHub CLI" | supports |
| 79 | S78 | https://docs.devin.ai/enterprise/vpc/overview | L70: Devin's brain always in Cognition cloud; Customer Dedicated Deployment is a Cognition-managed single-tenant VPC linked to the customer | "brain … always residing in Cognition's Cloud"; "Cognition hosts Devin in an auto-scaling, customer-isolated environment within a single-tenant VPC. Your VPC connects via [a cloud private-link service] (or IPSec tunnel)" | supports |
| 80 | S79 | https://docs.e2b.dev/secrets.md ; https://docs.e2b.dev/sandbox/environment-variables.md | L17: an HTTPS proxy injects stored keys outside the VM; env vars are readable by sandbox code | "The egress proxy injects the real value into matching outbound HTTPS requests outside the sandbox"; an env var is where "any code running in the sandbox can read it"; second page documents sandbox env vars | supports |
| 81 | S80 | https://fly.io/docs/apps/secrets/ | L21: app vault injects env vars at boot on all of the app's Machines; separate apps by trust level | "An app's secrets are available as environment variables at runtime on every Machine belonging to" the app; host agent decrypts and injects "at boot time". The per-trust app separation is the author's inference | supports |
| 82 | S05 (changed claim, same URL) | https://docs.e2b.dev/network/internet-access.md | L17: "domain rules have DNS/shared-endpoint caveats" | "When any domain is used, the default nameserver `8.8.8.8` is automatically allowed"; this "covers **all traffic to `8.8.8.8`**, not only DNS"; "On shared infrastructure such as CDNs and object storage, allowing one hostname can also permit access to other de[stinations]" | supports |
| 83 | S07 (changed claim, same URL) | https://www.daytona.io/docs/en/sandboxes.md ; https://www.daytona.io/docs/en/isolation.md | L18: container "<90 ms advertised"; "Tier-dependent defaults" for network | feature string `<90ms`; network "Open on Tier 3 and above; restricted on Tier 1 and 2" | supports |
| 84 | S10 (changed claim, same URL) | https://modal.com/docs/guide/sandbox | L19: Ready follows initialization/probe; 5-min default up to 24 h, then a new sandbox; inline secrets become environment; OIDC identity optional | "Ready — When a readiness probe is [configured] … fully initialized and ready to accept traffic"; "default maximum lifetime of 5 minutes … timeout of up to 24 hours"; past 24 h "Filesystem Snapshots … restore from that snapshot with a subsequent Sandbox"; "set environment variables using inline secrets"; `include_oidc_identity_token=True` injects an OIDC token | supports |
| 85 | S11 (changed claim, same URL) | https://modal.com/docs/guide/sandbox-networking | L19: "Open default; block/CIDR; beta HTTPS/SNI domains" | "By default, Sandboxes can make outbound connections to any public IP address"; `block_network=True`; `outbound_cidr_allowlist`; "Domain allowlist (Beta) … Only allows TLS traffic (port 443) to the listed domain names" | supports |
| 86 | S13 (changed claim, same URL) | https://vercel.com/docs/vercel-sandbox | L20: "Milliseconds"; Secure Compute/static egress; project/team credential configuration | "Sandboxes start in milliseconds"; "Attach sandboxes to a Secure Compute network to send their public-internet traffic through stati[c IPs]"; "credentials Vercel sets up from your project's or team's configuration" | supports |
| 87 | S17 (changed claim, same URL) | https://developers.cloudflare.com/sandbox/ | L22: "Keep credentials in mediating Worker; anything passed inside is exposed"; `enableInternet:false` blocks outbound | "Keep credentials in your Worker, and decide which services a sandbox reaches"; `enableInternet: false` appears in the example. The explicit exposure statement is on the uncited security page (https://developers.cloudflare.com/sandbox/concepts/security/: "Assume that the code uses everything it can reach there, including the files, credentials, and network access that your Worker provides") | supports (nit: the exposure clause's best source is uncited) |

Tally for the extension: 14 rows (74–87), 14 supports, 0 partly, 0 mismatches, 0 unreadable. With the v1 table (63 supports, 9 partly, 1 unreadable), the combined tally is 87 rows: 77 supports, 9 partly, 1 unreadable (S52, unchanged). v1 partly rows that v2 rewrote (Kiro S28, Restate S41, Plane S50, GitLab S51/S52, Conductor S64) are covered under F8 above.

### Option E review (new material, same standard as A–D)

| Element | Check | Result |
|---|---|---|
| Definition (L146) | Is E distinct from A–D? | Yes. It reuses an existing OSS server/platform (Paseo daemon/SDK/plugins, or an out-of-process Coder API) instead of building runner plumbing. A–D build or rent that layer. |
| Pro: "Paseo's Apache daemon/SDK/plugins offer reuse" [S69] | Sourced? | Supported. S69 shows "License Apache-2.0", "packages/server: Paseo daemon (agent process orchestration, WebSocket API, MCP server)", the client SDK and plugins that "run on the daemon", plus a server/VM/Docker install ("For a server, a VM, or any machine without the desktop app"; "Run the Paseo daemon and self-hosted web UI in Docker"). |
| Pro: "out-of-process Coder API integration can use customer workspaces without importing its AGPL source" [S55][S69] | Sourced? | Partly. S55 (github.com/coder/coder) documents Terraform workspaces, the coderd Terraform provider, the CLI, IDE extensions and GitHub Actions, but it does not document a workspace REST API. S69 is a Paseo source and says nothing about Coder. The pricing matrix S56 (cited on the cons line, not on this pro) lists "Web UI, CLI, REST API" under Platform Experience, and it puts "Trigger background agents via API" / "Coder Agents API" under AI Premium. See F14. |
| Con tags | Labelled correctly? | Yes. All four E cons are [ADVISORY]: shared neutrality/sandbox risk worded the same as A–D (L116 invariant), fork divergence, durability still to build, and Coder paid-governance overlap. None is a violated invariant. |
| Variant tag: "E-embedded-Coder … [BLOCKING]" | Labelled correctly? | Yes. Embedding AGPL-3.0 source in the Apache-2.0-only core breaks R3 §B's approved licence decision, which is a governing invariant, so BLOCKING is right. It is scoped to a variant, as with the bundled-BSL/SSPL variants (L102, L116), so E itself carries no BLOCKING con. This matches the F3 rule. |
| Mitigation (L150) | Actionable? | Yes: "test SDK/API integration without a fork or bundled Coder server; keep baton-owned gates and audit." |
| Why line (L151) | Present (F2 standard)? | Yes: "Why not preferred initially (MED)". It says E is "viable and should be spiked against B during HLD, not eliminated by judgment", so the BLOCKING-only verdict is kept. |
| Table column (L153–160) | Consistent with text? | Mostly. "Shared durable gate queue: Add baton layer" and "BLOCKING: None; embedded-AGPL variant excluded" match the text. "Customer-hosted team plane: Yes" is conditional: S69 describes Paseo as a daemon that "manages your coding agents", with no users, roles or team queue shown. A Paseo-based E gets a team plane only through Coder or the baton layer it adds. See F13. |
| Verdict (L162) | Follows from BLOCKING alone? | Yes. E survives; ranking E behind B is explicitly an advisory preference. |
| Rule 5 | Clean? | Yes. Only public vendor/OSS names are used. |

Conclusion: option E meets the A–D standard. No con is BLOCKING-mislabelled. One pro is under-sourced (F14), and one table cell needs a qualifier (F13). Both are ADVISORY.

### Rule 5 provenance scan (v2, full file)

The scan covered the whole of `R2-technical-landscape.md` v2: 48,137 B, 316 lines, 5,990 words (`wc -w`), inside the 3,000–6,000 target used in the v1 review (line 114 above). That target is not in BRIEF.md; it came with the R2 task.

| Check | Command / method | Result |
|---|---|---|
| Scanner sanity | `rg -c "https"` | 80 lines matched, so the scanner works on this file |
| Local paths | `rg -n "/Users/\|~/\|/home/\|Library/\|C:\\\\"` | 0 matches |
| Emails / handles | `rg -c "@"` | 0 matches |
| Employer / internal systems | case-insensitive `rg` for the operator's employer, its cloud brands, auth/ticket/wiki system names and the maintainer's login | 0 matches |
| Personal names | all capitalized word pairs (`\b[A-Z][a-z]+ [A-Z][a-z]+\b`), 31 distinct | All are products, concepts or sentence starts (e.g. Claude Code, Vercel Sandbox, Docker Compose, Coder Agents, Durable Object, "Removed Kiro", "Added Claude"). No personal names |
| URL hosts | host extraction over all URLs, 44 distinct | All public vendor, OSS or standards domains. GitHub owners are project organisations (agentclientprotocol, restatedev, inngest, dbos-inc, coollabsio, makeplane, coder, pingdotgg, getpaseo, stablyai, herdrdev, ParadevAI, daytonaio). No internal hosts |
| Maintainer references | read-through of the new v2 text (provider table, L41–80, L102–116, option E, verdict, open questions, Review log) | The maintainer is never named. "Operator" in R2 means a generic infrastructure operator (L12, L23, L55, L63, L102, L108, L110), not the maintainer |
| Non-public documents | source list S01–S80 (host list above) | Every cited host is a public site. This run fetched 20 of the 89 URLs: the 9 new or changed ones, plus 11 re-checks for changed claims and option E (one extra, uncited Cloudflare page was fetched for row 87). v1 fetched the rest. No URL needs a login or internal network |

Result: **PASS**. Review-file hygiene: rows 76 and 79 of this extension originally quoted vendor pages that name specific cloud providers' products. I paraphrased both to keep this file vendor-neutral. R2 itself does not name them (L41 says "inference can route elsewhere"; L70 omits the link type).

### New findings

| # | Tag | Location | Finding | Suggested fix |
|---|---|---|---|---|
| F13 | ADVISORY | L155 (E column, "Customer-hosted team plane: Yes"); L148 | The "Yes" is conditional. S69 describes Paseo as a daemon that "manages your coding agents" and can run on a server/VM/Docker, but it shows no users, roles or team queue. A Paseo-based E gets a team plane only through customer-hosted Coder or the baton layer E adds (the L156 "Add baton layer"). Read alone, the cell overstates what reuse buys. | Change the cell to "Yes, via Coder or added baton layer", or add one sentence to L148 saying that Paseo per S69 is an agent daemon without team roles or a queue. |
| F14 | ADVISORY | L148 (E pro, Coder API) | The "out-of-process Coder API integration" pro cites [S55][S69]. S55 documents only the Terraform provider, CLI, IDE extensions and GitHub Actions; S69 is about Paseo. The API evidence is S56's matrix ("Web UI, CLI, REST API"), which is not cited on this line. The same page puts "Trigger background agents via API" / "Coder Agents API" in AI Premium. | Cite S56 (or Coder's REST API docs) on L148, and state that E would use the workspace REST API, not the paid Coder Agents API. This sharpens E's existing "paid governance overlap" con. |
| F15 | ADVISORY (nit) | L22 (Cloudflare secrets cell) | The "anything passed inside is exposed" clause is stated most directly on Cloudflare's uncited sandbox security page (https://developers.cloudflare.com/sandbox/concepts/security/). S17 only says "Keep credentials in your Worker". | Optional: add the security page as a source, or drop the clause. |

None of F13–F15 is a violated invariant. None changes the BLOCKING-only verdict (eliminate A and D; B, C and E viable).

### Final verdict (v2)

**APPROVE.** All three BLOCKING findings (F1–F3) are closed. All nine ADVISORY findings (F4–F12) are closed as claimed in the Review log. Option E meets the A–D standard. All 14 rows for new sources and changed claims (74–87) support their claims; row 87 carries a nit. The new findings F13–F15 are ADVISORY: the author may apply or decline them with a Review-log "why not", and they do not hold back the lead's commit.
