# Intent

Written by a human. Agents read this before any product-level decision.

Confirmed by the tech lead on 2026-11-27.

## User and job

The tech lead, working daily across many AI harnesses (Claude Code and
Codex, often inside T3 Code; ChatGPT; their own Mastra agent, yarvis), hires
yumcp to **reach every tool they rely on
(Particle Pro, Firecrawl, Notion, and more) from any MCP client through one
URL**, so that adding, changing or revoking a tool happens once, in one
place, under one security policy, instead of in every harness.

## Job done

The outcome event that proves the job got done, with a target:

- `tool_call_succeeded` from at least **4 distinct harnesses**
  (`client_id`: Claude Code, Codex, ChatGPT, yarvis) in one week, with no
  tool or credential configured in any harness other than the yumcp URL (and
  yarvis's API key).
- Guardrail: tool-call success rate of 98% or more, excluding upstream
  outages that yumcp reports by name.

## First feature

The thinnest slice that completes the job end to end:

Add `https://yumcp.yushanweb.dev/mcp` to one harness → sign in through Cloudflare Access →
choose a profile → call **Firecrawl search** through yumcp, using a Firecrawl
key entered once in the portal → see that call in the portal's audit log.

## We will not build

- **Multiple users or organisations.** There is one user. The schema keeps
  an `owner_id` column, but there is no sign-up, teams or billing.
- **A marketplace or community MCP servers.** Official vendor servers only
  ([ADR 0004](./decisions/0004-proxy-official-mcp-servers.md)).
- **An agent or chat UI.** yumcp serves harnesses; it is not one.
- **Code Mode, for now.** Discovery tools first
  ([ADR 0003](./decisions/0003-one-url-profile-chosen-at-sign-in.md)).
- **Anything outside the Cloudflare Workers Free plan**
  ([ADR 0002](./decisions/0002-run-on-workers-free-plan.md)).

## One-way doors to protect

- **Data that cannot be lost:** upstream connections and the vault (losing
  them means reconnecting every tool), profile and policy definitions, and
  tool-definition approvals.
- **Public contracts other systems depend on:**
  - the MCP URL pasted into every harness;
  - the OAuth metadata and token behaviour;
  - tool names and the three discovery-tool schemas
    ([ADR 0003](./decisions/0003-one-url-profile-chosen-at-sign-in.md)).

## Decision rights

| Decision class                          | Who decides |
| --------------------------------------- | ----------- |
| Product scope and user-facing behaviour | Tech lead (sole user) |
| Architecture and one-way doors          | Tech lead |
| Everything else                         | Agent, within the principles |
