# 0005: Sign-in: workers-oauth-provider with CIMD, behind Cloudflare Access

- **Status:** accepted
- **Date:** 2026-11-26

## Context

Remote MCP servers authorize with OAuth 2.1 under the 2026-07-28 spec:
protected-resource metadata (RFC 9728), with Client ID Metadata Documents
(CIMD) as the recommended client registration. Dynamic Client Registration
(DCR) is deprecated
(https://modelcontextprotocol.io/specification/2026-07-28/basic/authorization).
yumcp must stay inside Cloudflare's free tier (0002), and has one user: the
tech lead.

## Decision

- **No hand-written authorization server.** Use
  `@cloudflare/workers-oauth-provider` 1.x with
  `clientIdMetadataDocumentEnabled: true` and the
  `global_fetch_strictly_public` compatibility flag. The flag stops CIMD
  fetches from reaching internal addresses (SSRF).
- **DCR stays on only until every daily harness is confirmed to use CIMD.**
  Then turn it off. Track this in a `Cleanup:` issue.
- **Human sign-in is Cloudflare Access** (Zero Trust free plan), in front of
  `/authorize` and `/portal`. Policy: the tech lead's identity only, with
  MFA. No password or passkey code in yumcp.
- **API keys for harnesses without OAuth.** Each key is scoped to one
  profile, prefixed `ymk_`, shown once, and stored only as a hash in D1.
  They are validated through the provider's `resolveExternalToken`.
- **Revocation is final immediately.** Every bearer use checks a marker in
  `ProfileGuard`, because grants in KV can outlive their deletion
  (https://github.com/cloudflare/workers-oauth-provider/issues/428).
- **Token lifetimes:** access tokens 1 hour (a free-tier concession, see
  0002), refresh tokens 30 days with rotation.
- **One hostname: `yumcp.yushanweb.dev`**, as a Workers Custom Domain. The
  `yushanweb.dev` zone must be on Cloudflare (full setup; partial CNAME
  setup needs the Business plan). The production Worker sets
  `workers_dev: false` and `preview_urls: false`: any other hostname that
  serves the same Worker serves `/portal` and `/authorize` outside Access.
  PR preview Workers use seed data only, never real upstream credentials.
- **Issuer identification (RFC 9207).** The authorization response carries
  `iss`, and the metadata advertises
  `authorization_response_iss_parameter_supported`. Without it, ChatGPT
  falls back to a per-connection client ID instead of its stable CIMD
  document (https://developers.openai.com/plugins/build/auth).
- **Headless clients have no human to approve.** A profile used only through
  an API key cannot complete URL-mode approval, so its write tools are
  denied unless the tech lead pre-approves each one in the portal.

### Client matrix

T3 Code is not a separate client: it runs Claude Code and Codex.

| Harness | Registration | Credential |
| --- | --- | --- |
| Claude Code | CIMD, DCR fallback (verify) | OAuth |
| Codex | verify | OAuth |
| ChatGPT | CIMD preferred, DCR fallback; token auth `none` or `private_key_jwt`; cannot send API keys | OAuth |
| yarvis (Mastra agent, server-side) | none: Mastra's OAuth provider has no DCR and its sign-in needs a loopback browser | `ymk_` key in `requestInit.headers` |

## Refutation test

Within 30 days of the first feature:

- any daily harness fails to connect through CIMD or DCR; or
- a revoked grant or API key succeeds on any call.

## Reversibility plan

Clients see only standard OAuth metadata, so the authorization server can be
replaced behind the same URL. Every client signs in once more.

## Consequences

- Almost no auth code of our own; Access adds MFA and an audit trail.
- The hostname is a custom domain the tech lead already owns, so it costs
  nothing extra; it ties yumcp to the `yushanweb.dev` zone being on
  Cloudflare.
- The walking skeleton proves that `/portal` and `/authorize` are
  unreachable without Access on every hostname that serves the Worker.

## Revisit if

`workers-oauth-provider` gains a consistent storage backend. Then remove the
revocation marker and shorten access-token lifetime.
