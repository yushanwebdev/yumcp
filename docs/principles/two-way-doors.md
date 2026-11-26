# Two-way and one-way doors

A two-way door is cheap to reverse. Ship it, and change it if it is wrong. A
one-way door is expensive to undo:

- stored data shapes
- public URLs and API contracts
- migrations that delete or rewrite data
- auth and permission changes

Spend review on one-way doors. Turn them into two-way doors where you can,
with a flag, a dual-write window, or a codemod.

Every PR declares `Door:`. Computed risk
([F0020](../factory/decisions/0020-risk-and-merge-enforced-by-github.md)) overrides an
under-declared door.

## Example

- Two-way: button copy, layout, an internal helper's signature.
- One-way: renaming a column that other services read; changing a webhook
  payload that clients parse.
