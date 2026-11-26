# 0013: Shipping runs through a deterministic state machine

- **Status:** accepted
- **Date:** 2026-11-22

## Context

Kody's ship-pr `tick` replaced a long skill with code. Each call returns a
checklist and details for the current step only. Later calls with
`brief: true` stay small. While waiting, the agent arms a timer instead of
polling. Decisions on a CI result or a merge apply to one head SHA only. The
loop ends only after deploy, health check, friction, and report.

## Decision

Each app has `scripts/ship.ts` with these states:

`draft → simplify pass → ready → ci → review → risk-gate (0020) → merge or
park → deploy → health SHA + synthetic checks (0023) → friction → report`

The ship loop does not check outcomes per PR. Bets get scheduled readouts
([0024](./0024-outcomes-measured-per-bet.md)).

- Each run prints the current state, what blocks it, and the exact next
  command.
- The ship skill is five lines: run the script and follow its output.
- Done means production `/health` serves the merge SHA or a later commit.
- After the merge, the script files leftover friction and posts one report:
  intent, door, cleanup, cost, PR link.
- Red `main` after a merge: message the implementer, and open a P0 item if it
  is not fixed within 60 minutes.

## Consequences

- Agents spend fewer tokens and behave the same way whichever agent runs.
- Changes to the process are code changes, so they get reviewed.

## Revisit if

The script grows past roughly 800 lines. Split it by state.
