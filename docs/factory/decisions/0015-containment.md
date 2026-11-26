# 0015: Containment

- **Status:** superseded by [0023](./0023-containment-scaled-to-traffic.md)
- **Date:** 2026-11-22

## Context

Kody's flag registry lives in code and requires `removeWhen` or `reviewBy`,
with a test that fails after the date passes. Flags carry success metrics.
Kody's weekly restore drills found real defects (2026-11-15: a restore came
up 400 rows short), but its RTO is undefined. Production deploys go to 100%
at once. The course's example: a company that never tested restores lost its
backups along with the production volume.

## Decision

Add containment once the app has real users, and before any scheduled signal
loop ([0002](./0002-grow-factory-in-stages.md)):

- **Flags:** declared in code with an owner, a `successMetric`, and
  `removeWhen` or `reviewBy`. A test fails after `reviewBy`. A hosted flag
  service is fine. The forced expiry is what matters.
- **Backups:** stored in a separate account or provider from production. An
  automated weekly restore drill runs into an isolated environment and is
  timed against a written RTO. A drill log records first-proven dates and
  what each drill found.
- **Migrations:** expand/contract only, so old and new code can run side by
  side.
- **Deploys:** canary (5% → 25% → 100%), with automatic rollback on error
  rate or p95 latency when the platform supports it. Otherwise keep a
  rehearsed rollback runbook.
- **Drill freshness:** if the last successful drill is more than 8 days old,
  any PR touching migrations is high risk.

## Consequences

- One-way doors become two-way doors where possible.
- Drills cost compute each week and need someone to own them.

## Revisit if

The app has no persistent user data. Then skip backups and keep flags and
rollback.
