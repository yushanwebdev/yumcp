# 0004: Proxy vendors' official MCP servers; pin their tool definitions

- **Status:** accepted
- **Date:** 2026-11-26

## Context

The first integrations (Particle Pro, Firecrawl, Notion) each run an official
remote MCP server; Notion's is `https://mcp.notion.com/mcp` with OAuth
(https://developers.notion.com/guides/mcp/get-started-with-mcp). Proxying
them means the vendor controls part of yumcp's contract. A changed tool
description is also a known attack: tool poisoning, or a "rug pull".

## Decision

- **Proxy a vendor's official remote MCP server where one exists.** Write a
  native wrapper (`apps/gateway/src/upstream/native/`) only where none does.
  No community or third-party MCP servers without an ADR.
- **Pin every upstream tool definition** (name, description, input and
  output schema, annotations) by SHA-256 hash when an admin approves it.
  Catalog sync runs off the tool-call path. When a hash changes, that tool
  is **quarantined**: hidden from every profile until the admin approves the
  diff in the portal.
- **Upstream credentials belong to the signed-in user.** OAuth upstreams
  (Notion, Particle Pro) connect through the portal, or through URL-mode
  elicitation the first time a tool needs them. API-key upstreams
  (Firecrawl) are entered in the portal. Both are stored envelope-encrypted
  in the vault, and harnesses never see them.
- **Tool annotations decide policy defaults.** Tools with `openWorldHint`
  taint a profile; tools without `readOnlyHint` are writes. An admin can
  override an upstream's annotation in the portal, never in a way that
  weakens it.

## Refutation test

Within 60 days, any of:

- an upstream changes a tool definition and yumcp serves it without approval;
- quarantine fires more than once a week for benign wording changes, so
  approvals become rubber stamps.

## Consequences

- Day one is mostly proxying, not writing integrations.
- Each upstream's behaviour and availability become yumcp's. Outages
  surface as tool errors that name the upstream.
- The approval queue in the portal is a core feature, not an extra.

## Revisit if

An official server is too limited or too unreliable; then write a native
wrapper for that vendor, recorded in a new ADR.
