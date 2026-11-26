# Adoption stages

Each stage starts only after the previous stage meets its exit criterion
([0002](../decisions/0002-grow-factory-in-stages.md)).

| Stage              | Build                                                                                                                                                             | Exit criterion                                                                                  |
| ------------------ | ----------------------------------------------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------- |
| **0 · Map**        | `AGENTS.md` capped at 20 lines (cap runs from `main`); 5 principles; PR template; issue forms; one `validate`; `/health` returns SHA; walking skeleton in production. Agents use a GitHub App identity without the `workflows` permission; default token read-only; CODEOWNERS and a ruleset on gate and one-way paths (0020, 0022) | 10 consecutive agent PRs fill the template correctly with no human edits                         |
| **1 · Knowledge**  | ADRs; `primitives.yaml` with owners; a classifier script (diff → primitives → rollup); recap block in PRs; shared remote build cache                                | The classifier maps at least 95% of changed files; a primitives check runs in CI                 |
| **2 · Verify**     | `ctl` CLI; seed users and data; PR previews with mocks and teardown; Feature Map notes                                                                            | An agent proves one UI change and one API change with no human login                             |
| **3 · Review and merge** | Fresh-context review plus a named-model reviewer from another vendor (0021); deterministic finding sort; `scripts/ship.ts` state machine; `risk-gate` required check (0020); PR body lint; `Escapes:` trailer | Low-risk PRs merge untouched for 4 weeks while escaped defects are tracked, then extend to medium |
| **4 · Contain**    | Expiring flags with success metrics; backups in a separate account; weekly restore drill timed against a written RTO; expand/contract migrations; post-deploy synthetic checks with auto-revert; 10-minute bake; error-budget merge freeze; per-version metrics and canaries only as traffic warrants (0023) | Weekly drills pass within the RTO; an injected bad deploy is reverted automatically             |
| **5 · Signal loops** | Quarantined triage agent emits schema-only output; `agent-ready` bound to a hash of the sanitised body (0022); friction sweep; Sentry → agent (tainted); red-`main` fixer; weekly reaper; diff-scoped docs and test gardeners; bet readouts (0024) | At least half of non-decision items close with no human; decision-queue p50 under 2 days        |

## Weekly rituals

- **Decision review:** clear `needs-decision` in one sitting, and write ADRs
  for the "no"s.
- **Factory metrics** ([0024](../decisions/0024-outcomes-measured-per-bet.md)):
  lead time, change failure rate, escaped defects per 100 PRs, decision
  latency, bets decided on time, bet refutation rate, friction recurrence.
- **Bet review:** 15 minutes on any bet past its decision date.
- **Monthly reviewer canaries**
  ([0021](../decisions/0021-reviewer-diversity-is-a-measured-hypothesis.md)):
  seeded bugs, bugs reconstructed from your hotfixes, and clean controls.
  Record recall, invalid-finding rate, and unique catches per reviewer.
