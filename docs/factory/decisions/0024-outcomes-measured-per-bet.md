# 0024: Outcomes are measured per bet, not per PR

- **Status:** accepted
- **Date:** 2026-11-22
- **Supersedes:** [0016](./0016-outcome-metrics.md)

## Context

0016 scheduled outcome checks at T+7 and T+30 for every work item. Most
agent PRs are fixes, refactors, and bumps that cannot move a product metric
detectably. Per-PR checks would mostly produce noise, and agents would fill
them with plausible "keep" verdicts, which is Goodhart's law applied to the
check itself. Kody already measures outcomes for flagged features
(`successMetric`, exposure logging, an on/off readout). What it lacks is a
default for unflagged work.

Research for this decision came from a subagent using Firecrawl,
2026-11-22. Some figures are from secondary summaries.

- Only about 10–33% of ideas improve their target metric at Bing, Google,
  Microsoft, and Booking
  (https://exp-platform.com/Documents/2014%20experimentersRulesOfThumb.pdf).
- Spotify reports a win rate of about 12%
  (https://engineering.atspotify.com/2025/9/spotifys-experiments-with-learning-framework).
- Developers using AI believed they were faster while measuring 19% slower,
  so perception is not evidence (https://arxiv.org/abs/2507.09089).
- Shape Up sets appetite and makes bets per project
  (https://basecamp.com/shapeup/2.2-chapter-08).

## Decision

**Levels of measurement:**

| Level      | Measured                                                               |
| ---------- | ---------------------------------------------------------------------- |
| PR         | Only 3–5 global guardrails: error rate, p95 latency, core-flow completion |
| Work item  | Names the bet it belongs to                                            |
| Bet        | Hypothesis, one event, a threshold written before building, a decision date |
| Quarter    | North Star metric and the hit rate of past bets                        |

**How bets work:**

- Bets live in `docs/bets/`, from `docs/bets/0000-template.md`.
- A typed event registry in the app. A bet cannot name an event that is not
  in the registry.
- At each decision date, a scheduled job posts a readout: kept, iterate,
  revert, or inconclusive (underpowered). The tech lead decides.
- Agents may write the queries and draft the readout. They never set
  thresholds or verdicts.
- Under 1,000 weekly users, use a before/after comparison plus 5 user
  conversations instead of an A/B test.

**Weekly factory metrics:**

- lead time
- change failure rate
- escaped defects per 100 PRs
- decision-queue latency
- bets decided on time
- bet refutation rate
- friction recurrence

Net lines of code and cost per PR are not headline metrics. Cost stays as a
budget alarm only.

## Consequences

- Outcome work concentrates where effects are large enough to see.
- A refutation rate near 0% is treated as a sign of gaming, not of success.
- Week 1: `docs/intent.md` names one outcome event, use the bet template,
  instrument three guardrail events, and hold a 15-minute weekly review of
  bets past their date.

## Revisit if

- The first 10 bets are all inconclusive for lack of statistical power.
  Then move to quarterly review only.
- Before/after comparisons prove too confounded to act on.
