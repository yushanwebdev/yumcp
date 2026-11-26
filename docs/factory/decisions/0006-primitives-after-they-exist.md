# 0006: Record primitives after they exist, each with an owner

- **Status:** accepted
- **Date:** 2026-11-22

## Context

Kody's `primitives.yaml` lists every primitive with its code roots, docs, and
ADRs. A classifier maps each diff to primitives and rolls the result up into
`composes` / `extends` / `adds`, which drives risk. A catalog designed before
the code exists is a guess. Kody's catalog has no owners, which works for a
solo founder and breaks in a team.

## Decision

- Start with an empty `primitives.yaml`. Add an entry the first time the next
  feature would reuse something the previous one created.
- Each entry has `id`, `group`, `summary`, `code` (directory roots), `docs`,
  `adrs`, and `owner`.
- Invariants (for example tenant isolation) go in an `invariants:` section,
  and every PR that touches one must name it.
- Generate CODEOWNERS from `owner`, so one-way doors route to the person who
  owns that primitive.

## Consequences

- The catalog reflects the system as built.
- A classifier script (longest-prefix match on `code`) becomes possible once
  there are a few entries.

## Revisit if

More than 5% of changed files match no primitive. Either the catalog is
stale or the roots are too narrow.
