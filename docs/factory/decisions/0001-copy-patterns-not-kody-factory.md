# 0001: Copy Kody's patterns; do not fork `@kody/kody-factory`

- **Status:** accepted
- **Date:** 2026-11-22

## Context

Kody's orchestration lives in public Kody packages: `@kody/kody-factory`
(published 2026-11-22), plus `@kentcdodds/ship-pr`, `friction-log`, and
`codebase-cleanup`. To run them you need a Devin v3 key, Cloudflare Workers AI,
a GitHub App, Sentry, a Grok Bot wake URL, and a Kody org. The factory still
runs in shadow mode by default. Two copies of each job are live because Kody
has no way to import packages across orgs. The design assumes one owner who
makes every product decision.

## Decision

Rebuild the patterns inside each app's repo with portable tools: Markdown
files, a few TypeScript scripts, `gh`, and GitHub Actions. Do not fork
`@kody/kody-factory` as the backbone of a new app.

## Consequences

- No dependency on one vendor's agent, model, or hosting for the control loop.
- Each pattern (ship loop, friction sweep, reaper) needs a small script written
  for us. About 300 lines covers the ship loop.
- We give up Kody's board UI and scheduler.

## Revisit if

- `@kody/kody-factory` leaves shadow mode, consolidates to one live copy, and
  supports multi-owner decision routing, or
- maintaining our own scripts costs more than one day a month.
