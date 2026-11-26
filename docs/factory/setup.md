# Factory setup: stage 0

Tracks [runbooks/new-app.md](./runbooks/new-app.md) for this repo. Tick items
as they land; delete this file once stage 0 meets its exit criterion and move
the remaining stages to issues.

## Done (2026-11-26)

- [x] Local git repo on `main`
- [x] Playbook templates copied to their target paths ([mapping](./README.md#where-the-playbook-templates-landed))
- [x] Five starter principles in `docs/principles/`
- [x] Factory decisions, runbooks, and analysis vendored to `docs/factory/`, cited as `FNNNN`
- [x] `AGENTS.md` (13 lines, pointers to files that exist only), `CLAUDE.md` = `@AGENTS.md`
- [x] `riskmap.yaml` at `autonomy: 0`; `high:` aligned with `.github/CODEOWNERS`, including `docs/factory/**` as a gate path
- [x] CODEOWNERS owner set to `@yushanwebdev` (from the local git user name; confirm it is the GitHub login)
- [x] `.gitignore`

## 1. Intent (human only, 1–2 h)

- [x] Draft [docs/intent.md](../intent.md) from the tech lead's answers (2026-11-26)
- [ ] Tech lead rewrites it in their own words and confirms the "job done" target

## 2. Stack decisions (about 1 h)

Run each through [Review and Recommend](./runbooks/review-and-recommend.md).
Write an ADR in `docs/decisions/` only for a "no" the next agent would
otherwise re-propose.

- [x] Language and runtime: TypeScript monorepo (npm workspaces)
- [x] Host: Cloudflare Workers, Free plan only ([ADR 0002](../decisions/0002-run-on-workers-free-plan.md))
- [x] Storage: Cloudflare-native, one D1 database plus a `ProfileGuard` Durable Object per profile ([ADR 0001](../decisions/0001-cloudflare-native-storage.md))
- [x] Users: one (the tech lead), who connects their own upstream accounts ([intent](../intent.md))
- [x] Tool exposure: one URL, profile chosen at sign-in ([ADR 0003](../decisions/0003-one-url-profile-chosen-at-sign-in.md))
- [x] Integrations: proxy official MCP servers, pin tool definitions ([ADR 0004](../decisions/0004-proxy-official-mcp-servers.md))
- [x] Auth: workers-oauth-provider with CIMD, behind Cloudflare Access ([ADR 0005](../decisions/0005-sign-in.md))
- [x] Host name: `yumcp.yushanweb.dev`, a Workers Custom Domain
      ([ADR 0005](../decisions/0005-sign-in.md)). Needs the `yushanweb.dev`
      zone on Cloudflare (full setup).
- [x] Implementing agent: Claude (Claude Code, Opus 5.5)
- [x] Reviewers: a fresh-context Claude reviewer, and Codex as the
      different-vendor reviewer for medium risk and above (F0021). Codex
      comments; it never approves.
- [x] Solo-maintainer review model: the agent runs locally and pushes as
      the App, the tech lead approves (section 3)
- [x] One-way-door paths in `riskmap.yaml` and `.github/CODEOWNERS` set for
      the layout below

Layout (the walking skeleton creates it):

```
apps/gateway/        one Worker: /mcp, OAuth endpoints, portal API, portal static assets
  src/auth/          OAuth provider, Access JWT check, API keys             high
  src/contract/      tool naming, discovery-tool schemas, profile resolution high
  src/vault/         envelope encryption of upstream credentials            high
  src/guard/         ProfileGuard Durable Object: policy, revocation, taint high
  src/upstream/      proxy to official MCP servers; native/ wrappers        medium
apps/portal/         admin UI (built to static assets the gateway serves)   low
packages/db/         D1 schema and the org-scoped repository layer          medium
  migrations/                                                               high
```

## 3. GitHub (human only, in settings; F0020, F0022)

- [x] **Repo visibility: public** (2026-11-27). On GitHub Free, rulesets,
      enforced code owners and environment protection rules work only in
      public repos (https://github.com/pricing). Public-repo duties:
  - [ ] Seed data, tests, fixtures and screenshots never contain real
        profiles, tool lists, tokens or upstream output
  - [ ] Secret scanning and push protection on
  - [ ] Settings → Actions: require approval for workflow runs from
        **all** outside contributors
  - [ ] Issues and PRs from non-members are tainted input (F0022, already
        in `riskmap.yaml`); agents never act on their text directly
  - [x] Dropped `docs/factory/reference/kody-analysis.md` before the first
        push (2026-11-27). The original stays in the playbook.
- [x] Create the public remote repo (`yushanwebdev/yumcp`, empty, HTTPS remote)
- [x] First push of `main`, by the tech lead (2026-11-27). It contains a
      workflow file, which the App cannot push.
- [x] Created the `yumcp-agent` GitHub App on 2026-11-27, through the
      manifest flow. Its live permissions were checked with a JWT and match
      the list below. App ID `5941501`, bot user `361682843`. The private
      key lives only in the tech lead's password manager. Its secret
      reference is `private_key_ref` in `~/.config/yumcp-agent/app.json`,
      outside the repo, next to the non-secret IDs. The App is used from this
      machine, not from Actions or the cloud.
- [x] Installed on `yushanwebdev/yumcp` only (2026-11-27). Installation ID
      `186243190`, recorded in `app.json`. A test token was minted, could
      reach only this repo with the permissions listed below, and was then
      revoked.
- Settings used when creating the App:
  - github.com/settings/apps/new; webhook **off**; installable on this
    account only
  - Repository permissions: Contents, Pull requests, Issues: read and
    write. Actions, Checks, Commit statuses: read. Workflows,
    Administration and everything else: **no access**
  - Install it on the yumcp repo only. Note the App ID and installation ID.
  - Generate a private key and keep it outside the repo
    (`~/.config/yumcp-agent/`, mode 600)
  - Commit identity:
    `<bot-user-id>+yumcp-agent[bot]@users.noreply.github.com`
  - Agents never push with your OAuth or SSH key
- [ ] Default-branch ruleset: require a PR, code-owner review, approval of
      the most recent push by someone other than the pusher, dismiss stale
      approvals, no bypass actors
- [ ] Settings → Actions: default `GITHUB_TOKEN` read-only; turn off "Allow
      GitHub Actions to create and approve pull requests"
- [ ] Deploy secrets in an environment with required reviewers; OIDC for cloud access

Solo-maintainer catch: "approval of the most recent push by someone other
than the pusher" means a commit you push yourself can never be approved
while you are the only human, and a PR's author can never approve it.

Decided (2026-11-27): **the agent runs on this machine and acts on GitHub
only as the `yumcp-agent` App.** It pushes and opens every PR, and the tech
lead approves and merges. The tech lead's own edits go through the agent. No
bypass actors.

Guards, because the agent runs as the same macOS user as the tech lead:

- [ ] The yumcp remote uses HTTPS with the App's one-hour token
      (`npm run ctl gh-token`), never the tech lead's SSH key or `gh`
      login. The App's private key lives outside the repo.
- [ ] Strict sandbox: `sandbox.allowUnsandboxedCommands: false`. Bypass
      permission mode together with unsandboxed retries lets the agent leave
      the sandbox silently
      (https://code.claude.com/docs/en/agent-sdk/typescript).
- [ ] Deny rules for `gh pr review`, `gh pr merge`, `gh auth token`,
      `gh api`, `security find-*-password` and `git credential`
- [ ] `npm run ctl doctor` fails if the agent session can obtain any GitHub
      credential other than the App token: through the `gh` keychain entry,
      `git credential fill` or `~/.config/gh`
- [ ] No production Cloudflare access for the agent (F0022: the implementer
      holds no production secrets). This machine has a `wrangler login`
      (`~/Library/Preferences/.wrangler/`) that could deploy, read D1 or
      replace secrets directly, skipping review. Block reads of that
      directory in the sandbox, deny `wrangler deploy`, `wrangler secret`,
      `wrangler login` and any `--remote` command, and have `doctor` check
      that `wrangler whoami` fails inside an agent session. Local
      development uses Miniflare (`wrangler dev`, no login). Production and
      preview deploys run only in CI with a scoped API token.
- **Accepted risk (2026-11-27):** yumcp agent sessions can reach the tech
  lead's full password manager, because it answers any process running as
  this user. Closing this needs a strict sandbox
  (`allowUnsandboxedCommands: false`, `enableWeakerNetworkIsolation: true`)
  plus a vault-scoped service account. **Revisit** before the vault holds the
  first real upstream OAuth grant, together with moving the agent into a
  container or a separate macOS user.
- [ ] Approvals happen in GitHub Mobile, a device the agent cannot drive. No
      Claude in Chrome or computer use in yumcp sessions.
- [ ] `.claude/**` is a gate path (riskmap, CODEOWNERS), so the agent cannot
      weaken its own settings through a PR

These guards stop mistakes and most prompt injection. They are not a real
boundary: anything running as this macOS user can reach the tech lead's
keychain unless the sandbox blocks it. Proposed trigger: before the vault
stores the first real upstream OAuth grant (Notion, Particle Pro), move the
agent into a container or a separate macOS user that holds only the App key
and Claude credentials.

## 4. Walking skeleton (day 1)

Prompt, once 1–3 are done:

> Build a walking skeleton for docs/intent.md as a TypeScript npm-workspaces
> monorepo on Cloudflare Workers (Free plan, ADR 0002), using the layout in
> docs/factory/setup.md: "hello world" running in production. Include
> `npm run validate` (format, lint, typecheck, unit tests, dead-export check)
> with CI blocking merges on it; production deploy from `main` with
> `wrangler`; `/health` returning the commit SHA; PR preview Workers and D1
> databases torn down on close (at most 6 at once); a seed script with one
> user; and `npm run ctl` with `doctor`, `dev`, `seed`, `health --sha --wait`,
> `gh-token` (mints the `yumcp-agent` App's one-hour installation token) and
> `bench` (CPU per request, failing above a 7 ms p99). `doctor` includes the
> credential-isolation check from setup section 3. Create the D1
> database, an empty `ProfileGuard` Durable Object, and the Cloudflare Access
> application in front of `/portal` and `/authorize` on `yumcp.yushanweb.dev`
> (a Workers Custom Domain, with `workers_dev` and `preview_urls` off in
> production), and add a check that those paths return the Access login on
> every hostname that serves the Worker. Set every workflow to `permissions: {}` by default,
> pin actions by SHA, and add zizmor to validate. Add a required check that
> the PR title follows Conventional Commits, because squash merges use it as
> the commit message. Add the `validate`, `ctl`,
> and ship-pr skill pointers to `AGENTS.md` and keep it under 20 lines. No
> feature code.

Read every line of that PR; every later PR copies its patterns.

## Stage 0 exit criterion

10 consecutive agent PRs fill the PR template correctly with no human edits.
Then start stage 1 in [adoption-stages.md](./runbooks/adoption-stages.md).
