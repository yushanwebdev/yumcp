# 0007: One validate gate; a review comment made twice becomes a check

- **Status:** accepted
- **Date:** 2026-11-22

## Context

Kody runs 35 legs behind one `npm run validate`, including a slop ratchet
(file-size budget, no decorative banners), knip for dead exports, and docs
checks for changelog-style wording. Its harness doc prefers enforcement in
this order: lint, types, or validate, then tests, then scripts, then prose.
Agents write to a shared remote build cache so CI mostly hits it.

## Decision

- One command, `npm run validate` (or the stack's equivalent), is the only
  local gate named in `AGENTS.md`. CI runs the same command.
- Day one includes format, lint, typecheck, unit tests, and dead-export
  detection.
- When a reviewer (human or AI) makes the same comment twice, encode it in
  the strongest cheap guardrail: a lint rule or check, then a test, then a
  script. A prose rule comes last.
- Use a shared remote cache (Nx, Turborepo, or similar) that cloud agents can
  write to.

## Consequences

- The gate grows from observed failures rather than guesses.
- CI time grows. Spending human time is worse than spending CI minutes.

## Revisit if

Validate p50 exceeds 15 minutes even with the cache warm.
