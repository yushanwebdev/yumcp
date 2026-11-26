# NNNN: Short decision title (usually the "no")

- **Status:** accepted <!-- accepted | superseded by [NNNN](./NNNN-slug.md) -->
- **Date:** YYYY-MM-DD

## Context

The proposal or situation that forced the decision. A few sentences on the
system state and constraints, with links to code or docs. Write this after
the decision is made, not as a design brief.

## Decision

What was decided, in one or two sentences. Usually a product-shaped "no"
that names the existing primitive covering the need. Add this record to that
primitive's `adrs` in `docs/architecture/primitives.yaml`.

## Refutation test

The observation within N days that would show this decision was wrong.

## Reversibility plan

One-way doors only: the flag, dual-write window, or codemod that lets us
back out.

## Consequences

What stays simple, what gets harder.

## Revisit if

The concrete condition that would reopen this decision.

<!-- Do not use this template for a layout tweak, for a library pick the code
already encodes, or because a PR shipped. -->
