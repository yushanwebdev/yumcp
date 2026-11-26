# 0004: Agent context is a 20-line map with progressive disclosure

- **Status:** accepted
- **Date:** 2026-11-22

## Context

Agents left unchecked turn `AGENTS.md` into an encyclopedia. Kody caps it at
20 lines with a check that runs from `main` via `pull_request_target`, so a
PR cannot raise the cap for itself, and a PR that edits the check fails. Agents
reliably follow links to the one page a task needs (course E2).

## Decision

- Root `AGENTS.md` holds only pointers, 20 lines at most, enforced by
  `.github/workflows/agents-md-cap.yml`.
- `CLAUDE.md` contains only `@AGENTS.md`.
- Detail lives one hop away: `docs/principles/` (one page per principle,
  each with a real code example), `docs/decisions/`, `docs/architecture/`,
  and `.agents/skills/`.
- A rule lives in exactly one place. Other pages link to it.

## Consequences

- Agents load less irrelevant context.
- Guidance that cannot fit as a pointer has to become a doc page or a check.

## Revisit if

Measurements show agents consistently miss a linked page that matters.
Prefer turning that page into a check before growing the map.
