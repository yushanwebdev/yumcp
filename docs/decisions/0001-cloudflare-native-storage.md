# 0001: Cloudflare-native storage only; no external database

- **Status:** accepted
- **Date:** 2026-11-26

## Context

yumcp runs on Cloudflare Workers. The first stack review recommended
external Postgres (Neon or PlanetScale through Hyperdrive) for portability
and queries across organisations. The tech lead chose to keep every store
inside Cloudflare: one vendor, one account, one deploy, and local development
entirely in Miniflare. Research: D1 limits
(https://developers.cloudflare.com/d1/platform/limits/), Durable Object
limits (https://developers.cloudflare.com/durable-objects/platform/limits/),
and the KV revocation race in `workers-oauth-provider`
(https://github.com/cloudflare/workers-oauth-provider/issues/428).

## Decision

No external database. Each kind of data goes to the Cloudflare primitive
whose consistency matches it:

| Data | Store | Why |
| --- | --- | --- |
| System of record: orgs, users, profiles, policies, upstream connections, encrypted vault entries, tool-definition hashes and approvals | **D1** (one database) | SQL; migrations run centrally from CI; Time Travel (7 days on Free, 30 on Paid); `wrangler d1 export` for off-account backups. Admin-portal writes only, never read on the tool-call path |
| Tool search index for `search_tools` | **D1 FTS5**; add **Vectorize** only if recall is poor | Keyword (BM25) search first |
| Hot-path authority: compiled policy snapshot, revocation markers, rate limits, taint windows, OAuth and URL-elicitation state, audit buffer | **One `ProfileGuard` Durable Object per profile** (SQLite) | Single-threaded, strongly consistent; **one RPC per tool call** ([0002](./0002-run-on-workers-free-plan.md)); admin writes go to D1 and then to the guard, and an alarm reconciles them |
| Inbound OAuth grants and tokens (`workers-oauth-provider`) | **KV** (the library requires it), unless the library accepts a custom store; then the guard and D1 | Every bearer use also checks the guard's revocation marker. Token lifetime: [0002](./0002-run-on-workers-free-plan.md) |
| Public catalog metadata, cached CIMD documents | **KV** | Read-heavy; staleness is harmless |
| Audit log of record | **Guard buffer → R2** in batches, flushed by an alarm, with a bucket lock | Append-only, outside D1's limits; no Queue operation per call |
| Logs and metrics | **Workers Logs** | Analytics Engine added only if a dashboard needs it |
| Large upstream outputs (offloaded), exports, backups | **R2** | Lifecycle rules for expiry |
| Master encryption key | **Workers Secrets** | Never stored in any data store |

Rules:

- **No authorization decision rests on KV alone.** KV is eventually
  consistent for 60 seconds or more, so a revocation that only KV knows
  about is not final.
- **D1 has no row-level security.** Organisation scoping lives in one
  repository layer that takes `orgId` as a required, typed argument. No
  raw D1 access outside it, enforced by a lint rule.
- **Backups (F0023):** nightly `wrangler d1 export` from CI to R2 in a
  **separate** Cloudflare account; weekly restore drill into a scratch D1
  database, timed against the RTO.

**One database, one user** ([intent](../intent.md)). If yumcp ever serves
other users or organisations, the system of record moves to one Durable
Object per organisation, with a small D1 global index (users, API-key
hashes, profile-to-organisation map). That needs the Paid plan (0002).
Record it as an ADR superseding the D1 row.

## Refutation test

Within 90 days of the first real users, any of:

- the D1 database passes 5 GB, half its limit;
- D1 latency at p95 adds more than 50 ms to a tool call;
- one revocation stays effective after its marker is written.

## Reversibility plan

D1 is SQLite, so `wrangler d1 export` produces portable SQL, and the
repository layer is the only code that touches it. Moving to Postgres means
replacing one module and replaying the export. Durable Object, KV and Queue
state is short-lived and can be rebuilt; it is not migrated.

## Consequences

- One vendor, one bill, one local runtime; no connection pooling or
  Hyperdrive caching traps.
- Queries across organisations are fine on D1 and become an export
  pipeline if the per-organisation Durable Object design is adopted.
- Lock-in to Cloudflare is accepted for state that can be rebuilt and kept
  low for the system of record.

## Revisit if

- D1's 10 GB limit or throughput per database becomes a constraint;
- `workers-oauth-provider` ships consistent (non-KV) storage, which would
  remove the revocation-marker workaround;
- the project needs to run outside Cloudflare.
