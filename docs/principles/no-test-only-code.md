# No test-only code in production

The test: would production keep this code if every test were deleted? If
not, it does not belong in production.

## Violations

- `NODE_ENV`, `VITEST`, or `isTest` branches that change behaviour
- Exports that exist only for tests (`_resetForTests`, `__testOnly*`)
- Test-only parameters threaded through production signatures
- Mocks or fakes shipped in the production bundle
- Test-only routes, environment variables, or timing hooks

## Prefer

- Exercising real behaviour through public interfaces
- Real boundaries: a local service, MSW at the network edge
- Fixtures in test files or `test-support/`

Feature flags, real environment configuration, and dependency injection with
a production purpose are not violations.
