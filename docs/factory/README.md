# Software factory

How this repo turns intent into shipped code: agents write the code, humans
make product calls and one-way-door calls.

Vendored on 2026-11-26 from the software-factory playbook (snapshot dated
2026-11-22, kept in the `kody/software-factory-playbook` folder). The
evidence behind the decisions stays in that playbook; each decision here
carries its own context and sources.

## Conventions

- Factory decisions are cited as **`FNNNN`** (for example F0020) and live in
  [decisions/](./decisions/). This app's own ADRs are plain `NNNN` and live in
  [docs/decisions/](../decisions/). The two numbering schemes never mix.
- Never edit a factory decision in place. To change one, add an app ADR in
  `docs/decisions/` that names the `FNNNN` it overrides and why. Port the
  change upstream to the playbook if it applies to other apps.
- `docs/factory/` is a gate path: CODEOWNERS requires a human review.

## Where to start

- Current stage and next steps: [setup.md](./setup.md)
- Setup through the first feature: [runbooks/new-app.md](./runbooks/new-app.md)
- Raising autonomy: [runbooks/adoption-stages.md](./runbooks/adoption-stages.md)
- Product or architecture calls: [runbooks/review-and-recommend.md](./runbooks/review-and-recommend.md)

## Where the playbook templates landed

| Playbook file                                  | In this repo                              |
| ---------------------------------------------- | ----------------------------------------- |
| `templates/AGENTS.md`, `templates/CLAUDE.md`   | `AGENTS.md`, `CLAUDE.md`                  |
| `templates/docs/intent.md`                     | `docs/intent.md`                          |
| `templates/docs/decisions/0000-template.md`    | `docs/decisions/0000-template.md`         |
| `templates/docs/bets/0000-template.md`         | `docs/bets/0000-template.md`              |
| `templates/docs/architecture/primitives.yaml`  | `docs/architecture/primitives.yaml`       |
| `templates/riskmap.yaml`                       | `riskmap.yaml`                            |
| `templates/.github/*`                          | `.github/*`                               |
| `principles/*`                                 | `docs/principles/*`                       |
| `decisions/`, `runbooks/`                      | `docs/factory/` (this folder)             |
| `reference/`                                   | not vendored (stays in the playbook)      |

## Decisions

| #                                                                    | Decision                                                                                                   |
| -------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------- |
| [F0001](./decisions/0001-copy-patterns-not-kody-factory.md)          | Copy Kody's patterns into the repo; do not fork `@kody/kody-factory`                                       |
| [F0002](./decisions/0002-grow-factory-in-stages.md)                  | Grow the factory with the app, in stages; containment before signal loops                                 |
| [F0003](./decisions/0003-walking-skeleton-first.md)                  | Deploy a walking skeleton to production before the first feature                                          |
| [F0004](./decisions/0004-lean-agent-context.md)                      | Agent context is a 20-line map with progressive disclosure                                                 |
| [F0005](./decisions/0005-review-and-recommend-adrs.md)               | Product calls go through Review and Recommend; ADRs record the "no"s                                       |
| [F0006](./decisions/0006-primitives-after-they-exist.md)             | Record primitives after they exist, each with an owner                                                     |
| [F0007](./decisions/0007-one-validate-gate.md)                       | One validate gate; a review comment made twice becomes a check                                             |
| [F0008](./decisions/0008-agents-prove-their-work.md)                 | Agents prove their own work with a ctl CLI, seed data, and previews                                        |
| [F0009](./decisions/0009-pr-is-a-contract.md)                        | The PR description is a contract                                                                           |
| ~~F0010~~                                                            | Superseded by F0021                                                                                        |
| ~~F0011~~, ~~F0012~~                                                 | Superseded by F0020                                                                                        |
| [F0013](./decisions/0013-deterministic-ship-loop.md)                 | Shipping runs through a deterministic state machine; done means health SHA                                |
| ~~F0014~~                                                            | Superseded by F0022                                                                                        |
| ~~F0015~~                                                            | Superseded by F0023                                                                                        |
| ~~F0016~~                                                            | Superseded by F0024                                                                                        |
| [F0017](./decisions/0017-signal-loops-when-friction-outpaces.md)     | Add scheduled signal loops only when friction outpaces manual handling                                     |
| [F0018](./decisions/0018-what-we-do-not-copy.md)                     | What we do not copy from Kody                                                                              |
| [F0019](./decisions/0019-autonomy-level-per-client-repo.md)          | Each repo declares an autonomy level                                                                       |
| [F0020](./decisions/0020-risk-and-merge-enforced-by-github.md)       | Risk and merge authority are enforced by GitHub, not agent code; trust ladder                              |
| [F0021](./decisions/0021-reviewer-diversity-is-a-measured-hypothesis.md) | Fresh-context review always; a named-model second reviewer kept only if canaries show unique catches   |
| [F0022](./decisions/0022-break-the-lethal-trifecta.md)               | No agent holds untrusted input, secrets, and write access together                                         |
| [F0023](./decisions/0023-containment-scaled-to-traffic.md)           | Containment scales with traffic: synthetic checks with auto-revert first, canaries only above ~100k req/day |
| [F0024](./decisions/0024-outcomes-measured-per-bet.md)               | Outcomes are measured per bet with thresholds set in advance; PRs get guardrails only                      |

## Known inconsistencies in the snapshot

Not fixed here, because decisions are never edited in place. Fix upstream.

- F0002 numbers containment as stage 3 and signal loops as stage 4;
  [adoption-stages.md](./runbooks/adoption-stages.md) numbers them 4 and 5
  (it adds stage 1, Knowledge). The ordering agrees; follow adoption-stages.
- [new-app.md](./runbooks/new-app.md) step 2 cites superseded F0010. Follow
  F0021: fresh-context review always; the named-model cross-vendor reviewer
  gates at medium risk and above.
