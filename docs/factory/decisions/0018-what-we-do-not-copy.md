# 0018: What we do not copy from Kody

- **Status:** accepted
- **Date:** 2026-11-22

## Context

Some of Kody's machinery fits a solo-founder platform with over a million
lines of code and per-user isolation. It would be cost without benefit in a
new app.

## Decision

Do not copy:

- **The 35-leg validate on day one.** Grow the gate per
  [0007](./0007-one-validate-gate.md).
- **The DR control plane** (separate DR account, signed manifests, paced D1
  lanes). A tested weekly restore drill delivers most of the value.
- **In-house feature flags.** That was Kent's call in Kody ADR 0007. Keep
  the forced-expiry rule and use whichever implementation is cheapest.
- **Two live copies of our own tooling.** One live copy per job, and the
  other gets a dated removal.
- **Implementer-declared risk, same-vendor review, self-granted merge
  authority, and agents pushing as a person.** See
  [0020](./0020-risk-and-merge-enforced-by-github.md),
  [0021](./0021-reviewer-diversity-is-a-measured-hypothesis.md), and
  [0022](./0022-break-the-lethal-trifecta.md).
- **Per-PR outcome checks and day-one canaries.** See
  [0023](./0023-containment-scaled-to-traffic.md) and
  [0024](./0024-outcomes-measured-per-bet.md).
- **A single named decision-maker.** Route decisions by primitive owner
  ([0006](./0006-primitives-after-they-exist.md)).

## Consequences

The factory stays small enough to maintain alongside client work.

## Revisit if

An app grows to platform scale: multiple services, tenant isolation, or data
you cannot lose.
