# Review and Recommend

Use this for any product or architecture call
([0005](../decisions/0005-review-and-recommend-adrs.md)). Adapted from Kent's
skill (course E1).

## Prompt

> Question: <the decision>.
>
> 1. Gather context first: docs/intent.md, relevant ADRs and primitives,
>    production errors, usage data, open issues, and git history. Say what you
>    could not access.
> 2. Give numbered options. For each: effort (easy / medium / hard), door
>    (one-way or two-way, and why), main risk, and what it costs to reverse.
> 3. Recommend one option.
> 4. Name the *right* fix if all else were equal. If it differs from your
>    recommendation, explain why.
> 5. For the recommended option, give a refutation test (what observation
>    within N days would show it was wrong) and, if it is one-way, a
>    reversibility plan (flag, dual-write window, or codemod).
>
> Do not implement anything until I choose.

## After you choose

- Write an ADR if the next agent would otherwise propose the rejected option
  again. Use `docs/decisions/0000-template.md`.
- Add the ADR to the `adrs` list of each primitive it shapes.
- If the decision is a product bet, put the refutation test in a
  `docs/bets/` file as its threshold
  ([0024](../decisions/0024-outcomes-measured-per-bet.md)).
- Look for options the agent missed. Adding one is the part of this work
  that is most yours.
