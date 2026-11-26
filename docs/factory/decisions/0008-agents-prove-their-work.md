# 0008: Agents prove their own work

- **Status:** accepted
- **Date:** 2026-11-22

## Context

Kody's `control-kody` CLI (`doctor`, `dev`, `request`, `browse`, `map`,
`verify`, `health`, `preview`) was built almost entirely from friction
issues that agents filed. A Feature Map records how to seed, reach, and assert
each surface. Previews deploy the whole stack, with mocked third parties and
seed users. CI green alone does not count as proof for a user-visible change.

## Decision

- Every app has a `ctl` CLI. It starts with `doctor`, `dev`, `seed`,
  `request`, `browse --screenshot`, and `health --sha --wait`, and grows only
  from friction.
- Verification is local first. Use a PR preview only when local cannot
  exercise the change (platform bindings, real OAuth or webhooks, deploy
  config).
- Medium and high risk require verification as a seeded user with data
  specific to the change. A login or health smoke does not count.
- UI changes include screenshots, or a video for multi-step flows.
- Mocks sit behind environment variables. No `if (isPreview)` branches in app
  code.
- Tests: "no new tests needed" is the default. Add a workflow test for
  behaviour that could break silently (auth, billing, data integrity,
  permissions) and one regression test per bug fix.

## Consequences

- One-off checks do not become permanent tests.
- The CLI becomes the agent's main verification tool.

## Revisit if

Agents still click through the UI to verify. Find the friction that causes
it.
