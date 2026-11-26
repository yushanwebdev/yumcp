# 0017: Add scheduled signal loops only when friction outpaces manual handling

- **Status:** accepted
- **Date:** 2026-11-22

## Context

Kody runs:

- a daily friction sweep (Clef groups issues, then one agent per group)
- a weekly Legacy Reaper (analysis only; instrument before deleting)
- daily docs-freshness and keep-tests-tight jobs that look only at files
  changed since the last cleanup merge, and start no agent when nothing
  changed
- a red-`main` fixer that dedupes per red stretch
- Sentry → agent with root-cause analysis attached

Sweep PRs skip friction filing, so they do not feed the next sweep. The test
deleter has no coverage check.

## Decision

Add each loop when its input exists and outpaces manual handling, usually
around feature 3–5, and always after containment
([0023](./0023-containment-scaled-to-traffic.md)) and the trust boundaries
in [0022](./0022-break-the-lethal-trifecta.md):

| Loop               | Trigger                  | Rules                                                                                     |
| ------------------ | ------------------------ | ----------------------------------------------------------------------------------------- |
| Friction sweep     | Daily cron               | `friction` + `agent-ready` only; group by area; one agent per group; one PR each           |
| Errors → fix       | Sentry alert             | Issue with root-cause analysis; tainted; agent starts after triage                        |
| Red `main`         | CI failure               | Wait 60 s, re-check, at most one agent per red stretch                                    |
| Legacy reaper      | Weekly                   | Analysis-only issue; checks the "Ready when" of `Cleanup:` issues; instrument when unsure |
| Docs gardener      | Nightly                  | Only files changed since the last cleanup merge; present tense; no agent if nothing changed |
| Test gardener      | Nightly                  | Same scoping; deletions must not lower mutation score or behaviour coverage               |
| Ship report        | Every merge              | One chat message                                                                          |

- Friction is filed after shipping, not mid-task.
- Loop PRs never file friction.
- Each loop starts in shadow mode and logs what it would do for a week
  before acting.

## Consequences

- No token spend on loops with nothing to do.
- Each loop needs a kill switch and a daily cap.

## Revisit if

A loop's PRs are reverted more than 10% of the time. Return that loop to
shadow mode.
