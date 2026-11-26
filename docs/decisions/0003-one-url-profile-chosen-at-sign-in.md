# 0003: One MCP URL; the profile is chosen at sign-in

- **Status:** accepted
- **Date:** 2026-11-26

## Context

yumcp exposes many upstream tools to many harnesses (D1 in the first stack
review). Listing every tool floods agent context and degrades tool choice
(https://modelcontextprotocol.io/docs/2026-07-28/develop/clients/client-best-practices).
A **profile** is a curated set of tools with a policy. The review
recommended one URL per profile (`/p/{id}/mcp`); the tech lead chose a single
URL, for easier management across harnesses.

## Decision

- **One endpoint:** `https://yumcp.yushanweb.dev/mcp`. Every harness is configured with
  this URL and nothing else.
- **The profile is chosen on the consent screen** and stored in the grant's
  encrypted props. The access token carries it. To switch profile, sign in
  again, or revoke the grant in the portal.
- **Profile isolation is a code check, not token audience.** Every request
  resolves the profile from the validated token in exactly one function, and
  the `ProfileGuard` Durable Object is keyed by that id. A workflow test
  proves a token for profile A cannot list or call profile B's tools.
- **Inside a profile:** up to 15 tools are listed directly. Above that, the
  profile exposes only `search_tools`, `describe_tool` and `call_tool`.
  These three schemas are frozen: changes are additive only.
- **Tool names** are `<integration>_<tool>`, using `[a-z0-9_]`, at most 48
  characters, and unique within a profile. Harnesses add their own prefix
  (Claude Code adds `mcp__<server>__`), and several model APIs cap names at
  64 characters and reject dots.

## Refutation test

Within 30 days, either:

- profiles are switched by signing in again more than 3 times a week; or
- one of the daily harnesses cannot hold two yumcp connections at once,
  because it keys stored OAuth credentials by URL.

## Reversibility plan

Additive. Add `/p/{id}/mcp` URLs alongside `/mcp`, with per-path
protected-resource metadata (RFC 9728) and token audience per URL
(RFC 8707). Existing harness configs keep working.

## Consequences

- One URL to paste into every harness; nothing to rotate when profiles change.
- Isolation between profiles depends on one code path and its test, not on
  the OAuth audience.
- The URL itself becomes the hardest thing to change: see the domain choice
  in [docs/factory/setup.md](../factory/setup.md).

## Revisit if

A harness needs two profiles at once every day.
