# 0005: Product calls go through Review and Recommend; ADRs record the "no"s

- **Status:** accepted
- **Date:** 2026-11-22

## Context

The course splits the work: agents take the tickets, and humans own what gets
built (E1). Kent's Review and Recommend skill gives numbered options with
effort and door type, a recommendation, and the *right* fix with the reason
it differs. Kody's ADRs are written after the decision, are mostly
product-shaped "no"s, and include a revisit-if.

## Decision

- Every product or architecture question goes through
  [runbooks/review-and-recommend.md](../runbooks/review-and-recommend.md).
  A human chooses.
- Write an ADR (`docs/decisions/0000-template.md`) when the next
  agent would otherwise propose the rejected option again. Skip one for
  two-way tweaks and for library picks the code already encodes.
- Each ADR adds two fields to Kody's template:
  - **Refutation test:** what observation within N days would show the
    decision was wrong. For product bets it becomes the bet's threshold
    ([0024](./0024-outcomes-measured-per-bet.md)).
  - **Reversibility plan** for one-way doors: a flag, a dual-write window,
    or a codemod.
- Link the ADR from the `adrs` list of each primitive it shapes.

## Consequences

- Decisions become falsifiable claims, not preferences.
- Agents stop re-arguing settled questions.

## Revisit if

The same decision gets reopened more than twice. That usually means the
revisit-if was wrong or the context changed.
