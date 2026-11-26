# 0019: Each client repo declares an autonomy level

- **Status:** accepted
- **Date:** 2026-11-22

## Context

Kent's product is his own, so he can accept agent merges on his authority.
Agency work spans clients with different contracts, regulatory exposure, and
appetite for risk. Some product decisions belong to the client.

## Decision

- Each repo declares `autonomy: 0–3` in `riskmap.yaml`:
  - **0:** agents open PRs; humans merge everything.
  - **1:** low risk auto-merges.
  - **2:** low and medium auto-merge.
  - **3:** level 2 plus scheduled signal loops act on their own.
- Regulated or contract-heavy clients cap at level 1.
- GitHub approvals on one-way doors are the audit trail.
- A decision-rights table in the repo's ADRs states which decisions the
  client, the product owner, the tech lead, and the agent may make.
- Report cost per merged PR to the client.

## Consequences

- The same factory serves clients with different risk appetites.
- Raising autonomy is an explicit, recorded decision.

## Revisit if

A client asks for autonomy beyond their cap. Record it as a new ADR in that
repo.
