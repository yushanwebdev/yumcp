# 0020: Risk and merge authority are enforced by GitHub, not by agent code

- **Status:** accepted
- **Date:** 2026-11-22
- **Supersedes:** [0011](./0011-risk-is-computed.md),
  [0012](./0012-merge-policy-and-trust-ladder.md)

## Context

Kody's ship-pr takes `risk` and `mergeAuthority` as parameters from the
implementing agent, and only `high` parks a PR. The friction-log sweep fixes
extra-care areas (auth, billing, secrets, migrations) at `high` and then
merges them with `mergeAuthority: true`. Its sessions push through the
owner's GitHub account, so the author and the approver are the same
identity.

Bugbot runs only on PRs the owner authored, so on agent PRs "medium"
behaves like "low". Kody's stricter paths, `agentMayMerge: 'none'` and the
log-only `webhook-merge`, show the safe default exists but is not applied
everywhere.

- OpenAI notes that a task-optimised agent "creates pressure to treat an
  approval boundary as just another obstacle"
  (https://openai.com/safety-alignment/auto-review/).
- Anthropic reports users approve 93% of permission prompts, so approvals
  drift toward rubber-stamping
  (https://www.anthropic.com/research/claude-code-auto-mode).
- Meta gates on model-predicted diff risk plus a landing delay in which a
  human can still reject, not on author declarations
  (https://engineering.fb.com/2025/08/06/developer-tools/diff-risk-score-drs-ai-risk-aware-software-development-meta/).

Research for this decision came from a subagent using Firecrawl, 2026-11-22.

## Decision

- `risk = max(declared, path floor)`. A declaration can only raise the tier.
  Add primitive-rollup, taint ([0022](./0022-break-the-lethal-trifecta.md)),
  and error-budget ([0023](./0023-containment-scaled-to-traffic.md)) floors
  once there is data to tune them.
- Enforce the path floors in GitHub:
  - `.github/CODEOWNERS` covers one-way-door paths and the gate
    files themselves (`riskmap.yaml`, `CODEOWNERS`, `.github/**`).
  - A ruleset on the default branch requires a PR, code-owner review,
    approval of the most recent push by someone other than the pusher, and
    dismissal of stale approvals, with no bypass actors.
- A `risk-gate` required check computes risk from `riskmap.yaml`, fails
  closed, and logs declared versus computed risk on every PR.
- Agents commit and open PRs as a GitHub App identity, never as a person's
  account.
- No agent or scheduled job holds merge authority for high risk, and none
  can edit the gate.
- The trust ladder (carried over from 0012):

  | PRs merged                    | Human reads                                    | Merge                     |
  | ----------------------------- | ---------------------------------------------- | ------------------------- |
  | 1–10                          | Every diff                                     | Human                     |
  | 10–30                         | Description + recap; diff for one-way doors    | Human for medium and high |
  | 30+ with zero escaped defects | High risk only                                 | Low and medium auto       |

- Merges are squash merges pinned to the head SHA. If an escaped defect
  traces to an auto-merged PR, drop back one rung for that risk class.

## Consequences

- Enforcement survives a buggy or manipulated agent, because GitHub refuses
  the merge.
- One-way doors queue for a human. Keep path floors narrow so the high tier
  stays rare and approvals stay meaningful.
- Week 1 cost is about 1–2 days (CODEOWNERS, ruleset, App identity). The
  full `risk-gate` check takes about a week.

## Revisit if

- High-tier approvals have a median under one minute with over 95% given
  without comment. Then switch to a delayed-merge veto window.
- Path floors push more than 20% of PRs into high. Then narrow the floors.
- Declared and computed risk disagree on under 1% of 100+ PRs. Then the
  computation may be redundant.
