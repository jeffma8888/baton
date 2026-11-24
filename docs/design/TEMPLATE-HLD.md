# <System / Capability> HLD

> HLD = architecture decisions, system boundaries, options analysis, data flows.
> No class names, config keys, or test file names (those belong in the LLD).
> Every decision cites its source (research doc §, measurement, or "verified-by: <how>").

## Background
Why this exists. The pain, the prior attempts, what changed. 2–4 paragraphs.

## Glossary
| Abbreviation | Full form | Description |
|---|---|---|

## P0 Scope
What the first launch includes and why it is limited to that.

## Requirements
Numbered. Functional first, then non-functional (latency, scale, availability, security,
cost, tenancy). Each requirement traceable to VISION.md or a research doc.

## Out of Scope
Numbered. Say where each excluded item will be designed instead.

## Assumptions
Numbered. Each one is a thing that, if false, changes the design.

## Design

### 1. End-to-end workflow
Diagram (mermaid) + a numbered step table:
| # | Description |
|---|---|

### 2. <Component / decision area>
State the question this section decides.

**Option 1: <name> (Preferred)**
Flow / diagram. Then:
**Pros:** numbered.
**Cons:** numbered. Label each con `[BLOCKING]` or `[ADVISORY]`.
**Why is this option preferred?**

**Option 2: <name>**
Same structure. For a rejected option, the verdict must follow from BLOCKING items alone;
state explicitly: "infeasible because of <X> alone; <Y> is advisory and overruling it does
not change this verdict."

**Comparison table**
| Criterion | Option 1 | Option 2 | Option 3 |
|---|---|---|---|

### 3. ... (repeat per decision area)

### N. Cross-cutting
Security & isolation model · Multi-tenancy · Failure modes and recovery · Observability ·
Cost model · Migration/rollout.

## Appendix
Deep dives, data, worked examples.

## References
Numbered, with URLs. Research docs, primary sources, prior art.

## Action Items / Open Questions
Numbered, each with an owner and the decision it blocks.

## Review log
| Date | Reviewer (model) | Verdict | Summary of changes requested |
|---|---|---|---|
