# Fail loudly at the right layer

Fail at the layer that can name the mistake and can still refuse the bad
write. A quiet fallback further down hides which fact was missing. Do not
mask slowness with longer timeouts or degraded modes. Fix the root
bottleneck, or file an issue if a workaround has to ship.

## Example

```ts
// Wrong: a partial scan becomes a "valid" empty cache
const subscribers = (await loadAll()).filter(Boolean)
cache.set(key, subscribers)

// Right: refuse to cache a partial result; the next read rescans
const { items, failures } = await loadAll()
if (failures > 0) throw new Error(`refusing to cache: ${failures} loads failed`)
cache.set(key, items)
```
