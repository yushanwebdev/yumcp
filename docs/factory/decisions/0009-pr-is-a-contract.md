# 0009: The PR description is a contract

- **Status:** accepted
- **Date:** 2026-11-22

## Context

Kody's PR template starts with Intent and declares the Door (two-way or
one-way) and Cleanup (none, needed, or done). It justifies each new
long-lived test and ends with a system recap: primitives touched, impact,
risk, and a mermaid sequence diagram. Reviewers (human and AI) review the
system change, not the diff. PR bodies double as release notes.

## Decision

- Use `.github/pull_request_template.md` as written.
- `Cleanup: needed` must link a `Cleanup:` issue with a falsifiable "Ready
  when" (`.github/ISSUE_TEMPLATE/cleanup.yml`).
- Lint the PR body in CI. Required sections must be present, the Door must be
  consistent with computed risk
  ([0020](./0020-risk-and-merge-enforced-by-github.md)), and
  `Cleanup: needed` must have a linked issue.
- Use no CHANGELOG. The PR body and ship report are the release notes.

## Consequences

- Reviewers can judge intent and blast radius without reading code.
- Agents spend a few minutes per PR on the description.

## Revisit if

Reviewers report that recaps do not match diffs. Then generate the recap
from the diff with a script.
