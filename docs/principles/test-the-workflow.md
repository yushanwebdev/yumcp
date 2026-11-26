# Test the workflow

One test is one workflow: the setup, the actions, and the assertions that
prove it. An assertion needs an oracle the production code does not share.

## Rules

- "No new tests needed" is a normal answer for copy, layout, config, and
  refactors that keep behaviour.
- Add a test for behaviour that can break silently (auth, permissions,
  billing, data integrity), and one regression test per bug fix.
- Do not pin exported constants, copy the production helper into a test, or
  assert that the UI rendered.
- Use flat `test(...)` blocks with inline setup. Tests run offline.
- In the PR, justify each new long-lived test: the behaviour it protects and
  why existing tests miss it.

## Example

```ts
// Wrong: the expected value is the production constant
expect(isResetMessage(RESET_MESSAGE)).toBe(true)

// Right: an independent oracle that fails if normalisation changes
expect(isResetMessage(RESET_MESSAGE.replace(/\.$/, ''))).toBe(true)
```
