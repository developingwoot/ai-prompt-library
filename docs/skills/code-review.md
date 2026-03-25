# Skill: Code Review

## Purpose

Review recently written code as a ruthless senior reviewer, checking for security vulnerabilities, spec compliance, and correctness before it is committed.

## When the agent should use this

- The user says "review this code", "check this before I commit", "do a code review", or "look for issues in X"
- A feature has just been completed and the user wants a fresh set of eyes on it
- Best run in a **fresh conversation** — not the same session that wrote the code. The model that wrote the code is biased toward its own output and will overlook the same edge cases it missed when generating.

## Opening interview

Ask these questions before proceeding. Skip any that are clearly answered by SPEC.md or the codebase.

1. **Which files or directories should be reviewed?** Name the specific files, a directory, or a feature area.
   *If skipped: review all files changed in the most recent feature implementation visible in the conversation or git diff.*

2. **Is there a specific concern that prompted this review?** For example, a security-sensitive area, a tricky edge case, or a pattern the user is unsure about.
   *If skipped: apply the full review protocol in priority order with no area deprioritized.*

## Protocol

You are acting as a senior code reviewer. The code was written by an AI assistant. Your job is to find issues the original author might have missed. You have no attachment to this code — be ruthless.

1. Before reviewing, read:
   - [AGENTS.md](AGENTS.md) — the project rules this code must follow
   - [docs/SPEC.md](docs/SPEC.md) — specifically the section describing the feature this code implements
   - The files identified in the opening interview

2. Review in this exact priority order:

   **1. Security vulnerabilities**
   - Can any input be used to exploit the system?
   - Can user A access, modify, or delete user B's data by manipulating IDs, parameters, or request bodies? Check every query that uses a user-supplied ID.
   - Are auth and authorization checks present on every route that needs them?
   - Are secrets, tokens, or passwords exposed anywhere — in responses, logs, or error messages?

   **2. Spec compliance**
   - Does the code actually implement what the spec describes?
   - Are there edge cases described in the spec that the code doesn't handle?
   - Does the code do anything the spec doesn't ask for?

   **3. AGENTS.md rule violations**
   - Check every rule in AGENTS.md against the code
   - Flag any violation with the specific rule being broken

   **4. Error handling**
   - Unhandled promise rejections, missing try/catch, unvalidated inputs
   - Error responses that expose stack traces or internal details
   - Error shapes that are inconsistent with the rest of the codebase

   **5. Language and framework issues** — based on the tech stack in the spec
   - Type safety problems appropriate to the language (e.g. `any` types in TypeScript, missing null checks)
   - Framework anti-patterns (e.g. data fetching that bypasses framework conventions)
   - Anything that works now but will cause problems as the codebase grows

   **6. Logic errors**
   - Code that will break on null, undefined, empty arrays, or boundary conditions
   - Off-by-one errors, race conditions, or assumptions about data that aren't validated

   **7. Pattern consistency**
   - Does this match the patterns established in the rest of the codebase?
   - If it deviates, is the deviation justified or accidental?

3. Apply these rules throughout:
   - Do not compliment the code
   - Be specific — "line 34 in auth.ts will throw if user is undefined because the query doesn't check for null before accessing .id" not "needs better error handling"
   - For each issue, rate severity: **Critical / High / Medium / Low**
   - If you find zero Critical or High issues, say so explicitly — that is meaningful signal

4. End with a clear verdict:
   - **APPROVE** — no blocking issues
   - **APPROVE WITH CHANGES** — list what must change before committing
   - **REQUEST CHANGES** — list the blockers that must be resolved

## What good output looks like

A numbered list of specific findings with file names, line references, and severity ratings. The security section explicitly confirms whether user A can access user B's data — not just "auth looks fine." The verdict is clear and actionable.

## Watch out for

- **Politeness overriding the brief.** Deep/extended models tend to soften findings despite the "do not compliment" instruction. Push back if findings feel hedged.
- **Skipping object-level authorization.** Confirming a route has an auth check is not enough — the review must verify that the authenticated user can only touch their own data.
- **Vague findings.** Any finding that doesn't name a file, line, and specific failure mode is not acceptable.
- **Code quality without spec compliance.** A review that only checks code style while ignoring whether the feature matches the spec misses half the job.
