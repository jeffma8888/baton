# Phase 1 — from research to an approved platform design

> Status: COMPLETE 2026-11-26 on document quality (all three rows approved); the R3 §G experiment hold on HLD-000 stands until the maintainer records results. Started 2026-11-25. Inputs: R1 APPROVED v2, R2 APPROVED v2 (2026-11-25), R3 APPROVED v4.
> Open advisory findings on R1 (F13) and R2 (F13-F15) are listed in each doc's status line.

## Deliverables, in order

| # | Deliverable | File | Author model | Reviewer model | Depends on | Done when |
|---|---|---|---|---|---|---|
| 1 | VISION v1 | `VISION.md` (replaces v0.1 in place; git keeps v0) | Fable | Opus | R1, R3, R2 verdict | DONE: APPROVED v2 2026-11-26 (A12-A15 advisory open) |
| 2 | PRD-000 product requirements | `docs/design/PRD-000-platform.md` | GPT | Fable | VISION v1 | DONE: APPROVED v2 2026-11-26 (F22-F25 advisory open) |
| 3 | HLD-000 platform overview | `docs/design/HLD-000-platform-overview.md` | Opus | GPT | PRD-000, R2 APPROVED | DONE: APPROVED v3 on document quality 2026-11-26 (3 review cycles, 22 findings closed); §G experiment HOLD stands |

Each deliverable follows the Review protocol in `BRIEF.md`: author writes, a different model
reviews into `<doc>.review.md`, author revises with a `## Review log`, reviewer re-verifies,
the lead flips the status line. Nothing in `docs/plans/` is written before HLD-000 is approved.

## What VISION v1 must settle (from the research)

- Thesis: the incumbent is alive, proprietary and vendor-owned; the cross-vendor incumbent is
  GitHub Agent HQ; free OSS desktop tools and OSS servers contest the team segment (R1 §1, §2, §5; R2 §5).
- Audiences in canonical order: 1 platform/DevEx, 2 engineering teams, 3 B2B2B, with R1's
  confidence and R3's 3a/3b split of the third.
- Licence: Apache-2.0 core + source-available enterprise module; FSL/BSL/ELv2/SSPL blocking
  for the core (R3 §B). The open core must run the defining workflow end to end (R2).
- Wedge and sequence: OSS self-hosted team server + thin client, then hosted, embeddable API
  last (R3). Architecture stance: build option B inside option C's boundaries; A and D
  eliminated (R2 verdict).
- Kill test for the payer hypothesis and its PASS criteria, plus R3 §G's other experiments and the day-90 rule (R3).
- Naming: `baton` is a codename and is not usable as a product name; leading candidate and
  what is still unchecked (R3 naming section).

## Scheduled alongside Phase 1

R3 §G's pre-HLD experiments (payer kill test first) are the maintainer's to run; they are
not authored by teammates. HLD-000 records which have been run and their results at the time
of writing, and marks assumptions that depend on an unrun experiment.

## Open after Phase 1

Component HLDs (control plane, execution/sandbox adapters, policy and gates, tenancy),
LLDs, and `docs/plans/PLAN-000`.
