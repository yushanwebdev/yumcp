# 0011: Risk is computed, not declared

- **Status:** superseded by [0020](./0020-risk-and-merge-enforced-by-github.md)
- **Date:** 2026-11-22

## Context

Kody's ship-pr takes `risk` as a parameter from the implementing agent, and
only `high` parks a PR. Every incentive therefore points toward under-declaring.
The friction sweep's `extra-care.ts` already forces `high` for auth, isolation,
billing, migrations, DR, and secrets, but only inside that sweep.

## Decision

```
risk = max(declared, primitive rollup, path floor, taint floor, error-budget floor)
```

- **Primitive rollup:** `composes` = low, `extends` = medium, `adds` = high.
- **Path floor:** `riskmap.yaml`. Migrations, auth, billing,
  secrets, workflows, and public API schemas are high.
- **Taint floor:** work that started from outside text is at least medium
  ([0014](./0014-only-triaged-work-reaches-agents.md)).
- **Error-budget floor:** while a surface's SLO budget is spent, PRs that
  touch its primitives are at least high.
- When the declared risk is lower than the computed risk, that is itself a
  signal. Log it.

## Consequences

- The ship script, not the agent, owns risk.
- `riskmap.yaml` needs upkeep as the app grows.

## Revisit if

Computed risk forces human review on more than 30% of PRs without catching
anything. Then loosen the path floors.
