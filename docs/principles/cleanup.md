# Cleanup

When a new way replaces an old one, the migration is not done until the old
way is gone. Do not run two lanes indefinitely. Delete compat shims, aliases,
dual reads, fallbacks, finished flags, and docs that only explain the old way.

If a leftover cannot go in this change (it needs a second deploy, a soak, or
a data backfill), open a `Cleanup:` issue in the same PR with a falsifiable
"Ready when" and a "How to verify". A TODO comment is not a record.

## Example

```ts
// Wrong: keeps the old lane alive "just in case"
const user = (await getUserV2(id)) ?? (await getUserLegacy(id))

// Right: one lane; the legacy reader is deleted in this PR
const user = await getUser(id)
```

One-way doors (dropping data, changing auth) also follow
[Two-way and one-way doors](./two-way-doors.md).
