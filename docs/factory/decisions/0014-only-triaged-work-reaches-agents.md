# 0014: Only work a member labelled `agent-ready` reaches an agent

- **Status:** superseded by [0022](./0022-break-the-lethal-trifecta.md)
- **Date:** 2026-11-22

## Context

Kody's daily all-issues sweep puts every open issue on kody-hq/kody, body and
comments included, into Devin prompts. Outsiders file issues there. The
sessions push through the owner's GitHub account and can merge. Anyone can
create a Sentry event from a browser console. "Treat it as data" is a
property of the model, not a control (E5).

## Decision

- Scheduled loops act only on items a team member has labelled
  `agent-ready`.
- Every work item records `source` and `taint` (`internal` or `external`).
  Sentry message text, user feedback, and issues from non-members are
  external.
- Tainted work:
  - runs with no production secrets
  - has risk of at least medium ([0011](./0011-risk-is-computed.md))
  - needs a human to merge any diff touching auth, secrets, or
    `.github/workflows/`
  - never receives merge authority
- Agent tokens cannot modify `.github/workflows/` unless a human approves.

## Consequences

- Outside reports wait for a human to triage them, which adds latency.
- Prompt-injection safety comes from privileges, not from model behaviour.

## Revisit if

The triage step becomes the bottleneck. Then automate labelling with a
classifier that can only add `needs-triage`, never `agent-ready`.
