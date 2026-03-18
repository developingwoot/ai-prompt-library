# Skill: Write Tests

## Purpose

Write a complete, behavior-focused test suite for a feature that was just built, then run the tests and fix any failures before handing back.

## When Claude should use this

- The user says "write tests for X", "add tests", "now test it", or "cover this with tests"
- A feature was just implemented and the user wants test coverage added
- Best used immediately after building, while Claude still has full context of what was written

## Opening interview

Ask these questions before proceeding. Skip any that are clearly answered by the codebase, SPEC.md, or CLAUDE.md.

1. **Which feature or files should be tested?** Name the specific feature, endpoint, component, or file.
   *If skipped: test whatever was most recently implemented in this session. State your interpretation before writing.*

2. **Is there anything explicitly out of scope for this test pass?** For example, integration layers to skip, third-party services to mock, or parts of the feature not yet stable enough to test.
   *If skipped: apply the testing patterns already established in the project and use the same mocking strategy as existing tests.*

## Protocol

1. Read [docs/SPEC.md](docs/SPEC.md) and find the section describing this feature. The tests should verify the spec — the expected user-facing behavior — not just the current implementation.

2. Check the existing test files in the project. If tests already exist, match their style, structure, and framework exactly. Check CLAUDE.md for any declared testing standards.

3. Identify the user journey this feature belongs to. Tests should verify user-facing behavior, not just internal functions.

4. Before writing any tests, check that required setup is in place: migrations, seed data, environment variables. If anything is missing, stop and ask the user whether to run migrations or seed data in this session. Do not mark tests as failed because of missing setup.

5. If no tests exist in the project yet, call this out explicitly and propose a minimal smoke/regression test set before writing feature tests. Do not skip this check.

6. Write tests using the project's established testing framework. For every function or endpoint, cover:
   - **Happy path** — correct behavior with valid input
   - **Invalid input** — graceful rejection with useful error messages
   - **Edge cases** — empty arrays, null values, zero-length strings, boundary conditions, maximum values
   - **Error states** — dependency failures, not found, unauthorized
   - **Forbidden access** — for any route with auth, a valid user attempting to access the wrong resource

7. Follow these rules for every test:
   - Name each test so it completes the sentence "it should..."
   - Test behavior and return values, not implementation details — tests should pass even if the internals are rewritten
   - Do not write tests that only verify the code doesn't throw. Verify it returns the right thing with the right shape.
   - Do not mock everything — verify that mocks behave like the real dependency they replace

8. Run the tests. Fix any failures before handing back. If a test reveals a bug in the implementation, fix the implementation too and tell the user what you found.

## What good output looks like

Test names that read like specifications. A test file where you could delete the implementation and rebuild it correctly using only the tests as a guide.

## Watch out for

- **Tests that mock everything and test nothing.** A mock that always returns success doesn't verify behavior.
- **Shallow assertions.** Tests that only check status codes or truthy values without verifying the actual response shape and data.
- **Framework drift.** Using a different testing framework or pattern than what's already established in the project.
