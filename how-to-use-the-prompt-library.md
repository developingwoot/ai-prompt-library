# How to Use the Agent-Native Prompt Library

## A Practical Guide for Every Stage of Your Project

---

## Before You Start

This guide walks you through how to use the prompt library at every stage of building an app — from first idea to deployment. Each section tells you exactly which prompts to run, in what order, and why.

### What You Need

You'll need an **AI coding agent** (e.g., Claude Code, Cursor, Windsurf, Cline, Aider) or a **chat-based AI interface**. In tools with workspace access, the agent can read your project files directly — just reference them by name or path. In chat-based interfaces, you'll need to paste or upload file contents when a prompt asks for them.

### Plan Mode and Act Mode

This is the most important habit in the entire workflow. Before the agent creates or changes code, use **plan mode** first — the agent will analyze your codebase and propose what it wants to do without touching any files. Read the plan, ask questions, and adjust. Only when you're satisfied do you let the agent implement. Some tools have built-in plan/act mode toggles (e.g., Claude Code); if yours doesn't, instruct the agent to "propose a plan and wait for approval before writing any code."

This applies to all feature-building prompts ([Section 2](agent-native-prompt-library%202.3.md#section-2-building-features)), test-writing prompts ([Section 4](agent-native-prompt-library%202.3.md#section-4-testing)), and refactoring prompts ([Section 8](agent-native-prompt-library%202.3.md#section-8-refactoring--code-health)). It does not apply to [Section 0](agent-native-prompt-library%202.3.md#section-0-codebase-onboarding-existing-apps) and [Section 1](agent-native-prompt-library%202.3.md#section-1-project-setup-new-apps), which generate documents from scratch.

### The Four Files That Run Everything

The entire system revolves around four markdown files. Think of them as your project's memory — they're how the agent maintains context across conversations, and they're how you stay in control of what gets built.

| File | What It Does | Where It Lives |
| ------ | ------------- | ---------------- |
| **SPEC.md** | Describes what your app is, what it does, who the users are, and what's out of scope | `docs/SPEC.md` |
| **PROGRESS.md** | Your build checklist — what's done, what's next, what's blocked, and how many sessions remain | `docs/PROGRESS.md` |
| **DECISIONS.md** | A log of every major technical decision and why it was made | `docs/DECISIONS.md` |
| **AGENTS.md** | The rulebook your AI agent follows every session — coding standards, forbidden actions, and how sessions start and end. References the other three files. | Project root |

Without these files, every new conversation starts from zero and you lose all accumulated context. The prompts in this library automatically detect which AI tool(s) you're using and generate the appropriate wrapper file — so your tool reads AGENTS.md at session start without any manual setup. Keeping these files updated is what makes this workflow sustainable over weeks and months.

---

## Starting a New Project

Use this when you're starting from an idea with no code yet. The goal is to create all four foundation files before you write a single line of code. Each step produces a file the next step depends on, so complete them in order.

### Step 1: Create your spec → [Prompt 1.1](agent-native-prompt-library%202.3.md#11--spec-file-generator)

This is a conversation, not a one-shot prompt. You describe your idea, your technical background, and your preferred tech stack. The agent will interview you with 3-5 clarifying questions before writing anything. Take the interview seriously — the quality of everything downstream depends on how thorough your spec is.

If you're unsure what tech stack to use, run [**Prompt 1.5 (Tech Stack Decision Helper)**](agent-native-prompt-library%202.3.md#15--tech-stack-decision-helper) first, then come back to [1.1](agent-native-prompt-library%202.3.md#11--spec-file-generator) with your stack decided.

**Output:** `docs/SPEC.md`

### Step 2: Create your build plan → [Prompt 1.2](agent-native-prompt-library%202.3.md#12--progressmd-initializer)

This breaks your spec into session-sized tasks, ordered by dependency. Each task has a testable outcome so you know when it's done. The ordering prioritizes completing your first full user journey end-to-end, so you have something real to click through early.

If the total session count exceeds 20-25, your spec is probably too ambitious for v1. Trim the spec before moving on.

**Output:** `docs/PROGRESS.md`

### Step 3: Document your founding decisions → [Prompt 1.3](agent-native-prompt-library%202.3.md#13--decisionsmd-initializer)

This records why you chose your frameworks, database, auth strategy, and deployment target — including honest reasoning like "I already know React" or "this was the simplest option." Future sessions reference this so the agent doesn't second-guess decisions that have already been made.

**Output:** `docs/DECISIONS.md`

### Step 4: Create your rulebook → [Prompt 1.4](agent-native-prompt-library%202.3.md#14--agentsmd-scaffolding)

This is the last setup step because AGENTS.md references the other three files, so they need to exist first. It generates the rules the agent follows in every session — coding standards, security rules, what's forbidden, how sessions start, and how sessions end. The prompt automatically detects your tool and creates the appropriate wrapper file so your tool reads AGENTS.md at session start.

**Output:** `AGENTS.md` in project root (plus a tool-specific wrapper file generated automatically)

After these four steps, you're ready to start building. Move on to the **Starting a Coding Session** section below.

---

## Onboarding an Existing Project

Use this when you have a codebase that already exists but wasn't built with this workflow. The goal is the same as above — create the four foundation files — but by reading your existing code rather than starting from an idea.

### Step 1: Reverse-engineer your spec → [Prompt 0.1](agent-native-prompt-library%202.3.md#01--the-codebase-cartographer-reverse-engineering-specmd)

The agent scans your entire codebase and produces a SPEC.md based on what actually exists today. Half-built features are marked as incomplete. This is meant to be an honest mirror of your current app, not an aspirational document.

**Output:** `docs/SPEC.md`

### Step 2: Map your progress and decisions → [Prompt 0.2](agent-native-prompt-library%202.3.md#02--state-of-the-union-reverse-engineering-progress--decisions)

This produces both files in one step. PROGRESS.md maps what's done, what's half-built, and what technical debt exists. DECISIONS.md captures the architectural choices baked into your codebase with honest reasoning about why they were likely made.

**Important:** The DECISIONS.md entries from this prompt are inferred from code patterns, not verified institutional knowledge. Every "Why" entry is the AI's best guess. Verify these against anyone who was involved in building the original app before treating them as settled fact.

**Output:** `docs/PROGRESS.md` and `docs/DECISIONS.md`

### Step 3: Extract your coding rules → [Prompt 0.3](agent-native-prompt-library%202.3.md#03--the-rule-extractor-reverse-engineering-agentsmd)

The agent analyzes your code patterns — error handling, naming conventions, file structure, state management — and writes an AGENTS.md that documents your actual conventions. This ensures AI-generated code matches your existing style rather than introducing new patterns.

**Output:** `AGENTS.md` in project root (plus a tool-specific wrapper file generated automatically)

### Step 4: Find where your code breaks its own rules → [Prompt 0.4](agent-native-prompt-library%202.3.md#04--the-delta-audit-alignment-check)

Now that you have documented standards, this prompt compares your actual code against them and produces a prioritized list of inconsistencies. This becomes your first cleanup sprint — an actionable hit list of alignment fixes ranked by severity.

**Output:** A prioritized list of fixes (not a file — this is an action item list you work through)

After these four steps, your existing project is fully onboarded into the workflow. From here, everything works the same as a new project — use the session management, feature building, and review prompts described below.

---

## Starting a Coding Session

Do this every time you sit down to work on the project.

If you set up AGENTS.md with [Prompt 1.4](agent-native-prompt-library%202.3.md#14--agentsmd-scaffolding) (new project) or [Prompt 0.3](agent-native-prompt-library%202.3.md#03--the-rule-extractor-reverse-engineering-agentsmd) (existing project), your agent runs a session start ritual automatically — it checks recent git history, reviews your progress file, and asks what you want to work on. For most sessions, this is enough.

If you need more context — for example, you haven't touched the project in a few days, or you're starting a fresh conversation — use [**Prompt 7.2 (Session Start Orientation)**](agent-native-prompt-library%202.3.md#72--session-start-orientation). This gives you a fuller briefing: what was last completed, what's in progress and where it was left off, the next three priorities from your build plan, any blocked items, and roughly how many sessions remain.

The key discipline is that the agent should ask you what to work on, not propose its own agenda. You drive the session.

---

## Ending a Coding Session

Do this every time you stop working on the project, before closing your AI agent.

Use [**Prompt 7.1 (End-of-Session Handoff)**](agent-native-prompt-library%202.3.md#71--end-of-session-handoff). This does four things:

1. Updates PROGRESS.md — moves completed items to "Implemented," marks in-progress items with notes on where you left off, adds any new items discovered during the session.
2. Suggests a git commit message with a type prefix and bullet points describing what changed.
3. Updates DECISIONS.md if any architectural decisions were made during the session.
4. Recommends what to tackle first next time.

This is non-negotiable. If you skip it, the next session starts without knowing what happened, and you lose the context that makes this entire workflow effective. It takes two minutes and saves you twenty minutes of reorientation next time.

---

## Building a New Feature

This is the day-to-day workflow you'll repeat for every item on your build plan. Always use plan mode first, review the plan, then let the agent implement.

### Pick the right prompt for what you're building

| What You're Building | Prompt to Use |
| --------------------- | --------------- |
| Database schema (do this early) | [**2.2 — Database Schema Design**](agent-native-prompt-library%202.3.md#22--database-schema-design) |
| Auth system (do this early) | [**2.5 — Authentication Implementation**](agent-native-prompt-library%202.3.md#25--authentication-implementation) |
| API endpoints | [**2.3 — API Endpoint Implementation**](agent-native-prompt-library%202.3.md#23--api-endpoint-implementation) |
| UI components or pages | [**2.4 — UI Component Implementation**](agent-native-prompt-library%202.3.md#24--ui-component-implementation) |
| Anything else | [**2.1 — Feature Implementation**](agent-native-prompt-library%202.3.md#21--feature-implementation) |

[Prompt 2.1](agent-native-prompt-library%202.3.md#21--feature-implementation) is the general-purpose workhorse. The others are specialized versions with domain-specific guidance — [2.2](agent-native-prompt-library%202.3.md#22--database-schema-design) includes schema validation against your user journeys, [2.3](agent-native-prompt-library%202.3.md#23--api-endpoint-implementation) covers object-level authorization checks, [2.4](agent-native-prompt-library%202.3.md#24--ui-component-implementation) handles all four UI states (loading, error, empty, success), and [2.5](agent-native-prompt-library%202.3.md#25--authentication-implementation) has an extensive security checklist.

### The build cycle for each feature

1. **Orient** — Every feature prompt starts by having the agent read the spec, check the progress file for dependency readiness, scan the existing codebase for patterns, and review the project rules. Don't skip this. If the agent proposes a plan without referencing specific existing files, push back.
2. **Plan** — The agent proposes what files it will create or modify, how the feature connects to existing code, and anything that seems ambiguous. Review this carefully. If the plan touches more than 5-6 files, the feature may need to be broken into smaller pieces.
3. **Act** — After you approve the plan, the agent implements the feature, runs existing tests to make sure nothing broke, and writes new tests for what it just built.
4. **Verify** — The agent tells you what to manually test. Do it. Automated tests catch regressions; manual testing catches things that are technically working but feel wrong.

### Write tests immediately after each feature

Use [**Prompt 4.1 (Write Tests for Code Just Built)**](agent-native-prompt-library%202.3.md#41--write-tests-for-code-just-built) in the same session, while the agent still has full context of what it wrote. Tests should verify behavior (what the code does), not implementation (how it does it). If you want to check whether your tests are actually meaningful, follow up with [**Prompt 4.2 (Fix Tests That Pass for the Wrong Reason)**](agent-native-prompt-library%202.3.md#42--fix-tests-that-pass-for-the-wrong-reason).

### When you need to modify risky existing code

If you need to change something complex or fragile — code you've lost context on, or code where your gut says "this feels risky" — use [**Prompt 2.6 (Explain Before You Touch)**](agent-native-prompt-library%202.3.md#26--explain-before-you-touch) instead of jumping straight into [2.1](agent-native-prompt-library%202.3.md#21--feature-implementation). This forces the agent to prove it understands the file, trace all dependencies, and identify fragile parts before proposing any changes.

### End the session

Finish with [**Prompt 7.1**](agent-native-prompt-library%202.3.md#71--end-of-session-handoff) as described above.

---

## When You Reach a Milestone

A milestone is a natural boundary in your project — you've finished the auth system, the core API is complete, the full backend is done and you're about to start the frontend, or you're approaching deployment. At these points, pause building and run reviews before moving on.

### Architecture review → [Prompt 3.3](agent-native-prompt-library%202.3.md#33--architecture-review)

Run this in a **fresh conversation** (not the one that built the code). This evaluates whether the architecture can support what you need to build next. It checks separation of concerns, pattern consistency, AGENTS.md compliance across the codebase, decision log validity, missing pieces, and tech debt. The output is a verdict (ready / ready with changes / blocked) and a prioritized list of at most five things to fix before starting the next phase.

### Code review → [Prompt 3.1](agent-native-prompt-library%202.3.md#31--standard-code-review)

Also in a **fresh conversation**. This is a file-level review rather than a project-level review. It checks security vulnerabilities, spec compliance, AGENTS.md rule violations, error handling, type safety, logic errors, and pattern consistency. Run this on the code that was built during the milestone.

### Why fresh conversations matter

The model that wrote the code is biased toward its own output. It will overlook the same edge cases it missed when generating. Running reviews in a fresh conversation gives the agent a genuinely different perspective — it approaches the code as a reviewer, not as the author.

### Audit your rules → [Prompt 7.4](agent-native-prompt-library%202.3.md#74--audit-against-agentsmd)

This is a good time to run an AGENTS.md audit. Over the course of several sessions, patterns drift. This prompt checks every rule in your AGENTS.md against the actual codebase and reports what's followed consistently, what has exceptions, and what's being violated. It also helps you evolve your rules — sometimes a rule is consistently violated because it turned out to be impractical, not because the code is wrong.

---

## Running a Security Audit

Run this after your core API and auth system are complete, and again before any deployment that involves authentication, payments, or sensitive user data.

### OWASP Top 10 Audit → [Prompt 5.2](agent-native-prompt-library%202.3.md#52--owasp-top-10-audit)

Run in a **fresh conversation**. This is a comprehensive, category-by-category audit against the OWASP Top 10 web application security risks. The most important category is A01 (Broken Access Control) — for every route that accepts a resource ID, the audit should verify that the query scopes to the authenticated user and show you the specific query. This is the number one vulnerability in AI-generated code.

### Cross-Model Security Review → [Prompt 3.2](agent-native-prompt-library%202.3.md#32--cross-model-security-review)

This is the one prompt in the library you deliberately run outside your primary AI agent. Open a different model family (e.g., if you use Claude, try ChatGPT or Gemini), paste the relevant code files and the context block from the prompt, and send it. The point is to get a genuinely different perspective from a different model family. Different models have different blind spots, and security is exactly where you want those blind spots exposed.

### What about [Prompt 5.1 (Pre-Deploy Security Checklist)](agent-native-prompt-library%202.3.md#51--pre-deploy-security-checklist)?

The pre-deploy checklist (covered in the next section) is a faster, shallower sweep. The OWASP audit is deeper and more thorough. If you're only going to do one, do the OWASP audit. Ideally, do the OWASP audit once after your core API is complete, then use the pre-deploy checklist as a quicker gate before each deployment.

---

## Before You Deploy

Run these checks before going live. They can also be repeated before any major deployment after the initial launch.

### Step 1: Audit environment variables → [Prompt 5.3](agent-native-prompt-library%202.3.md#53--environment-variable-audit)

Scan the codebase for every environment variable referenced anywhere. This checks that each one is validated at startup (not just when a route happens to use it), that none are hardcoded, that a `.env.example` exists with safe placeholder values, that `.env` is in `.gitignore`, and that no secrets are exposed to the frontend. The output includes a ready-to-commit `.env.example` file.

### Step 2: Run the pre-deploy security checklist → [Prompt 5.1](agent-native-prompt-library%202.3.md#51--pre-deploy-security-checklist)

Run in a **fresh conversation**. This is a structured sweep organized by severity — Critical items that block deployment, High items to fix before shipping, and Medium items to address soon after launch. It checks for hardcoded secrets, missing auth middleware, injection risks, broken object-level authorization, insecure token storage, missing rate limiting, and more. Every item gets an explicit pass/fail.

### Step 3: Audit dependencies → [Prompt 8.2](agent-native-prompt-library%202.3.md#82--dependency-audit)

AI assistants are eager to add packages. This prompt checks every dependency for actual usage, identifies packages that could be replaced with native code, flags duplicates, and runs the package manager's security audit. Expect to find at least 2-3 unnecessary dependencies.

### Step 4: Run the OWASP audit if you haven't already → [Prompt 5.2](agent-native-prompt-library%202.3.md#52--owasp-top-10-audit)

If you already ran this at the security audit stage, you can skip it here unless significant code has changed since then. If you haven't run it yet, do it now. Don't deploy an app with authentication or sensitive data without a comprehensive security review.

### Step 5: Cross-model security review → [Prompt 3.2](agent-native-prompt-library%202.3.md#32--cross-model-security-review)

Same as above — if you already ran this and the code hasn't changed significantly, you can skip it. If this is your first deployment, run it.

---

## A Note on Redundancy

You may have noticed that the security audit and pre-deployment sections overlap. This is intentional in the prompt library but worth clarifying for how you actually use them in practice.

The [**OWASP audit (5.2)**](agent-native-prompt-library%202.3.md#52--owasp-top-10-audit) and [**cross-model review (3.2)**](agent-native-prompt-library%202.3.md#32--cross-model-security-review) are deep, thorough checks you run once after your core API is complete. They take real time and effort. The [**pre-deploy checklist (5.1)**](agent-native-prompt-library%202.3.md#51--pre-deploy-security-checklist) and [**environment variable audit (5.3)**](agent-native-prompt-library%202.3.md#53--environment-variable-audit) are faster gates you can run before each deployment. Think of the first pair as the comprehensive exam and the second pair as the pre-flight check.

If you're doing a single initial deployment, run them all. For subsequent deployments where the scope of change is smaller, the pre-deploy checklist and environment variable audit are usually sufficient unless you've made significant changes to auth, authorization, or data handling.

---

## When Things Go Wrong

| Problem | Prompt to Use |
| --------- | --------------- |
| You hit an error | [**6.1 — The Debugging Formula**](agent-native-prompt-library%202.3.md#61--the-debugging-formula) (fill in all sections before sending) |
| You've been debugging 30+ minutes in circles | [**6.2 — Fresh Context Debug**](agent-native-prompt-library%202.3.md#62--fresh-context-debug) (start a new conversation) |
| Database errors or migration issues | [**6.3 — Database Debug**](agent-native-prompt-library%202.3.md#63--database-debug) |
| The agent suggests the wrong framework or forgets the project | [**7.3 — Context Collapse Recovery**](agent-native-prompt-library%202.3.md#73--context-collapse-recovery) (start a new conversation) |
| Tests pass but you don't trust them | [**4.2 — Fix Tests That Pass for the Wrong Reason**](agent-native-prompt-library%202.3.md#42--fix-tests-that-pass-for-the-wrong-reason) |
| A file has grown too complex | [**8.1 — Targeted Refactor**](agent-native-prompt-library%202.3.md#81--targeted-refactor) (plan mode first) |

---

## Best Practices

**Your four files are your project's memory.** SPEC.md, PROGRESS.md, DECISIONS.md, and AGENTS.md are what make AI-assisted development sustainable across sessions. Keep them updated. If they fall out of date, the agent's output quality drops because it's working from stale context.

**Plan before you act.** Always review what the agent proposes before letting it write code. This is especially critical for auth, database schema, and anything touching sensitive data. The plan/act workflow is your single strongest safeguard against AI mistakes.

**Fresh conversations for reviews.** The model that wrote the code is biased toward its own output. Code reviews, security audits, and architecture reviews should always happen in a new conversation where the agent approaches the code without attachment.

**End every session with [Prompt 7.1](agent-native-prompt-library%202.3.md#71--end-of-session-handoff).** This is how context survives between sessions. Two minutes of cleanup saves twenty minutes of reorientation next time.

**Don't skip the spec.** It's tempting to jump straight to building. Every downstream prompt depends on a solid spec. An extra 30 minutes on [Prompt 1.1](agent-native-prompt-library%202.3.md#11--spec-file-generator) saves hours of rework when features are vaguely defined and the agent interprets them differently than you intended.

**Build the first user journey end-to-end first.** Your progress file is ordered to prioritize this. Having something real to click through early gives you confidence that the architecture works and exposes problems when they're cheap to fix.

**Tests are not optional.** Write them after every feature ([Prompt 4.1](agent-native-prompt-library%202.3.md#41--write-tests-for-code-just-built)). Validate them periodically ([Prompt 4.2](agent-native-prompt-library%202.3.md#42--fix-tests-that-pass-for-the-wrong-reason)). Before a milestone, audit your coverage ([Prompt 4.4](agent-native-prompt-library%202.3.md#44--test-coverage-audit)). AI-generated code needs more testing, not less, because the person reviewing it didn't write it and may miss subtle issues.

**When the agent gets confused, start fresh.** If the agent starts suggesting the wrong framework, contradicting earlier decisions, or duplicating code that already exists, don't fight it in the same conversation. Use [Prompt 7.3 (Context Collapse Recovery)](agent-native-prompt-library%202.3.md#73--context-collapse-recovery) to start a new conversation and rebuild context cleanly. This happens naturally in long sessions and is not a failure — it's a known limitation you manage around.
