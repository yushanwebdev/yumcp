# 0012: Merge policy follows a trust ladder; one-way doors always need a human

- **Status:** superseded by [0020](./0020-risk-and-merge-enforced-by-github.md)
- **Date:** 2026-11-22

## Context

Kent auto-merges low and medium risk once CI is green and reviews are
handled, and parks high risk for himself. Kody's friction sweep merges
"high" extra-care fixes with `mergeAuthority: true`, so for scheduled work
"high" means more AI review, not a human. Trust in the pipeline has to be
earned with evidence, not assumed.

## Decision

| PRs merged                     | Human reads                                     | Merge                    |
| ------------------------------ | ----------------------------------------------- | ------------------------ |
| 1–10                           | Every diff                                      | Human                    |
| 10–30                          | PR description + recap; diff for one-way doors  | Human for medium and high |
| 30+ with zero escaped defects  | High risk only                                  | Low and medium auto      |

- High risk and every one-way door need a GitHub approval from a human, always.
- No scheduled job or agent can grant itself merge authority.
- Merges are squash merges pinned to the head SHA. Do not use GitHub
  auto-merge.
- A weekly decision review clears the `needs-decision` queue in one sitting,
  with a 2-day p50 target.

## Consequences

- Early throughput is limited by human review. That is intended.
- The escaped-defect metric ([0016](./0016-outcome-metrics.md)) has to exist
  before reaching the third rung.

## Revisit if

An escaped defect traces to an auto-merged PR. Drop back one rung for that
risk class.
