# VISION.md DRAFT v1: cross-model review

status: COMPLETE (v1 review: REVISE; v2 re-verification: APPROVE, see '## Re-verification (v2)')

Reviewer: Opus. Target: `VISION.md` DRAFT v1, 10,998 B, written by a different model. Sources checked: R1 APPROVED v2, R2 APPROVED v2, R3 APPROVED v4 (per its Review log row R3 L290). VISION lines are `L<n>`; research lines are `R1 L<n>`, `R2 L<n>`, `R3 L<n>`. `VISION.md` was not edited and git was not run.

## Verdict

REVISE: three BLOCKING findings (B1 a per-seat price contradicted by its cited R3 §3; B2 audience confidence labels cited to an R1 section that does not hold them, with R1's audience-3 LOW missing; B3 licence eliminations asserted while L10 weakens the customer invariant R3 says they rest on), each fixable in under 150 B and fundable by deleting the stale R2 labels (A1).

- Findings: 3 BLOCKING, 12 ADVISORY.
- Citation tally (47 groups): 36 supports, 8 partly, 2 not there, 1 contradicts.
- Re-verification on revision: re-check B1 to B3 on disk, the byte size stays within 7,000 to 11,000, and the lead's v0.1 diff for A11.

## Findings

### B1 [BLOCKING] "$50 to $80 per seat" is contradicted by the cited R3 §3

- VISION L17: "teams still pay $50 to $80 per seat (R1 §2, §3b, §5; R3 §3)".
- R3 §3 L41 classes the $80 anchor as "Per team flat fee: Devin 'Teams' $80/month for up to 100 users".
- R1 §3a L115 records the same fact: "Teams $80/mo for up to 100 users".
- R1 §5 L205 says "team plans existed or exist at $50–80", not "per seat"; its per-user plans are Conductor Teams $60 and Paradev Team $60 (Conductor Pro is $50/mo per Pro user, R1 §1 L51).
- So $80 is a per-team price, and the per-seat claim is false on its own cited evidence.
- Fix (bounded, about +30 B): "teams pay $50 to $60 per user (Conductor, Paradev) and $80 a month for a Devin team (R1 §5; R3 §3)". Drop "not the UI" in the same bullet (A5) to fund it.

### B2 [BLOCKING] Audience confidence labels cite a section that does not hold them, and R1's audience-3 confidence is missing

- VISION L22 sets the convention "R1 confidence in brackets".
- VISION L24 "[R1 §5: real, MED]" and L25 "[R1 §5: real but contested, MED]": R1 §5 (L195 to L213) holds only the For/Against evidence; the verdicts "real — MED" and "real but contested — MED" are in R1 Recommendations, "Per-audience verdicts", L256 and L257. Citation to a section that does not contain the claim.
- VISION L26 3a "[MED, absence of evidence]" reads, under L22's convention, as R1's confidence. It is R3 §A's label (R3 L84).
- R1's own verdict for audience 3, "not evidenced — LOW" (R1 L258), appears nowhere in VISION, although PHASE-1 asks for "R1's confidence and R3's 3a/3b split".
- Fix (bounded, about +40 B): change both "R1 §5:" to "R1 verdict:" (or "R1 Recommendations:"); add "[R1 verdict: not evidenced, LOW]" after "B2B2B embedders"; prefix 3a's label "R3 §A:".

### B3 [BLOCKING] The licence eliminations are stated as BLOCKING while L10 weakens the customer invariant they rest on

- R3 §B L91: "VISION.md says any company can offer baton 'to its teams *or to its customers*'; a licence that forbids offering the software to customers is `[BLOCKING]`".
- R3 Handoffs L212: "the BLOCKING eliminations in §B and §D rest on VISION v0.1 invariants and reopen if those change".
- VISION L10 now reads "for your teams and, if demand proves real, your customers". That makes offering baton to customers conditional on demand, conflating the Audience 3 sales motion with a licence right.
- VISION L38 still eliminates FSL, BSL, ELv2 and Sustainable Use "because they forbid a customer offering the software to its own customers", but VISION no longer states that right unconditionally anywhere.
- Under R3's own handoff, a weakened invariant reopens those eliminations; asserting them as BLOCKING contradicts R3 L212.
- The "open" invariant is carried adequately: L38 ties SSPL's elimination to "not OSI-approved (R3 §B)", and L40's closed-first elimination cites R3 §D, which uses R3 §B's definition.
- Fix (bounded, about +110 B): add one line under Strategy: "Invariants (R3 §B): any company may offer baton to its own teams or customers; the core licence is OSI-approved; self-host first, hosted later." Optionally reword L10 to "your teams and, later, your customers".

### A1 [ADVISORY] Stale R2 labels: R2 is now APPROVED v2

- R2 L3: "Status: APPROVED v2 (re-verified 2026-11-25 ...)".
- L4 status line: "Inputs: ... R2 DRAFT v2; claims resting on R2 alone are marked (R2 v2, re-verification pending)".
- L18: "(R2 §1; R2 v2, re-verification pending)".
- L42: "(R2 options analysis and verdict; R2 v2, re-verification pending)".
- L48: "Every R2 citation in this section is R2 v2, re-verification pending."
- Fix: L4 "R2 APPROVED v2" with the clause deleted; delete "; R2 v2, re-verification pending" at L18 and L42; delete the L48 sentence. Frees 202 B, which funds B1 to B3.

### A2 [ADVISORY] "vendor-owned" is stated as fact; the approved docs mark ownership UNVERIFIED

- L8: "alive, proprietary and vendor-owned ... (deal terms UNVERIFIED)".
- R3 §4 L55: "Deal structure and ownership: `UNVERIFIED`; the primary wording is 'joining Meta'". R3 §C L113: "developed at Meta (ownership `UNVERIFIED`)".
- R2 §5 L76: the changelogs "do not establish legal ownership or where that development occurs (UNVERIFIED)".
- Not BLOCKING: VISION carries an adjacent UNVERIFIED tag and PHASE-1 mandates the "vendor-owned" wording; the tag is scoped too narrowly.
- Fix (+14 B): "(ownership and deal terms UNVERIFIED)".

### A3 [ADVISORY] "Free OSS desktop tools" understates the OSS competitors

- L8: "Free OSS desktop tools contest the team segment (R1 §3b, §5)".
- R2 §5 L78: "OSS competitors are **not merely offline desktop apps**" (T3 Code local server, Paseo Docker self-host, Orca cloud VM, Herdr background server); "Desktop UI alone cannot differentiate baton".
- R3 §C L114: they "partially occupy the self-hosted slot".
- The thesis gap ("a self-hosted control plane", L8) reads stronger than R3 allows if the OSS field is framed as desktop-only.
- Fix: "Free OSS conductors, several with servers or self-host, contest the team segment and partly occupy the self-hosted slot (R1 §3b, §5; R2 §5; R3 §C)". PHASE-1 L21 uses the same "desktop tools" wording (handoff).

### A4 [ADVISORY] Ona is closed to new customers per R2

- L16: "Coder and Ona sell governed agent platforms (R1 §3c, §4; R3 §4)".
- R2 §4 L68: Ona's "current official docs say **new-customer signup is closed**"; R2 Learnings L311 records this correction.
- VISION faithfully reports R1 §4 L181, so this is a cross-doc drift, not a miscitation.
- Fix (+40 B): append "(Ona closed to new customers, R2 §4)". Handoff to R1.

### A5 [ADVISORY] Sub-claims cited to the wrong sibling section

- L8 "(R1 §3a)": "every agent vendor ships a surface" is R1 §4 L175; "Teams increasingly run" has no measuring source. Fix: cite "R1 §3a, §4"; drop "increasingly".
- L17: "not the UI" is a gloss; R1 §2 L88 records UI polish ("beautiful clean UI, great UX") as something users value. Fix: drop "not the UI".
- L31 "(R1 §3d)": "may use one as a substrate" is R1 Analysis L225. Fix: "(R1 §3d, Analysis)".
- L34 "(R3 §C, §D)": the quoted "a better Conductor UI" is R3 §A L87. Fix: "(R3 §A, §C, §D)".
- L42: "at most two Git hosts" is R2 "Three-month scope" L166, not the options analysis. Fix: add "three-month scope".
- L67 "(R3 §G item 7)": item 7 (R3 L183) names Anthropic, OpenAI and Google; Cursor comes from R1 Open questions L264 and R2 Open questions L189. Fix: add "R2 Open questions".

### A6 [ADVISORY] "Adapters normalize one capability set" overstates parity

- L50.
- R2 Analysis L88: "interfaces have unequal capabilities: normalize `start`, `events(cursor)` ... `capabilities` ...".
- R2 §2 L43: "negotiate capabilities instead of assuming total parity".
- R2 says "not a platform job specification" of ACP only (L88); MCP and A2A are described by what they do not standardize (L49). "none a job specification" is a fair synthesis.
- Fix: "Adapters implement one normalized interface and declare per-agent capabilities; parity is never assumed".

### A7 [ADVISORY] Edition-boundary wording is internally loose

- L55: "Core includes two adapters"; L38: "all adapters are core"; R2 L110: "at least two adapters". Fix: "at least two adapters".
- L10 promises customizing "branding" with no edition; L38 and R3 §B L102 put branding in Enterprise. Fix: "branding (Enterprise)".

### A8 [ADVISORY] Adopted recommendations drop their confidence labels

- L8 gap: R1 L221 and R3 §C L118 both label it MED; VISION does not.
- L38 licence: R3 Recommendations L193 MED.
- L40 wedge: R3 §D L130 HIGH that option 1 is viable, MED that it beats 3 or 4.
- L42 "Build option B": R2 L162 calls it "Advisory preference (MED)"; "Blocking items cannot rank survivors". R2 also eliminates "D first" (L144, L162: "later hosted mode can reuse C"), while L42 reads as a permanent elimination.
- L10 "hypothesis, MED" has no one-clause reason (BRIEF rule 3); R2 Open questions L191 shows the scope of "onboard in an hour" is undefined.
- Fix: one bracket each, for example "(R2 advisory preference, MED)", "D eliminated for the first release"; give L10's MED a reason or lower it to LOW.

### A9 [ADVISORY] "How we will know" drops two R3 decision rules

- R3 Recommendations L197 (HIGH): "Run the seven §G experiments before any HLD is approved". VISION commits only item 1 before HLD (L60); items 4 and 7 appear as open questions (L72, L67); items 2, 3, 5 and 6 are absent with no reason.
- R3 §G L173 day-90 rule ("continue if activation ≥ 40% and multi-adapter share ≥ 20% and ≥ 2 paid pilots; one miss buys 30 days; two misses mean re-scope") is the actual continue or stop test for L62's indicators; it is omitted.
- Fix: one sentence each, or record the decision to defer items 2, 3, 5 and 6 with a reason.

### A10 [ADVISORY] Naming: USPTO was knock-out checked; EUIPO and UKIPO never were

- L44: "Unchecked: USPTO, EUIPO and UKIPO clearance".
- R3 §5 L64, L68, L69: USPTO returned 0 records for CUELIST, "CUE LIST" and KAPELL (reviewer-checked); "EUIPO and UKIPO: not checked for any name".
- R3 §E L139 still requires full USPTO, EUIPO and UKIPO searches before announcing, so "clearance" is correct; the distinction is lost.
- R3 §E L137 records cuelist's weak distinctiveness con ("cue list" is a show-control term of art); VISION omits it.
- Fix: "USPTO knock-out 0 records; full clearance and EUIPO/UKIPO searches pending; cuelist has weak distinctiveness".

### A11 [ADVISORY] Principle continuity could not be verified against v0.1

- L48 claims "none was dropped"; v0.1 exists only in git history, which this reviewer may not read.
- Indirect confirmation: R2 L114 quotes v0.1 "agent-agnostic by contract" (L41), an optional sandbox (L43 to L44) and "self-host first, hosted later" (L47), matching three of VISION's five "kept" principles.
- "Policy as configuration" and "Customizable, not forkable" as v0.1 names: UNVERIFIED by this reviewer.
- v0.1 marked its principles "to be challenged in HLD-000" (quoted at R3 review L170); v1 does not say whether that still holds.
- Fix: the lead diffs v0.1 against v1 Design principles before APPROVE; VISION adds one clause on whether principles remain open to HLD-000 challenge.

### A12 [ADVISORY] Form: multi-fact bullets, and a 2-byte margin

- Size 10,998 B, inside 7,000 to 11,000, with 2 B headroom: every fix above must be funded by cuts.
- Bullets that pack several facts: L15 (three), L16 (three), L17 (five), L18 (four), L34 (three). Principle bullets are compound by design and acceptable.
- No em-dashes or en-dashes (grep: 0); the only non-ASCII character is `§`.
- Funding: A1 frees 202 B; "How this repo works" (596 B) largely repeats BRIEF and PHASE-1 and can shrink to two lines; "Changes from v0.1" (601 B) can shrink to the four changes without restating the thesis.
- Fix: split L17 and L18 into one-fact bullets after the cuts.

### Checks that passed

- (d) Thesis discipline: no statement or implication that Conductor shut down. L8 "reversed the shutdown on 2026-11-12/13" and "announced ... winding down" are correct usage (R3 §4 L54 quotes "winding down"); L14 rejects the "stranded Conductor users" wedge.
- (f) Rule 5: grep for URLs, `@`, `~/`, `/Users`, `.com`: 0 hits; no personal names or handles; the operator appears only as "the maintainer" (L73). Company names (Stripe, Ramp, Spotify, Meta) are public sources already in R1 and R3.
- (b) Audience order 1 platform/DevEx, 2 teams, 3 B2B2B with 3a/3b: matches R1 §5 L197 and R3 §A.
- (b) Licence: Apache-2.0 core plus commercial source-available `enterprise/`: matches R3 §B L100.
- (b) Architecture: B inside C's boundaries, A and D eliminated, E survives and is spiked: matches R2 L162.
- (b) Kill test: 10 to 15 calls, half with in-house harnesses, two-page spec, PASS ≥ 3 letters AND ≥ 2 budget lines, FAIL re-scope or stop: verbatim R3 §G L177.
- (b) Naming: baton unusable, cuelist then kapell, trademark clearance and `.io` open: matches R3 §E L134 to L139 and Open questions L206 (nuance A10).
- (e) The two added principles trace: SHA-bound fail-closed gates to R2 Analysis L90 and Recommendations L179 (HIGH); open core runs task -> run -> diff -> gate -> PR to R2 Analysis L110 and R3 §B L102.

## Citation spot-check table

One row per citation group (47). Verdicts: S supports, P partly, N not there, C contradicts.

| # | Citation (VISION line) | VISION claim | What the section says | Verdict |
|---|---|---|---|---|
| 1 | R3 §E (L3) | `baton` not usable as a product name | L134: "not recommended (HIGH) ... Codename only"; live USPTO BATON marks | S |
| 2 | R1 §3a (L8) | teams increasingly run parallel agents on isolated copies; every vendor ships its own surface | §3a L100 to L118 lists Claude, Codex, Cursor, Gemini, Devin, Factory surfaces; "every major agent vendor ships its own surface" is R1 §4 L175; no source measures "increasingly" | P |
| 3 | R1 §1; R3 §4 (L8) | alive, proprietary, vendor-owned; joined Meta and winding down 11-04; reversed 11-12/13; builds inside Meta; no shipped self-host, GitHub-only cloud, unsandboxed local | R1 L54 to L58 dates and reversal, L55 "here at Meta", L42 self-host "Not yet", L40 GitHub only, L23 "without sandboxing"; R3 L54 "winding down"; R3 L55 ownership UNVERIFIED; "proprietary" is R1 §3e and R3 §C, not §1 or §4 | P |
| 4 | R1 §5; R3 §C (L8) | Agent HQ strongest cross-vendor incumbent: multi-vendor, Copilot-bundled, enterprise policies, audit logging, self-hosted runners, one code host's cloud | R1 L201 "strongest ... cross-vendor ... enterprise AI controls ... self-hosted runners"; R3 L111 multi-vendor, bundled, policies, "promising audit logging", "inside one code host's cloud" | S |
| 5 | R1 §3b, §5 (L8) | free OSS desktop tools contest the team segment | §3b L120 to L133 catalogue; §5 L206 "strong free OSS options" (wording: A3) | S |
| 6 | R1 Analysis; R3 §C (L8) | the three-part gap | R1 L221 to L224 and R3 L118, near verbatim (both MED) | S |
| 7 | R1 §1, §2, Recommendations O2a (L14) | incumbent shipping; stranded wedge BLOCKING | R1 L58 releases; L91 trust damage; L237 "[BLOCKING] the premise is false" | S |
| 8 | R1 §3a, §5 (L15) | Agent HQ hosts Claude and Codex in preview; Claude and Cursor customer-side execution; Cursor loop in vendor cloud | R1 L112, L102, L109; L201 | S |
| 9 | R1 §3c, §4; R3 §4 (L16) | build-from-scratch partly true; Stripe, Ramp, Spotify built; Coder and Ona sell; Coder governance paid | R1 L185 to L187, L136 to L138, L180 to L181; R3 L60 (Ona drift: A4) | S |
| 10 | R1 §2, §3b, §5; R3 §3 (L17) | commoditised (20+ tools, 3-day rebuild, two failures); $50 to $80 per seat; losses Cloud, API, multi-machine, not UI | R1 L83 rebuild; L127 to L128 failures; L205 "$50–80" team plans incl. Devin $80/mo; R3 L41 Devin is a "Per team flat fee"; R1 L88 UI polish valued | C |
| 11 | R2 §1 (L18) | checkout is not isolation; worktrees share network and credentials; containers share kernel; microVMs stronger, not proof | R2 L7 heading, L9, L11, L33, L36 | S |
| 12 | "R1 §5: real, MED" (L24) | audience 1 confidence | §5 L199 to L202 evidence only; verdict at R1 Recommendations L256 | N |
| 13 | R3 §A (L24) | payer; comparables monetise; named teams built | R3 L81, L87 | S |
| 14 | "R1 §5: real but contested, MED" (L25) | audience 2 confidence | §5 L204 to L207 evidence only; verdict at R1 L257 | N |
| 15 | R1 Recommendations O2b; R3 §A (L25) | user and funnel; free OSS and bundles cap the price | R1 L239 "OSS install as the funnel", L240 crowded, L257 "cap the price"; R3 L87 "Audience 2 is the user" | S |
| 16 | R3 §A (L26) | split into 3a and 3b | R3 L83 to L85 | S |
| 17 | "[MED, absence of evidence]" under L22's "R1 confidence" (L26) | 3a not a launch audience | R3 L84 "(MED: absence of evidence.)"; R1 rates audience 3 "not evidenced — LOW" (L258) | P |
| 18 | R1 §5; R3 §A: LOW (L26) | 3b later channel, direct conversations, LOW | R1 L212 "remains LOW"; R3 L85 "A later channel, tested by direct conversations ... (LOW" | S |
| 19 | R1 §3d (L31) | not a workflow engine; may use one as substrate | §3d L141 to L144 the class; "possible substrate" is R1 Analysis L225 | P |
| 20 | R3 §A, Recommendations (L33) | not a dev-environment platform or sandbox provider | R3 L87 "not a dev-environment platform", pluggable providers; L194 integrate as providers | S |
| 21 | R3 §C, §D (L34) | not hosted-first, not "a better Conductor UI"; invest in policy, gate, audit, adapters | R3 L120 UI commoditised and invest; L125 hosted-first BLOCKING; the quoted phrase is R3 §A L87 | P |
| 22 | R3 §B (L38) | Apache-2.0 plus source-available `enterprise/`; FSL, BSL, ELv2, Sustainable Use BLOCKING; SSPL BLOCKING (non-OSI) | R3 L96 to L100 | S |
| 23 | R3 §B (L38) | core vs Enterprise split | R3 L102 (VISION omits policy packs with approvals and cross-vendor cost accounting) | S |
| 24 | R3 §F, Handoffs (L38) | DCO, no CLA | R3 L159 (Q3), L212 | S |
| 25 | R3 §D (L40) | sequence; hosted-first and closed-first BLOCKING | R3 L124 to L130 | S |
| 26 | R3 §B, Recommendations (L40) | pricing per conducting user, sandbox-hours, no usage metering self-hosted | R3 L104, L195 | S |
| 27 | R2 options analysis and verdict (L42) | B inside C; A, D eliminated; E spiked; first slice two agents, two executors, at most two Git hosts | R2 L118 to L162; L136, L181 two and two; "at most two Git hosts" is L166 (three-month scope); D eliminated "first" (L162); B is an advisory preference (MED) | P |
| 28 | R3 §E, Open questions (L44) | cuelist lead, kapell weak second; clearance and `.io` open | R3 L136 to L139, L206 (nuance A10) | S |
| 29 | R2 §2, Analysis; R3 §C (L50) | one capability set; ACP, MCP, A2A complementary; API-key and enterprise billing | R2 L49 complementary; L50 and R3 L120 API-key billing; R2 L88 "unequal capabilities", L43 no "total parity" | P |
| 30 | R2 §1, Analysis (L51) | one run, one checkout; `trusted-local`; sandbox never mounts host checkouts or credentials | R2 L86, L116 | S |
| 31 | R2 Analysis, Recommendations (L52) | SHA-bound, fail closed; two gate kinds; no self-approve or merge | R2 L90; L179 (HIGH); L94 deny on policy or audit failure | S |
| 32 | R2 Analysis; R3 §B (L53) | declarative per org and repo; append-only audit, not a feed; both open | R2 L94, L96, L183; R3 L102 | S |
| 33 | R2 Analysis; R3 §D (L54) | one binary plus Postgres in Compose, not infrastructure-free | R2 L102 (MED); R3 L124 | S |
| 34 | R2 Analysis; R3 §B (L55) | open core runs task -> run -> diff -> gate -> PR; adapters, backends, roles, events | R2 L110 ("at least two adapters", A7); R3 L102 | S |
| 35 | R2 §4, Analysis (L56) | versioned extension points; no untrusted plugins in server; no UI fork | R2 L66, L108 ("roles" is R3 §F L149) | S |
| 36 | R3 §G item 1 (L60) | kill test, PASS, FAIL | R3 L177 | S |
| 37 | R3 Recommendations (L60) | runs before any HLD is approved | R3 L197 all seven §G experiments before any HLD (scope narrowed: A9) | S |
| 38 | R3 §G (L62) | three leading indicators, opt-in denominator | R3 L165 to L169 | S |
| 39 | R3 §G item 1 (L66) | buy rather than build | R3 L177, L203 | S |
| 40 | R3 §G item 7 (L67) | Anthropic, OpenAI, Google and Cursor in writing | R3 L183 names Anthropic, OpenAI, Google only | P |
| 41 | R2 Open questions (L68) | two agents, two Git hosts; "onboard in an hour" scope | R2 L190, L191 | S |
| 42 | R2 Open questions (L69) | trust tier sets container or microVM default | R2 L188 | S |
| 43 | R2 verdict (L70) | B or E spike in HLD-000 | R2 L151, L162 | S |
| 44 | R3 Open questions (L71) | Agent HQ hosts and control plane; Conductor self-host date | R3 L201, L202 | S |
| 45 | R3 §G item 4 (L72) | 3b letter of intent | R3 L181 "≥ 2 letters of intent" ("latent" is R2 L193) | S |
| 46 | R3 §E (L73) | trademark clearance for both | R3 L139 | S |
| 47 | R3 §A (L88) | audience 3 split | R3 L83 to L85 | S |

Tally: 36 supports, 8 partly, 2 not there, 1 contradicts (47).

## What is strong

- Thesis discipline is exact: the incumbent is alive, the reversal is dated 2026-11-12/13, "reversed the shutdown" is used correctly, and nothing implies a shutdown.
- The gap is stated in R1's and R3's own words and is explicitly "narrower than 'nothing exists'".
- The kill test, PASS and FAIL are verbatim from R3 §G item 1; the three leading-indicator thresholds match R3 §G.
- Licence, core and Enterprise split, pricing, DCO without CLA and wedge sequence match R3 §B, §D and §F.
- The architecture stance reproduces R2's verdict, including E surviving and being spiked in HLD-000.
- Each principle is tagged kept, restated, extended or added, with a reason; "Self-host first" is made honest ("one product binary plus Postgres", R2 L102) instead of repeating R1's single-binary preference.
- Both new principles trace to approved R2 text with a HIGH or MED label.
- Every open question names the deliverable that answers it.
- Clean on Rule 5 and on dashes; 36 of 47 citation groups are fully supported.

## Handoffs to other docs (not VISION findings)

- R1: Recommendations O2b L239 "about $50–80 per seat" includes Devin's per-team $80 (R1 L115; R3 L41); same unit error as B1.
- R1: §4 L181 "Ona sells governed background agents" predates R2 §4 L68 (new-customer signup closed).
- R3: header L3 still reads "Status: DRAFT v4"; its Review log L290 records the lead's APPROVE and `docs/research/README.md` says APPROVED v4.
- PHASE-1: L3 still lists R2 as "DRAFT v2 (under re-verification)".
- PHASE-1: L21 repeats "free OSS desktop tools" (A3).
- PHASE-1: no row schedules R3's pre-HLD §G experiments, which R3 L197 (HIGH) requires before any HLD, including HLD-000, is approved.

## Learnings / mistakes

Reviewer's own run:

- My first `conversation_search` for the v0.1 principles used an unquoted multi-word query and matched unrelated threads on common words; caught on reading the hit list (I opened none of them); rule: quote exact phrases and scope the search to the project thread.
- I first read the L24 and L25 brackets as supported because R1 §5 is the audiences section; caught when the "real — MED" text turned up in R1 Recommendations L256; rule: check that the cited section holds the specific label or number quoted, not just the topic.
- I nearly accepted "$50 to $80 per seat" because R1 O2b says the same; caught when R3 §3 L41 classed Devin as a per-team flat fee; rule: check the unit of every price range, even when an approved doc repeats it.
- I first drafted B3 as "VISION never defines open"; re-reading L38 showed the OSI test is carried by its R3 §B citation, so I narrowed B3 to the customer invariant; rule: re-read the target line before claiming an omission.
- R3's header still says DRAFT v4; I took APPROVED v4 from its Review log L290 and the research README; rule: confirm a doc's status from its review log, not its header alone.
- The v0.1 text exists only in git history; I considered decoding `.git` objects without the git binary and rejected it as a workaround of the no-git rule, recording A11 as UNVERIFIED instead; rule: do not reimplement a prohibited tool to get around it.
- I created the skeleton early but ran about ten checks before the first content checkpoint; rule: write findings text every three or four checks.

Author's mistakes found (for the corpus, generic form):

- Copied a price range from a recommendation without checking the unit against the fact section (B1).
- Cited a confidence label to the evidence section instead of the verdict section, and let a second doc's label sit under a "first doc's confidence" convention (B2).
- Rewrote an invariant that a downstream approved doc names as the premise of its BLOCKING verdicts, without restating it (B3).
- Left "pending" labels after the upstream doc was approved; stale-status labels need a sweep before submission (A1).

## Re-verification (v2)

status: COMPLETE

Target: `VISION.md` DRAFT v2, 12,474 B total. Lines below are v2 lines (`L<n>`). `VISION.md` was not edited and git was not run.

### Per-finding status

| Finding | Tag | Status | Evidence |
|---|---|---|---|
| B1 | BLOCKING | closed | L20 now reads "Conductor Teams and Paradev Team $60 per user; Devin Teams $80 a month flat for up to 100 users (R1 §5; R3 §3)". R1 §5 L205 gives Conductor Teams $60/user, Paradev Team $60/user and Devin Teams $80/mo; R3 §3 L41 classes Devin as "Per team flat fee ... for up to 100 users". Units now match both sources; "per seat" is gone. Tense nuance: A14. |
| B2 | BLOCKING | closed | The v1 "R1 confidence in brackets" convention line is gone; each label is attributed in place. L25 "[R1 Recommendations: real, MED]" = R1 L256; L26 "[R1 Recommendations: real but contested, MED]" = R1 L257; L27 "[R1 Recommendations: not evidenced, LOW]" = R1 L258 (was missing); 3a "[R3 §A: MED, absence of evidence]" = R3 L84; 3b "[R3 §A: LOW]" = R3 L85. |
| B3 | BLOCKING | closed | L40 adds "Invariants (R3 §B, §D). Any company may offer baton to its own teams or to its customers; the core is under an OSI-approved licence; self-host first, hosted later." That matches R3 §B L91 (customer and open invariants) and R3 §D L125 ("self-host first, hosted later"). L10 now reads "for your teams and, later, your customers": "later" is sequencing, matching R3 §A L87 ("Audience 3 a later channel"), not a demand condition, and L40 states the right unconditionally. Residual wording: A13. |
| A1 | ADVISORY | closed | L4 "Inputs: R1 APPROVED v2, R2 APPROVED v2, R3 APPROVED v4" (= R2 L3). Grep of v2: no "R2 v2" and no "re-verification pending"; the only "pending" left (L48) is trademark clearance, and "re-verification" (L4) is VISION's own status. |
| A2 | ADVISORY | closed | L8 "(ownership and deal terms UNVERIFIED)", matching R3 §4 L55, R3 §C L113, R2 §5 L76. |
| A3 | ADVISORY | closed | L8 "Free OSS conductors, several with servers or self-host, contest the team segment and partly occupy the self-hosted slot (R1 §3b, §5; R2 §5; R3 §C)" = R2 L78 "not merely offline desktop apps", R3 L114 "partially occupy the self-hosted slot". |
| A4 | ADVISORY | closed | L18 "Ona is closed to new customers (R1 §4; R2 §4; R3 §4)" = R2 L68. Residual: the same bullet still says "Coder and Ona sell", which reads against R2 L68 (existing organisations only); folded into A12. |
| A5 | ADVISORY | closed | L8 "(R1 §3a, §4)", "increasingly" gone (= R1 L175); "not the UI" gone (grep: only in the Review log L105); L32 "(R1 §3d, Analysis)" (= R1 L225); L36 "(R1 Analysis; R3 §A, §C)" (quoted phrase = R3 L87); L46 "First slice (R2 Three-month scope)" (= R2 L166); L75 "R3 §G item 7; R2 Open questions" (Cursor = R2 L189). The bullet splits introduced two new misallocations: A15. |
| A6 | ADVISORY | closed | L54 "Adapters implement one normalized interface and declare per-agent capabilities; parity is never assumed" = R2 L88 ("unequal capabilities", normalize `capabilities`) and R2 L43. |
| A7 | ADVISORY | closed | L59 "at least two adapters" (= R2 L110); L10 "branding (Enterprise)" (= R3 L102). Consistent with L42 "Core: ... all adapters". |
| A8 | ADVISORY | closed | Labels now carried: gap L8 "both MED" (R1 L221, R3 L118); licence L42 "MED" (R3 L193); wedge L44 "viable HIGH; preferred MED" (R3 L130); pricing L44 "MED" (R3 L195); architecture L46 "advisory preference, MED" (R2 L162), with "D ... first" (R2 L162); L10 hour claim "LOW: scope undefined, open question 3" (R2 L191; OQ3 is L76). |
| A9 | ADVISORY | closed | L66 lists all seven R3 §G experiments in R3's order (R3 L177 to L183) under "R3 Recommendations, HIGH" (= R3 L197); L70 day-90 rule matches R3 L173 (40%, 20%, 2 paid pilots, 30 days, re-scope). |
| A10 | ADVISORY | closed | L48 "A USPTO knock-out search returned 0 records for both; full USPTO, EUIPO and UKIPO clearance is pending; `cuelist` has weak distinctiveness" = R3 L64 (EUIPO/UKIPO not checked), L67, L68 (USPTO 0), L137 (term of art), L139 (full searches). |
| A11 | ADVISORY | closed by lead | The lead diffed v0.1 (git) against v2: five v0.1 principles restated, two added, none dropped. v2 L52 adds "all stay open to challenge in HLD-000". Reviewer did not read v0.1 (no git); the seven principle tags (5 kept/restated/split/extended/qualified, 2 added) are consistent with the lead's count. |
| A12 | ADVISORY | partially | Split: v1 L15 into L15/L16, v1 L16 into L17/L18, v1 L17 into L19/L20, v1 L34 into L35/L36; "How this repo works" is one paragraph (L85); "Changes from v0.1" is four bullets. Still multi-fact: L18 (Coder sells; Coder's governance is paid; Ona closed) and L21 (unchanged from v1 L18: worktrees; containers; microVMs). Body 10,982 B, so 18 B headroom. Fix: split L18 into a Coder bullet and "Ona serves existing customers only (R2 §4)"; split L21 after "credentials". |

### Re-checked citations (new or changed in v2, and v1 partly / not there)

| # | Citation (v2 line) | v2 claim | What the section says | Verdict |
|---|---|---|---|---|
| r1 | R1 §3a, §4 (L8) [v1 #2 P] | teams run parallel agents on isolated copies, review and merge; every vendor ships its own surface | §3a L101 to L115 worktrees, isolated VMs, diff review and PRs; §4 L175 "Every major agent vendor ships its own surface"; "increasingly" dropped | S |
| r2 | R1 §1; R3 §4 (L8) [v1 #3 P, unchanged] | alive, proprietary, vendor-owned; 11-04 join and wind-down; 11-12/13 reversal; no self-host, GitHub-only cloud, unsandboxed local | dates, reversal, self-host, cloud and local all in R3 L50, L54, L55; "proprietary" is only in R1 table 3e L150 and R3 §C L113 | P (A15) |
| r3 | R1 §3b, §5; R2 §5; R3 §C (L8) [changed, A3] | OSS conductors with servers or self-host; contest the team segment; partly occupy the self-hosted slot | R1 L121 to L124 (T3 Code control plane, Orca own cloud, Paseo daemon and Hub, Herdr); R1 L206; R2 L78; R3 L114 | S |
| r4 | R1 Analysis; R3 §C; both MED (L8) [label added] | the three-part gap | R1 L221 to L224 "(MED, inferred ...)"; R3 L118 "(MED)" | S |
| r5 | R1 §1, §2, Recommendations O2a (L14) [O2a label checked] | incumbent shipping; stranded wedge carries a BLOCKING con | O2a (L235) is the "Conductor refugees" option; L237 "[BLOCKING] the premise is false"; §1 releases; §2 L91 trust | S |
| r6 | R1 §5 (L15) [new by split] | Agent HQ hosts Claude and Codex in public preview; not every vendor surface is single-vendor | §5 L201 says only "cross-vendor"; "Claude and Codex ... in public preview" is §3a L112 and §4 L176; "Not every vendor surface is vendor-locked" is Analysis L219 | P (A15) |
| r7 | R1 §3a, §5 (L16) [new by split] | Claude and Cursor offer customer-side execution; Cursor's loop stays in the vendor cloud | L102, L109; §5 L201 | S |
| r8 | R1 §3c, §4; R3 §4 (L17) [new by split] | "build from scratch" only partly true; Stripe, Ramp, Spotify built in-house | R1 §4 L185 to L187; "only partly true" is R1 Analysis L220; R3 §4 (L48 to L61) has no Stripe, Ramp or Spotify; R3 §A L81 does | P (A15) |
| r9 | R1 §4; R2 §4; R3 §4 (L18) [changed, A4] | Coder and Ona sell; Coder's governance paid; Ona closed to new customers | R1 L180, L181; R2 L67, L68; R3 L60 | S |
| r10 | R1 §2, §3b, Analysis (L19) [new by split] | local conductor commoditised: 20+ tools, 3-day rebuild, two monetisation failures | R1 L217 (all three, HIGH); L83 rebuild; L127, L128 shutdowns; O2b L240 "documented monetisation failures" | S |
| r11 | R1 §5; R3 §3 (L20) [v1 #10 C, B1] | Conductor Teams and Paradev Team $60 per user; Devin Teams $80 a month flat for up to 100 users; "Teams still pay" | R1 L205 all three prices, units match; R3 L41 "Per team flat fee"; R1 L205 hedges "existed or exist" and R3 L50 says Conductor's pricing page is archived | P (A14) |
| r12 | R1 Recommendations: real, MED (L25) [v1 #12 N, B2] | audience 1 confidence | L256 "real — MED" | S |
| r13 | R1 Recommendations: real but contested, MED (L26) [v1 #14 N, B2] | audience 2 confidence | L257 "real but contested — MED" | S |
| r14 | R1 Recommendations O2b; R3 §A (L26) [O2b label checked] | the user and adoption funnel; free OSS and vendor bundles cap the price | O2b (L238) is "team governance"; L239 "OSS install as the funnel"; L240 "crowded with free tools and vendor bundles"; "cap the price" is the L257 verdict in the same section; R3 L87 | S |
| r15 | R1 Recommendations: not evidenced, LOW (L27) [new, B2] | audience 3 confidence | L258 "not evidenced — LOW" | S |
| r16 | R3 §A: MED, absence of evidence (L27) [v1 #17 P] | 3a not a launch audience | L84 "Not a launch audience. (MED: absence of evidence.)" | S |
| r17 | R3 §A: LOW (L27) | 3b later channel, direct conversation | L85 "(LOW: two self-reported signals, no identified buyer.)" | S |
| r18 | R1 §3d, Analysis (L32) [v1 #19 P] | not a workflow engine; may use one as a substrate | §3d L141 to L144; Analysis L225 "a possible substrate" | S |
| r19 | R3 §D (L35) [new by split] | not a hosted-first cloud | L125 hosted-first "[BLOCKING]"; L130 | S |
| r20 | R1 Analysis; R3 §A, §C (L36) [v1 #21 P] | not "a better Conductor UI"; UI commoditised; invest in policy, gate, audit, adapters | R3 L87 quoted phrase; R3 L120 "treat the UI as commoditised and invest in the policy/gate/audit layer and adapters"; R1 L217 "local conductor is commoditised" | S |
| r21 | R3 §B, §D (L40) [new, B3] | customer, open and self-host-first invariants | L91 customer and open (OSI-approved); L125 "self-host first, hosted later"; L128 "open" as defined in §B | S |
| r22 | R3 Handoffs (L40) [new, B3] | eliminations "rest on these and reopen only if they change" | L212 "rest on VISION v0.1 invariants and reopen if those change"; "only" is not in R3, and R3 L91 adds they are conditional on principles "to be challenged in HLD-000" | P (A13) |
| r23 | R3 §B; Recommendations, MED (L42) [label added] | Apache-2.0 plus source-available `enterprise/`; four licences fail the customer invariant, SSPL the open one | L100 verdict; L193 "MED — Licence" | S |
| r24 | R3 §D: viable HIGH; preferred MED (L44) [label added] | wedge sequence | L130 "(HIGH that 1 is viable ...; MED that 1 beats 3 or 4 ...)" | S |
| r25 | R3 §B; Recommendations, MED (L44) [label added] | pricing | L104; L195 "MED — Price Enterprise per conducting user" | S |
| r26 | R2 options analysis verdict; advisory preference, MED (L46) [v1 #27 P] | B inside C; A standalone and D first eliminated; E spiked against B in HLD-000 | L162 "eliminate A standalone and D first; B, C and E remain viable ... Advisory preference (MED): B first, within C's boundaries"; L151 "spiked against B during HLD" | S |
| r27 | R2 Three-month scope (L46) [new] | first slice: two agents, two executors, at most two Git hosts | L166 "at most two Git hosts"; L169 worktree `trusted-local` and hardened container; L170 "two production-path adapters" | S |
| r28 | R3 §5, §E (L48) [changed, A10] | USPTO 0 for both; full and EUIPO/UKIPO pending; cuelist weak distinctiveness; `.dev` or `.io` open | L64, L67, L68, L137, L139; `.io`/`.dev` status L137 (the open question itself is R3 L206) | S |
| r29 | R2 §2, Analysis; R3 §C (L54) [v1 #29 P] | one normalized interface, declared capabilities, no parity; ACP, MCP, A2A complementary; API-key and enterprise billing | R2 L88, L43, L49; R3 L120 "API-key and enterprise billing from day one" | S |
| r30 | R3 Recommendations, HIGH (L66) [v1 #37 narrowed] | all seven §G experiments before any HLD | L197; the seven names match L177 to L183 in order | S |
| r31 | R3 §G (L70) [new] | day-90 rule | L173, exact | S |
| r32 | R3 §G item 7; R2 Open questions (L75) [v1 #40 P] | Anthropic, OpenAI, Google and Cursor in writing | R3 L183 Anthropic, OpenAI, Google; R2 L189 Anthropic, OpenAI, Cursor, Kiro | S |
| r33 | R2 options analysis verdict (L78) | B or E HLD-000 spike | L151, L162 | S |
| r34 | R3 §5, §E (L81) [changed] | full clearance for both; `.dev` or `.io` | L139; L137 | S |

Not tallied: L10 "open question 3" is an internal cross-reference; OQ3 (L76) does ask what "onboard in an hour" includes.

### New findings

No new BLOCKING finding. Three new ADVISORY findings; their fixes net to -3 B (python byte count of each old and new string), so the body stays under 11,000 B.

**A13 [ADVISORY] L40 "reopen only if they change" is stronger than R3**

- L40: "The licence and wedge eliminations below rest on these and reopen only if they change (R3 Handoffs)."
- R3 Handoffs L212: "rest on VISION v0.1 invariants and reopen if those change". R3 §B L91 adds that both eliminations "are conditional on VISION v0.1, whose principles are 'to be challenged in HLD-000'".
- "only" is not in R3. It also sits awkwardly with L52 ("all stay open to challenge in HLD-000"), since "self-host first, hosted later" is both an invariant (L40) and a principle (L58).
- Fix (-5 B): delete "only".

**A14 [ADVISORY] L20 "Teams still pay" is in the present tense; R1 hedges it**

- L20: "Teams still pay: Conductor Teams and Paradev Team $60 per user; Devin Teams $80 a month flat for up to 100 users (R1 §5; R3 §3)."
- R1 §5 L205: "team plans existed or exist at $50–80". R3 §4 L50: Conductor's pricing is from an archive of 2026-09-29, and "the live page redirects to the homepage". R1 L51: Conductor Teams is "invite-only".
- The units are now correct (B1 is closed); only the tense goes beyond the sources for Conductor.
- Fix (+12 B): "Team plans exist or existed:" in place of "Teams still pay:".

**A15 [ADVISORY] The v2 bullet splits left two citations on the wrong sibling section; one v1 partial remains**

- L15 "(R1 §5)": §5 L201 says only "cross-vendor". "hosts Claude and Codex in public preview" is R1 §4 L176 (and §3a L112); "not every vendor surface" is R1 Analysis L219. In v1 this sentence shared "(R1 §3a, §5)" with L16. Fix (+10 B): "(R1 §4, Analysis)".
- L17 "(R1 §3c, §4; R3 §4)": R1 §4 L185 to L187 hold Stripe, Ramp and Spotify; "only partly true" is R1 Analysis L220; R3 §4 (L48 to L61) has none of it, while R3 §A L81 does. Fix (+4 B): "(R1 §4, Analysis; R3 §A)".
- L8 "(R1 §1; R3 §4)" (v1 #3, partly, unchanged): "proprietary" is in R1 table 3e L150 and R3 §C L113, not §1 or §4. "which defined the category" is not stated in R1 or R3 (it may be carried over from v1; I did not flag it then). Fix (-24 B): delete ", which defined the category," and cite "(R1 §1; R3 §4, §C)".
- These are partial citations, not "not there": in each group at least one cited section holds part of the claim, so they are graded P, as in v1 (rows 19, 21, 40).

### Rule 5 and form

- Em-dashes 0, en-dashes 0; the only non-ASCII character is `§` (python scan).
- Rule 5: no `http://`, `https://`, `www.`, `@`, `~/`, `/Users` or `.com` (0 hits); no personal names or handles; "the maintainer" is gone. Repo paths in L85 are repo-relative, and `docs/design/TEMPLATE-HLD.md` exists (ls).
- Size: body before `## Review log` is 10,982 B (in range, 18 B headroom); total with the 15-row Review log is 12,474 B. I applied the lead's "body under 11,000 B" convention; if the bound applied to the whole file, it would fail.
- Bullets: all split except L18 and L21 (A12, partially). Principle bullets stay compound by design.
- Thesis discipline (d) re-checked: L8 "reversed course on 2026-11-12/13", L14 "still shipping"; nothing implies a shutdown.
- Hypotheses labelled: L10 "hypothesis, LOW" with a reason; audiences, gap, licence, wedge, pricing and architecture all carry labels (A8).

### v1 handoffs, status on disk

- R1 O2b L239 "about $50–80 per seat": still open (R1 not revised).
- R1 §4 L181 Ona: still open.
- R3 header L3: closed ("Status: APPROVED v4").
- PHASE-1 L3 R2 status: closed ("R2 APPROVED v2 (2026-11-25)").
- PHASE-1 L21 "desktop tools": closed ("free OSS desktop tools and OSS servers").
- PHASE-1 §G experiments: partially. L29 now asks VISION for "R3 §G's other experiments and the day-90 rule", but I found no row that schedules running them before HLD-000.

### Updated citation tally

- v2 has 56 citation groups (47 in v1; the bullet splits and the new invariants, day-90 and label citations add groups). I re-checked 34 groups (rows r1 to r34). The other 22 keep their v1 citation and claim (some reworded, for example L59 per A7, checked in the A-rows); all 22 were S in v1, and their v2 text still matches the v1 table rows.
- Tally: 51 supports, 5 partly (r2, r6, r8, r11, r22), 0 not there, 0 contradicts.
- v1 was 36 / 8 / 2 / 1 (47). All v1 "not there" and "contradicts" grades are cleared; 7 of 8 v1 partials are cleared (v1 #3 remains as r2); r6, r8, r11 and r22 are new partials from v2 text.

### Final verdict

APPROVE: B1 to B3 are closed on disk; A1 to A11 are closed (A11 by the lead's v0.1 diff); A12 is partially done; new A13 to A15 are ADVISORY, bounded and fundable within the 18 B headroom, and no new BLOCKING finding was found.

### Learnings / mistakes (re-verification run)

- I first planned to grade L17 "(R1 §3c, §4; R3 §4)" as supported because v1 row 9 had graded the combined bullet S; I caught it when an R3 grep for Stripe found no hit in §4; rule: re-grade every citation that a bullet split re-attached, not only the sentences whose text changed.
- My v1 row 9 note "R3 L60 (Ona drift: A4)" read as if R3 mentions Ona; an R3 grep found no "Ona" anywhere, so R3 §4 supports only the Coder half of L18 (R1 §4 and R2 §4 carry Ona); rule: write each line reference so it names what it supports.
- My working notes put R2's weeks 3 to 5 and 6 to 8 at L168 and L169; a line print showed L169 and L170 before I wrote row r27; rule: print the line before quoting its number.
- "which defined the category" (L8) is unsourced and may have been in v1, where I did not flag it; rule: on re-verification, read the whole thesis again, not just the changed clauses.
- My first byte deltas for the A15 fixes were estimates (+11, +1, -23); a python count of each old and new string gave +10, +4, -24, and I corrected them; rule: compute byte budgets, never estimate them.
- My first draft of these learnings had a "nearly passed the size check" item that did not happen (I computed body and total in the same step); I removed it after auditing it against my steps; rule: audit each learning against what was actually run.
- Author's mistakes (generic form, for the corpus): splitting a multi-fact bullet without re-allocating its citations per new sentence (A15); strengthening a source's "if" into "only if" when restating it (A13); making a hedged source's tense present (A14).
