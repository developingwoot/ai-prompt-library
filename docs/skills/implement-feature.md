# Skill: Implement Feature

## Purpose

Guide Claude through a structured plan-then-build workflow to implement a new feature safely, with full codebase orientation before any code is written.

## When Claude should use this

- The user says something like "implement X", "build X", "add X feature", or "I need X to work"
- A new piece of functionality needs to be created (not a bug fix or refactor)
- Use plan mode first; switch to act mode only after the user approves the plan

## Opening interview

Ask these questions before proceeding. Skip any that are clearly answered by SPEC.md, PROGRESS.md, DECISIONS.md, or the codebase.

1. **What is the feature?** Describe what it should do from the user's perspective.
   *If skipped: infer the feature name and scope from the user's message and the spec. State your interpretation before planning.*

2. **Are there any constraints or requirements not captured in the spec?** For example, a specific API contract, a third-party service to integrate, or a deadline-driven scope cut.
   *If skipped: assume the spec and codebase are the complete source of truth.*

3. **Should this be one implementation session or broken into steps?** Some features are large enough to warrant a phased approach.
   *If skipped: propose a single implementation if the scope looks contained (5 files or fewer); suggest breaking it up if it looks larger.*

## Protocol

### Plan mode (run this first)

1. Read [docs/SPEC.md](docs/SPEC.md) and locate the section describing this feature. Note the acceptance criteria, user journeys it belongs to, and any constraints mentioned.

2. Read [docs/PROGRESS.md](docs/PROGRESS.md) and verify that every dependency this feature requires is marked as Implemented. If any dependency is missing, stop and tell the user which dependencies need to be built first before proceeding.

3. Scan the existing codebase to understand what files are present, what patterns are established, and what can be reused. Do not skip this step — generic plans that ignore existing code are a failure mode.

4. Read CLAUDE.md and identify any rules especially relevant to this feature. If this feature touches auth, payments, or sensitive data, call out the specific rules that apply by name.

5. Present your plan:
   - The files you will create or modify, and why each one is needed
   - How this connects to existing code and patterns
   - Anything that concerns you or feels ambiguous
   - If the spec describes the user's skill level as intermediate or below, explain the key decisions in plain terms. If the user is experienced, keep the plan concise.

6. Wait for the user to approve the plan before writing any code. Do not proceed to act mode on your own.

### Act mode (only after approval)

1. Implement the feature following all rules in CLAUDE.md. Stay within the scope of the approved plan — do not modify files outside it.

2. Before running tests, check whether required setup is in place: migrations, seed data, environment variables. If anything is missing, stop and ask the user whether to run migrations or seed data in this session. Do not mark tests as failed because of missing setup.

3. Run existing tests to confirm nothing is broken by your changes.

4. If no tests exist yet, call this out explicitly and propose a minimal smoke/regression test set before continuing. Do not skip the testing step — note the absence and recommend creating baseline tests.

5. Write new tests for what you just built. Test behavior and outcomes, not internal implementation details.

6. Tell the user what to manually verify to confirm the feature works, and note which user journey from the spec this feature advances.

## What good output looks like

**In plan mode:** A clear, scoped proposal that references specific existing files by name. The user should be able to read the plan and know exactly what will change before anything changes.

**In act mode:** A clean implementation that follows the approved plan. Existing tests still pass. New tests cover the happy path, invalid inputs, and relevant edge cases. The manual testing instructions let the user verify the feature works in under five minutes.

## Watch out for

- **Skipping orientation.** Claude jumping straight to a plan based on the feature name alone, without reading the spec or scanning the codebase.
- **Scope creep in the plan.** If the plan proposes modifying more than 5–6 files for a single feature, the feature may need to be broken into smaller pieces.
- **Skipping the test run.** After implementation, Claude must run existing tests — not assume they still pass.
