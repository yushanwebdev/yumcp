# 0016: Every decision and work item names an outcome metric

- **Status:** superseded by [0024](./0024-outcomes-measured-per-bet.md)
- **Date:** 2026-11-22

## Context

The course's central claim is outcome over output: "the job is done when the
job is done" (E1). Kody's pipeline ends at the health SHA and a Discord
report. Flags carry success metrics, but nothing reads the outcome of an
ordinary PR. Lines merged and tickets closed are easy to game.

## Decision

- `docs/intent.md` names the app's outcome metric.
- Every work item names the event it moves, and every ADR names a refutation
  test.
- The ship loop schedules an outcome check at T+7 and T+30 days that reads
  the metric and posts one of keep, iterate, or revert.
- Factory metrics reviewed weekly:
  - PR lead time (p50 and p90)
  - change failure rate
  - escaped defects per 100 PRs (production errors traced to a PR within
    14 days)
  - decision-queue latency
  - friction recurrence (same area reopened within 30 days)
  - net lines of code per primitive
  - cost per merged PR

## Consequences

- Instrumentation is part of every feature.
- Features that do not move their metric get removed, not polished.

## Revisit if

A metric cannot be read for a class of work, for example internal tooling.
Then name a proxy in the work item.
