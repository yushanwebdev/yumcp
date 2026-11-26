# 0002: Run entirely within the Cloudflare Workers Free plan

- **Status:** accepted
- **Date:** 2026-11-26

## Context

The tech lead requires yumcp to run on Cloudflare's free tier. Limits as of
2026-11-26 (https://developers.cloudflare.com/workers/platform/limits/,
https://developers.cloudflare.com/workers/platform/pricing/,
https://developers.cloudflare.com/d1/platform/limits/). All daily limits
reset at 00:00 UTC, and requests fail once a limit is exceeded. D1 has
enforced this since 2026-09-01.

| Resource | Free limit | What it caps for yumcp |
| --- | --- | --- |
| Worker requests | 100,000/day (Error 1027 beyond) | Tool calls, plus a second request for each elicitation |
| Worker CPU | **10 ms per invocation**, cron included | JSON parsing, MCP SDK setup, crypto |
| Subrequests | 50 external, 1,000 to Cloudflare services, per invocation | Upstream calls per tool call |
| Durable Objects | 100,000 requests/day; 13,000 GB-s/day; 100,000 rows written/day; 1 GB per object, 5 GB per account | Guard calls; time the guard stays active |
| D1 | 10 databases; 500 MB each; 5M rows read and 100,000 rows written per day; Time Travel 7 days | Admin data; **PR preview databases** |
| KV | 1,000 writes/day, 1,000 deletes/day | **OAuth token issuance and refresh** |
| Queues | 10,000 operations/day; 24 h retention | One message costs about 3 operations |

## Decision

Design every tool call to fit this budget:

| Per tool call | Budget |
| --- | --- |
| Worker requests | 1 (2 with elicitation) |
| Durable Object requests | **1**: a single `ProfileGuard` RPC that checks revocation, policy, rate limit and taint and appends to the audit log |
| D1 reads | Only the vault entry (about 2 rows); policy comes from the guard |
| KV | Reads only (bearer validation); no writes |
| External subrequests | 1–2 (upstream, plus an occasional upstream token refresh) |
| CPU | p99 under 7 ms, measured by `ctl bench` from the walking skeleton onwards |

That gives a ceiling of roughly **30,000–40,000 tool calls a day** for the
whole account. It suits one organisation or a small team, not a public
service. A design with one Durable Object per organisation (see
[0001](./0001-cloudflare-native-storage.md)) needs the Paid plan anyway.

Rules that are good design whatever the plan:

- **The upstream `fetch` runs in the Worker, never inside the guard.** The
  guard is active for milliseconds, which keeps it well under 13,000 GB-s a day.
- **Upstream responses stream through without being parsed.** Large bodies
  go to R2 through a stream, so a 300 KB Firecrawl scrape costs no JSON CPU.
- **No full MCP server is built per request.** Tool schemas are
  precompiled JSON Schema; parse only the JSON-RPC envelope.
- **Catalog sync with upstream `tools/list` runs off the hot path**, one
  upstream per cron or Queue invocation, so no single invocation exceeds 10 ms.

**Free-tier concessions.** Each one gets a `Cleanup:` issue with "Ready
when: account is on Workers Paid":

1. **Access-token lifetime is 1 hour, not 5 minutes.** Each refresh writes
   about 3 KV keys, so 5-minute tokens would fit only one active client in
   1,000 writes a day. With 1-hour tokens the ceiling is about 13 clients
   that are always on. Revocation stays immediate because every use checks
   the guard's marker. The longer lifetime only widens the window if a
   token is stolen.
2. **Synthetic checks every 5 minutes, not every minute (overrides
   F0023).** Three paths at 1 per minute would cost 4,320 Worker requests
   and 4,320 guard requests a day.
3. **No Queues on the tool-call path.** The audit log is buffered in the
   guard and flushed to R2 in batches.
4. **At most 6 concurrent PR preview databases.** That leaves 4 of the 10
   D1 databases for production, staging, the restore drill and one spare.
   Previews are torn down when the PR closes, with no exceptions.

**First thing to check:** whether `workers-oauth-provider` 1.x accepts a
custom storage provider. If it does, keep tokens in the guard and D1. That
removes both the KV write cap and the KV revocation race
(https://github.com/cloudflare/workers-oauth-provider/issues/428), and
concession 1 goes away.

## Refutation test

Within 30 days of the walking skeleton, any of:

- CPU p99 per tool call exceeds 7 ms after the rules above;
- the account hits any daily limit with fewer than 10 active users;
- free-tier workarounds cost more than 2 engineering days in total.

## Reversibility plan

Two-way. Upgrading to Workers Paid ($5/month) is a billing switch. Then
close each concession's `Cleanup:` issue: shorten token lifetime, set
synthetic checks to every minute, and lift the preview cap.

## Consequences

- yumcp costs nothing to run, at the price of a hard daily ceiling: once
  a limit is hit, calls fail until the 00:00 UTC reset.
- CPU becomes a design constraint, so `ctl bench` joins `validate` early.
- The free-tier ceiling answers part of the "who are the users" question:
  one organisation or a small team.

## Revisit if

- Daily usage passes 50% of any limit for 7 consecutive days;
- a feature (Code Mode, semantic search on every call) cannot fit in 10 ms of CPU;
- yumcp gets users outside one organisation.
