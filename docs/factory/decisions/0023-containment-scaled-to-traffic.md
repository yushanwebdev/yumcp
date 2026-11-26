# 0023: Containment scales with traffic

- **Status:** accepted
- **Date:** 2026-11-22
- **Supersedes:** [0015](./0015-containment.md)

## Context

0015 prescribed 5% → 25% → 100% canaries. A canary needs volume to say
anything. Seeing zero errors in n requests only bounds the failure rate
below about 3/n, so ruling out a 1% regression takes about 300 canary
requests. At low traffic, a 5% slice is a handful of sticky users: a
cohort, not a sample.

Kody already runs a post-deploy multi-region health check
(`tools/ci/regional-health-check.ts`, 150 s of Globalping probes). It fails
the job on outages or a mixed commit. Detection exists. Recovery is manual.

Research for this decision came from a subagent using Firecrawl, 2026-11-22.

- Cloudflare Workers gradual deployments are per script. Durable Objects are
  pinned per version, and there is no native automatic rollback
  (https://developers.cloudflare.com/workers/versions-and-deployments/gradual-deployments/).
- Cloudflare's own 2025 outages came from global, ungated config pushes,
  and its fix was health-mediated, gated rollout
  (https://blog.cloudflare.com/code-orange-fail-small-complete/).
- Google SRE: most incidents come from binary or config pushes, and impact
  is proportional to the traffic exposed
  (https://sre.google/workbook/canarying-releases/).
- DORA 2025: AI adoption correlates with higher instability unless feedback
  loops are strong
  (https://cloud.google.com/blog/products/ai-machine-learning/announcing-the-2025-dora-report).

## Decision

**Always, from the first real user:**

- **Flags:** expiring flags with a success metric for risky features.
- **Migrations:** expand/contract only.
- **Backups:** stored in a separate account, with a weekly restore drill
  timed against a written RTO and a drill log.
- **Rollback:** rehearsed once per quarter.
- **Post-deploy synthetic checks:** 3–5 critical paths, probed from outside
  on every deploy and every minute. Two consecutive failures trigger an
  automatic revert.
- **Bake time:** 10 minutes before the next merge deploys.
- **Error-budget merge freeze:** while a surface's budget is spent,
  auto-merge above low risk pauses for PRs touching it.
- **Drill freshness:** if the last successful drill is older than 8 days,
  PRs touching migrations are high risk.

**Then scale by traffic:**

| Requests/day | Add                                                                                     |
| ------------ | --------------------------------------------------------------------------------------- |
| < 1k         | Nothing more                                                                            |
| 1k–100k      | Per-version error and latency metrics (version tag on every log line); ramp an early cohort (internal or free users) before everyone; hold merges while red |
| > 100k       | Percentage canaries (5 / 25 / 100) with automated analysis and automatic rollback       |

**Host notes:**

- **Cloudflare:** `versions upload` + `versions deploy`, version affinity,
  `ScriptVersion` in logs. Durable Object class changes deploy alone.
- **Vercel:** Skew Protection, plus your own alarm wiring for Rolling
  Releases.
- **Fly.io:** blue-green with health checks.
- **Kubernetes:** Argo Rollouts AnalysisRuns.

## Consequences

- New apps get fast detection and an automatic revert in about a day of
  work, without canary complexity they cannot use yet.
- Canary machinery arrives only when the statistics support it.

## Revisit if

- The host ships native automatic rollback on error rate. Then adopt it at
  any traffic level.
- The synthetic checks miss an incident that a canary would have caught.
  Then lower the canary threshold.
