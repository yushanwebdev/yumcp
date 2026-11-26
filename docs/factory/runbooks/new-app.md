# New app: setup through the first feature

Build the factory with the first feature, not before it. Get the delivery
path to production working on day one, push the first feature through every
stage of it, and write principles, primitives, and checks after you have
seen what the agent gets wrong.

## 1. Decide what you are building (1–2 h, human only)

Write `docs/intent.md` from `docs/intent.md` yourself:

- The user, and the job they hire the app for, in one sentence.
- "Job done" as a measurable outcome event.
- What you will not build.

Pick the first feature: the thinnest slice that completes the job end to end.
It should be a feature, not a layer like "auth".

## 2. Make the stack decisions (about 1 h)

Run each big choice (framework, host, database, auth, implementing agent,
reviewing agent) through
[review-and-recommend.md](./review-and-recommend.md). You choose. Write ADRs
only for the "no"s the next agent would otherwise propose again. Record the
implementer and reviewer vendors to satisfy
[0010](../decisions/0010-reviewer-differs-from-implementer.md).

## 3. Walking skeleton, deployed (day 1)

Copy `templates/` into the repo, then prompt the agent:

> Build a walking skeleton for docs/intent.md on <stack>: "hello world" running
> in production. Include `npm run validate` (format, lint, typecheck, unit
> tests, dead-export check) with CI blocking merges on it; production deploy
> from `main`; `/health` returning the commit SHA; PR previews torn down on
> close; a seed script with one user; and `npm run ctl` with `doctor`, `dev`,
> `seed`, and `health --sha --wait`. Set every workflow to `permissions: {}`
> by default, pin actions by SHA, and add zizmor to validate. Keep
> `AGENTS.md` under 20 lines. No feature code.

Then do these yourself in GitHub settings, not through the agent
([0020](../decisions/0020-risk-and-merge-enforced-by-github.md),
[0022](../decisions/0022-break-the-lethal-trifecta.md)):

- Create a GitHub App for agents with no `workflows` permission.
- Add `CODEOWNERS` and a default-branch ruleset requiring code-owner review
  and approval of the most recent push by someone other than the pusher.
- Turn off "Allow GitHub Actions to create and approve pull requests".

Read every line of this PR. Every later PR copies its patterns.

## 4. First feature, through the full loop

**4a. Write the bet, then the work item.** The first feature is the app's
first bet. Fill in `docs/bets/B001-*.md` from the template: hypothesis, one
event, a threshold fixed before building, a decision date
([0024](../decisions/0024-outcomes-measured-per-bet.md)). Then open the
`work-item` issue: intent, acceptance criteria, door, bet, out of scope. Do
not write a spec.

**4b. Review the plan in plan mode.** Ask the agent for a plan naming the
data model, external contracts (URLs, API and webhook shapes), and the
reusable pieces it will create. Spend your attention on the one-way doors.

**4c. Keep the process proportionate.** With no users, almost everything is
a two-way door, so skip flags. Canaries wait until traffic can support them
([0023](../decisions/0023-containment-scaled-to-traffic.md)). Keep validate,
a fresh-context reviewer plus a named-model reviewer from another vendor,
health SHA verification, a post-deploy synthetic check of the critical path,
and backups of the first real table.

**4d. Implement and verify.** The agent adds whatever `ctl` commands it needs
to prove the feature: `request`, `browse --screenshot`. Before marking the PR
ready, it makes one simplify pass:

> Before marking the PR ready, do one simplify pass over your complete body
> of work. Look for missed chances to simplify and apply them. Reuse code that
> makes sense to share. Remove dead code. Find leftovers from earlier
> attempts. Find shortcuts you took, and do them properly. No changes may be
> necessary.

**4e. Open the PR and get it reviewed.** Fill in the template. Get a
fresh-context review and a review from a named-model reviewer from another
vendor. Read the full diff for roughly the first 10 PRs
([0020](../decisions/0020-risk-and-merge-enforced-by-github.md)). You are
collecting the corrections you keep making.

**4f. Merge and deploy.** The agent waits until `/health` serves the merge
SHA, then posts the report: intent, door, cost, PR link.

**4g. Retro, which is where the factory gets built.**

- The agent files friction for whatever slowed it down.
- Encode each correction you made twice: a lint rule or check, then a test,
  then a principle page.
- Add anything the next feature will reuse to `primitives.yaml`.
- Confirm the outcome metric fires in production.

## 5. Raise autonomy as trust is earned

Follow the trust ladder in
[0020](../decisions/0020-risk-and-merge-enforced-by-github.md) and the stages in
[adoption-stages.md](./adoption-stages.md). Add containment, then scheduled
loops, once friction outpaces manual handling, usually around feature 3–5.

## Mistakes to avoid

- Writing a long spec or a full `primitives.yaml` before the code exists.
- Letting the agent choose the data model or public URLs on its own.
- Building sweeps, reapers, or gardeners before there is friction or cruft
  for them to work on.
- Accepting tests that only check that the UI rendered. Ask for one workflow
  test per behaviour that could break silently.
- Skipping the outcome metric because the feature "obviously works".
