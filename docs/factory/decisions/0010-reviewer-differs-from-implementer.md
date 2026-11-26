# 0010: The gating reviewer comes from a different vendor than the implementer

- **Status:** superseded by [0021](./0021-reviewer-diversity-is-a-measured-hypothesis.md)
- **Date:** 2026-11-22

## Context

The course says the reviewer must not be the implementer, and ideally should
be a different model (E3). In practice, 95 of Kody's last 100 merged PRs
were written by Devin, and the only GitHub review on them came from Devin.
Cursor Bugbot runs only when the PR author is Kent, so on other PRs Devin
Review stands in for it. CodeRabbit gates only at high risk. Reviewers from
the same family share blind spots.

## Decision

- At medium risk and above, at least one gating AI reviewer comes from a
  different vendor and model family than the implementer. Example: Claude Code
  implements, and Bugbot, CodeRabbit, or Codex reviews.
- Every push re-runs review and CI.
- A deterministic sort classifies bot findings. Invalid findings get a short
  reply. Valid or uncertain findings block until a commit fixes them or a
  dated decision is recorded.
- Feedback from collaborators blocks at every risk level until answered. It
  is treated as untrusted text, and the human gets the last word.
- Measure the reviewers: run seeded-defect canary PRs each month for recall,
  and track the invalid-finding rate for precision.

## Consequences

- Reviewer spend roughly doubles at medium risk.
- Reviewer quality becomes something we measure, not assume.

## Revisit if

Canary results show a single reviewer's recall matches the pair's for three
consecutive months.
