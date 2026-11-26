# Engineering principles

Short, durable rules. One page per principle. Load only the page the task
needs. Add a principle only after correcting an agent the same way twice,
and prefer a check over a principle when the repo can enforce it.

| Principle                                              | Open it when                                                              |
| ------------------------------------------------------ | ------------------------------------------------------------------------- |
| [Cleanup](./cleanup.md)                                | A new way replaces an old one                                             |
| [Two-way and one-way doors](./two-way-doors.md)        | Deciding how much review a change needs                                   |
| [Fail loudly at the right layer](./fail-loudly.md)     | An error could be swallowed so a partial result continues                 |
| [Test the workflow](./test-the-workflow.md)            | Adding or reshaping tests                                                 |
| [No test-only code in production](./no-test-only-code.md) | Adding test seams, `NODE_ENV` branches, or `*ForTests` exports         |

Factory decisions (`FNNNN`) live in [docs/factory/decisions/](../factory/decisions/).

Replace each generic example with a real one from your codebase as soon as
one exists.
