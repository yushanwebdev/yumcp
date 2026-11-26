# 0003: Deploy a walking skeleton before the first feature

- **Status:** accepted
- **Date:** 2026-11-22

## Context

Agents verify their work against a real deploy: Kody's ship loop waits until
production `/health` serves the merge SHA. When production only appears after
the first feature, deploy, migration, and environment problems all surface at
once, at the worst time.

## Decision

The first PR in every new app is a "hello world" that runs in production and
includes:

- the single `validate` gate, with CI blocking merges on it
- production deploys from `main`, and `/health` returning the commit SHA
- PR previews, seed data, and one seeded user
- `AGENTS.md`, `CLAUDE.md` (`@AGENTS.md`), the PR template, and issue forms
- empty `docs/principles/`, `docs/decisions/`, and
  `docs/architecture/primitives.yaml`

A human reads every line of this PR.

## Consequences

- The first feature travels the real path, and its PR shows what the gates
  miss.
- Day one produces no visible product.

## Revisit if

The app is a throwaway prototype that will never reach production.
