# 0021: Reviewer diversity is a hypothesis we measure

- **Status:** accepted
- **Date:** 2026-11-22
- **Supersedes:** [0010](./0010-reviewer-differs-from-implementer.md)

## Context

On Kody, 95 of the last 100 merged PRs were written by Devin, and the only
GitHub review on them came from Devin. Kent's claim that the reviewers
"catch the important things" has no denominator. Research for this decision
came from a subagent using Firecrawl, 2026-11-22; some sources are
secondary, as noted in the report.

- LLM evaluators favour their own generations
  (https://neurips.cc/virtual/2024/poster/96672).
- When two models both err, they agree about 60% of the time, including
  across providers, and more so as models get stronger
  (https://arxiv.org/abs/2506.07962). A different vendor therefore buys
  less independence than assumed.
- The best AI review systems find about a third of known defects
  (CR-Bench-v2, https://arxiv.org/html/2611.02566v1).
- No study found shows that adding a cross-vendor reviewer produces
  measurable unique catches on real bugs.
- Vendor benchmarks are often tuned on their own test sets.

## Decision

- **Fresh context:** every PR is reviewed in a fresh context, never by the
  implementing session.
- **Named second reviewer:** at medium risk and above, add one gating
  reviewer whose model is named and comes from a different vendor than the
  implementer. CodeRabbit-style reviewers with undisclosed models do not
  count. Keep this reviewer only if the ledger shows it makes unique
  catches.
- **Canaries:** each month, open draft canary PRs in
  `quality/review-canaries/`:
  - seeded bugs, each one plausible logic, auth, or boundary bug
  - bugs reconstructed from your own hotfixes
  - clean PRs as controls

  Record per reviewer, in a JSONL ledger: recall, invalid-finding rate, and
  unique catches. Canary recall is a relative comparison between reviewers,
  not an absolute guarantee.
- **Escaped defects:** every production fix carries an `Escapes: #PR`
  trailer. Escaped defects per 100 PRs is the denominator.
- **Containment:** with AI review catching roughly a third of known
  defects, containment ([0023](./0023-containment-scaled-to-traffic.md))
  does more of the work than the course implies. Never trade it for more
  reviewers.

## Consequences

- Reviewer spend is justified by data, and a reviewer that adds nothing
  gets dropped.
- Setup is about 1–2 days, then about an hour a month to read results.
- Week 1: add a named-model second reviewer at medium risk, hand-write 10
  canaries and 5 clean PRs, and adopt the `Escapes:` trailer.

## Revisit if

- After 3 months, the cross-vendor reviewer shows near-zero unique catches.
  Then drop it and rely on fresh context.
- Canary recall does not predict real escapes. Then rely on escaped defects
  alone.
