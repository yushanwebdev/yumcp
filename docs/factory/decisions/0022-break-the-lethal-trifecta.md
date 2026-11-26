# 0022: No agent holds untrusted input, secrets, and write access together

- **Status:** accepted
- **Date:** 2026-11-22
- **Supersedes:** [0014](./0014-only-triaged-work-reaches-agents.md)

## Context

Kody's documented path: the friction-log all-issues sweep puts every open
issue on a public repo, body and comments included, into Devin prompts.
Those sessions push through the owner's GitHub account, may edit workflows,
and merge extra-care fixes with `mergeAuthority`. Kody marks PR comments as
untrusted, but the docs describe no provenance filter for issue text.
Research for this decision came from a subagent using Firecrawl,
2026-11-22.

Incidents and guidance:

- "Comment and Control" (ACM CCS 2026): outside contributors stole repo
  secrets via Claude Code Security Review (PR title), the Gemini CLI Action
  (issue comment), and Copilot Agent (hidden HTML comment)
  (https://oddguan.com/blog/comment-and-control-prompt-injection-credential-theft-claude-code-gemini-cli-github-copilot/).
- The GitHub MCP exploit leaked private data via a malicious public issue
  (https://simonwillison.net/2025/May/26/github-mcp-exploited/).
- Anthropic's own GitHub Action docs say new sanitisation bypasses may
  emerge (https://github.com/anthropics/claude-code-action/blob/main/docs/security.md).
- OWASP LLM01:2025 says no fool-proof prevention is known
  (https://genai.owasp.org/llmrisk/llm01-prompt-injection/).
- The lethal trifecta: untrusted input, private data, and an exfiltration
  path in one agent
  (https://simonwillison.net/2025/Jun/16/the-lethal-trifecta/).
- Meta's Rule of Two (https://ai.meta.com/blog/practical-ai-agent-security/).
- Adaptive attacks beat published defences
  (https://simonwillison.net/2025/Nov/2/new-prompt-injection-papers/).

## Decision

- **Three agent roles.**
  - **Triage agent:** reads raw external text (issues, Sentry, feedback).
    It has no tools, no secrets, and no write access. Its only output is
    schema-validated enums plus a duplicate id. Free text never passes
    through.
  - **Implementer agent:** works from a spec a team member wrote, never the
    raw external text. It has no production secrets and an egress
    allowlist. It uses a GitHub App token with `contents` and
    `pull_requests` write and no `workflows` permission. It never uses a
    person's OAuth.
  - **Gate:** the [0020](./0020-risk-and-merge-enforced-by-github.md)
    ruleset and CODEOWNERS. Diffs touching auth, secrets, or
    `.github/workflows/` need a human merge, enforced by GitHub.
- **`agent-ready`:** only a team member applies it. It is bound to a hash of
  the sanitised body, with HTML comments and invisible characters stripped,
  because the person labelling sees rendered Markdown. Editing the body
  voids the label, and comments added after labelling are ignored.
- **Tainted work:** each item records `source` and `taint`. Tainted work is
  at least medium risk and never gets merge authority.
- **GitHub Actions hygiene:**
  - `permissions: {}` by default, granted per job
  - no `pull_request_target` or `workflow_run` that checks out PR code
  - actions pinned by SHA
  - event text passed only through environment variables
  - no `allowed_non_write_users`
  - "Allow GitHub Actions to create and approve pull requests" turned off
  - deploy secrets kept in environments with required reviewers, and OIDC
    for cloud access
  - zizmor run in `validate`

## Consequences

- Safety comes from privileges, not model restraint. A successful injection
  can at worst produce a PR that a human must merge.
- Work the team files, internal CI failures, and dependency bumps stay fully
  autonomous. Only external text pays the triage cost.
- Full build is about 2–3 weeks. Week 1 (about 2 days):
  - require the label plus a recorded author association
  - strip HTML comments
  - switch to a GitHub App without the `workflows` permission
  - add CODEOWNERS and a ruleset on sensitive paths
  - make the default token read-only
  - run tainted work without secrets

## Revisit if

Red-team results show the triage-to-implementer boundary leaking
instructions. Then tighten the triage schema, or require a human-written
spec for all external work.
