# 0002: Grow the factory with the app, in stages

- **Status:** accepted
- **Date:** 2026-11-22

## Context

The course orders the work in three pillars (quality in, catch mistakes,
contain mistakes) and then adds signal inputs. Building every loop up front
spends effort on problems the app does not have yet. Wiring signal loops
before containment lets outside text reach production quickly with no
blast-radius control.

## Decision

Add factory capability in the order of
[runbooks/adoption-stages.md](../runbooks/adoption-stages.md): map → verify
→ review and merge → contain → signal loops. A stage starts only after the
previous stage meets its exit criterion. Containment (stage 3) always comes
before scheduled signal loops (stage 4).

## Consequences

- Early features ship through a short, heavily supervised path.
- Autonomy rises only as there is evidence for it: escaped defects, drill
  results.
- Some patterns (flags, canary) arrive after the first features, when there
  are users to protect.

## Revisit if

A project starts with existing users or production data. In that case,
containment moves to day one.
