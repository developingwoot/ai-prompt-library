# Skill: Pre-Deploy Security Checklist

## Purpose

Run a structured security sweep across the full codebase before every deployment, checking for critical vulnerabilities that must be resolved before shipping.

## When Claude should use this

- The user says "pre-deploy check", "security checklist", "is this safe to deploy", or "check before I ship"
- A feature or milestone is complete and the user is preparing to deploy
- Run in a **fresh conversation** — a model that worked on the codebase has blind spots toward its own output, which is especially dangerous in a security context
- This is a quick sweep; for a deeper audit use the OWASP skill instead

## Opening interview

Ask these questions before proceeding. Skip any that are clearly answered by SPEC.md or CLAUDE.md.

1. **Is there a specific area of concern going into this deploy?** For example, a new auth flow, a new payment integration, or a route added since the last deploy.
   *If skipped: apply the full checklist to the entire codebase with no area prioritized.*

2. **Has the dependency audit been run recently?** Some environments require a manual step (e.g. `npm audit`, `pip audit`).
   *If skipped: run it as part of the checklist and include the output in the findings.*

## Protocol

Run this in a fresh conversation. Read the full codebase, [docs/SPEC.md](docs/SPEC.md) for the tech stack and auth approach, and [CLAUDE.md](CLAUDE.md) for security-related rules before starting. Adapt every check to the specific framework and database defined in the spec.

Work through the checklist below in order. For every item — passing or failing — mark it explicitly. Do not skip items and do not mark anything as passing without actually checking.

---

**CRITICAL — do not deploy if any of these exist:**

- [ ] Hardcoded API keys, secrets, or passwords anywhere in the code (including comments and example values that look real)
- [ ] Environment variables referenced without validation at startup
- [ ] Auth middleware missing on routes that should be protected
- [ ] User input used directly in database queries without the ORM's parameterization (injection risk)
- [ ] Passwords, tokens, or sensitive data logged anywhere
- [ ] User A can access User B's resources (broken object-level authorization) — check every route that accepts a resource ID and verify the query scopes to the authenticated user

**HIGH — fix before shipping:**

- [ ] Missing input validation on any route that accepts user data
- [ ] CORS configured to allow `*` for browser-facing APIs with auth or PII — `*` is only acceptable for non-auth, purely public, read-only assets; otherwise lock to known origins via an env-driven whitelist
- [ ] Error responses that expose stack traces, file paths, or internal details
- [ ] No rate limiting on auth routes
- [ ] Tokens or sessions stored insecurely (missing `httpOnly`, `secure`, or `sameSite` flags if using cookies)

**MEDIUM — fix soon after launch:**

- [ ] Missing HTTPS enforcement
- [ ] No request size limits
- [ ] Dependency vulnerabilities — run the package manager's audit command and include the output
- [ ] Sensitive data returned in API responses that shouldn't be (e.g. password hash, internal IDs)
- [ ] Missing security headers (Content-Security-Policy, X-Content-Type-Options, etc.)

**CI/CD and secrets:**

- [ ] Secrets only injected via the platform's secret manager; never echoed, printed, or persisted in logs or artifacts
- [ ] `.env` and real-looking values are not committed; build artifacts do not bundle `.env`
- [ ] Pipeline fails fast if required secrets or environment variables are missing

---

For each issue found, provide:

- The file name and location
- The risk explained in one sentence
- A one-line fix or a clear remediation step

For each check that passes, mark it explicitly in the output.

## What good output looks like

A complete checklist where most items are marked as passing, with specific findings on the ones that don't. Any Critical finding is a hard blocker — do not deploy until it's fixed. The output should feel like a pre-flight check, not a code review.

## Watch out for

- **False positives from framework misunderstanding.** Deep/extended models sometimes flag things that aren't real issues for the specific ORM or framework in the spec. Verify findings against the actual framework docs before treating them as blockers.
- **Skipping object-level authorization.** This is the single most important item and must be checked explicitly for every route that accepts a resource ID — not just confirmed at the middleware level.
- **Marking items as passing without checking.** Every checklist item must be actively verified, not assumed to pass because the code "looks fine."
