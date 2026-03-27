# Agent-Native Prompt Library

## The Complete AI Prompt Collection

---

## How to Use This Library

Every prompt in this library follows the same format:

- **When to use it** — the specific situation this prompt is designed for
- **Model** — which AI model to run this prompt with, and why
- **The prompt** — copy, fill in the `[brackets]`, and send
- **What good output looks like** — how to know it worked
- **Watch out for** — common failure modes

### Setup Note

> These prompts are designed for use with any AI coding agent (e.g., Claude Code, Cursor, Windsurf, Cline, Aider). When a prompt references a project file like SPEC.md or PROGRESS.md, provide it to the agent using your tool's native method. In tools with workspace access, the agent can read files directly — just mention the file by name or path. In chat-based interfaces, paste or upload file contents when a prompt references a project file.
>
> **File reference syntax varies by tool.** Claude Code uses `@docs/SPEC.md`, Cursor uses `@file`, and other tools have their own conventions. This library uses plain path references like `docs/SPEC.md` — translate to your tool's syntax as needed.

### Recommended Workflow for Building Features

> **Plan before you build.** Before any code is created or modified, have the agent analyze your codebase and propose changes without touching any files. Review the plan, ask questions, and adjust until you're satisfied. Only then let the agent implement. This gives you a hard guarantee that nothing changes until you're ready.
>
> Some tools have built-in support for this workflow (e.g., Claude Code's plan/act mode toggle). If your tool does not have an explicit plan mode, instruct the agent to "propose a plan and wait for approval before writing any code" — the prompts in this library already include this instruction.
>
> This applies to any prompt where the agent is creating or modifying code — the prompts in Section 2, the test-writing prompts in Section 4, and the refactoring prompts in Section 8. Section 0 and 1 prompts generate new documents from scratch, so plan mode isn't necessary for those.

### Model Guide

| Model label | Use it for | Examples |
| ------------- | ------------ | ---------- |
| **Deep/extended reasoning** | Architecture reviews, security audits, complex debugging, pre-deploy checks | (e.g., Claude Opus, GPT-4o, Gemini Pro) |
| **Fast/code-focused** | Feature work, everyday prompting, writing code & tests, quick iterations | (e.g., Claude Sonnet, GPT-4o mini, Gemini Flash) |
| **Cross-model second opinion** | Critical code (auth, payments, PII) — fresh blind spots | Use a different model family than your primary agent |

> Pick the closest tier available in your service; names may differ.

---

## Section 0: Codebase Onboarding (Existing Apps)

---

### 0.1 — The Codebase Cartographer (Reverse-Engineering SPEC.md)

**When to use it:** When bringing an existing, undocumented (or poorly documented) codebase into this AI workflow for the first time. This prompt reads your existing code and generates the foundational spec required by all downstream prompts.

**Model:** Deep/extended reasoning tier — requires deep, cross-file architectural reasoning to understand how existing pieces fit together.

**The prompt:**
I am onboarding an existing codebase into an AI-assisted development workflow. I need you to reverse-engineer a SPEC.md file based strictly on what actually exists in the code today.

Before writing anything, scan the entire codebase:

- Identify the frontend and backend frameworks, database, and ORM
- Scan the routing/controller files to map all existing API endpoints
- Scan the data layer/ORM schema to map the existing entities
- Scan the UI components or views to understand the user-facing surface
- Look for stubbed out or half-finished features

Write a SPEC.md following this exact structure:

1. What this app is — a 2-3 sentence plain English description inferred
   from the code.
2. Who the users are and how they authenticate (look at the auth
   middleware/providers actually implemented).
3. Each major feature currently implemented, broken into bullet points
   describing exact behavior. Mark half-built features clearly as
   "(Incomplete)".
4. The 2-3 main user journeys you can definitively trace through the
   existing UI and API routes.
5. The data models (entities and their core relationships based on
   the actual database schema/migrations).
6. API endpoints that currently exist (method, path, core function).
7. Tech stack decisions (list the actual stack found in the package
   files/project configs).

CRITICAL RULE: Document reality, not aspiration. Do not invent features that seem like they "should" exist. If the app has user registration but no password reset, do not include password reset in the spec.

Format this as a clean markdown file I can save as docs/SPEC.md.

**What good output looks like:** A SPEC.md that feels like a brutally honest mirror of your current app. The user journeys should accurately reflect the actual routes and clicks a user takes today. If the output highlights gaps in your app (e.g., "Authentication exists but no authorization middleware was found"), that means the AI accurately mapped your current technical debt.

**Watch out for:** Hallucinated completeness. Deep/extended models often assume that because a controller exists, a full CRUD suite is implemented. Verify that the endpoints and features it lists are actually fully wired up, not just empty boilerplate files.

---

### 0.2 — State of the Union (Reverse-Engineering PROGRESS & DECISIONS)

**When to use it:** Immediately after 0.1. Creates your actionable backlog and documents the architectural choices already implicitly baked into the code — before you write the agent rulebook. AGENTS.md will reference these files, so they need to exist first.

**Model:** Fast/code-focused tier

**The prompt:**
Based on the SPEC.md (see docs/SPEC.md) and the actual codebase, I need to generate two files: docs/PROGRESS.md and docs/DECISIONS.md.

Task 1: Generate PROGRESS.md Scan the codebase for TODO comments, half-implemented endpoints, and features mentioned in the UI that don't have backend logic. Create a PROGRESS.md with:

- ## Implemented: Everything fully working in the spec

- ## In Progress: Things that exist in the code but are clearly unfinished (list specific missing pieces)

- ## Tech Debt & Blockers: Any obvious structural issues, hardcoded

  values, or missing test coverage you noticed while scanning.

Task 2: Generate DECISIONS.md
Look at the architecture and infer the foundational technical decisions that have already been made. For each major piece (Frameworks, Auth strategy, Database/ORM, State management), write a decision entry using this format:

- **Decision:** What was chosen
- **Status:** Active
- **Why:** The likely technical or practical reason this was chosen
  based on how it's implemented. Mark every Why entry as either
  "(inferred from code)" if this is your interpretation of the
  patterns you found, or "(stated in codebase)" if you found explicit
  documentation, comments, or a README that confirms the reasoning.
- **Consequences:** How this choice is currently shaping the codebase.
- **Revisit if:** The specific conditions under which this decision should
  be reconsidered. For example: "Revisit if we need real-time updates" or
  "Revisit if auth requirements expand beyond email/password." If there's
  no realistic scenario where this changes for v1, write "Stable for v1."

Output both files clearly separated so I can save them as docs/PROGRESS.md and docs/DECISIONS.md.

**What good output looks like:** The PROGRESS.md acts as an immediate hit-list of technical debt and broken windows. The DECISIONS.md accurately captures the reality of the app's foundation without judging it.

**Watch out for:** Two things. First, the "In Progress" section missing items because the code looks "complete" structurally but lacks business logic. You may need to manually seed the "Not Started" section of PROGRESS.md with the new features you actually intend to build next. Second, and more important: treating the DECISIONS.md entries as documented institutional knowledge when they are inferred. Every "Why" entry produced by this prompt is the AI's best interpretation of code patterns — not a verified record of why someone made a choice. Before treating any entry as settled fact, verify it against anyone who was actually there or against any documentation that predates this onboarding process.

---

### 0.3 — The Rule Extractor (Reverse-Engineering AGENTS.md)

**When to use it:** After 0.1 and 0.2. PROGRESS.md and DECISIONS.md now exist, so AGENTS.md can reference them correctly. This establishes the coding rules for future sessions by analyzing how you currently write code, ensuring AI-generated additions match your existing style.

**Model:** Fast/code-focused tier

**The prompt:**
I need to generate an AGENTS.md file for this existing project. Read the reverse-engineered spec (see docs/SPEC.md) to understand the stack, then deeply analyze the actual codebase to understand the established patterns.

Analyze the following across the codebase:

- Type safety strictness (e.g., are we using explicit types, `any`,
  interfaces vs. types, specific architectural patterns?)
- Error handling patterns (e.g., centralized middleware vs.
  try/catch everywhere)
- State management and data fetching patterns on the frontend
- File naming and directory structure conventions

Generate an AGENTS.md file with the following structure, but populate the "Code rules" section based on the ACTUAL conventions used in this codebase.

Include:

1. A project context section referencing SPEC.md, PROGRESS.md, and
   DECISIONS.md (use your tool's file import syntax if available,
   otherwise list them as files to read at session start).
2. A session start ritual — steps the agent should follow before
   writing any code:
   - Run git log --oneline -10 to understand recent work
   - Run git status to check current state
   - Review PROGRESS.md to confirm what's built and what's next
   - Ask the user what we're working on before touching any files
3. The exact tech stack found in the code.
4. Code rules: Document the specific patterns you found for routing, error handling, styling, and data access. Make these highly specific to this codebase (e.g., "API routes always return a standardized ApiResponse object", not just "handle errors well").
5. Testing configuration: Scan the codebase to identify the test framework in use, the exact commands to run the full suite and a single test file, any environment setup required before tests will pass (test database, env vars, seed data), and how to interpret the output to confirm pass or failure. Document this in a Testing section of the AGENTS.md based on what actually exists — not what should ideally exist.
6. A forbidden section: Add standard protections (no unexpected package installations, no DB schema changes without asking), but add specific guardrails based on any anti-patterns you noticed we are actively avoiding in this repo.
7. CI/CD & secrets: Never echo, print, or persist secrets in logs/artifacts. Use the platform's secret store; do not hardcode or inline secrets in build configs. Fail the pipeline if required secrets are missing. Validate env vars at startup/build time.
8. Developer experience — read the developer background in the spec and calibrate accordingly. If the developer is early-career or learning, explain what you're doing and why as you go. If the developer is experienced, be concise and just execute.
9. Disagreements — if the user asks you to do something that would violate a rule in this file, introduce a security risk, create technical debt, or go against best practices for the stack, say so before proceeding. Explain why you disagree and what you'd recommend instead. Then let the user decide. Do not silently comply with something you think is a mistake.
10. End-of-session — when the user says "we're done", follow this process:

- Update PROGRESS.md with what was completed, what's in progress, and any new items discovered
- Suggest a git commit message with type prefix and bullet points
- Update DECISIONS.md if any architectural decisions were made
- Recommend what to tackle first in the next session

CRITICAL RULE: Do not suggest idealized "best practices" if the codebase does not actually follow them. If the codebase uses a specific, slightly unorthodox pattern, document that pattern as the rule so future AI code matches the existing codebase.

After generating AGENTS.md, detect which AI coding tools are configured
for this project by scanning the project root for these indicators:

- `.claude/` directory or existing `CLAUDE.md` → Claude Code
- `.cursor/` directory → Cursor
- `.windsurfrules` file → Windsurf
- `.github/copilot-instructions.md` or `.github/workflows/` directory → GitHub Copilot
- `.clinerules` file → Cline

For each detected tool, create the appropriate wrapper file:

- **Claude Code:** Create `CLAUDE.md` in the project root containing only `@agents.md`
- **Cursor:** Create `.cursor/rules/agents.mdc` with frontmatter `alwaysApply: true` and a note to read AGENTS.md at session start
- **Windsurf:** Create `.windsurfrules` containing `@agents.md`
- **GitHub Copilot:** Copilot cannot import files — create `.github/copilot-instructions.md` by inlining the core rules from AGENTS.md (session ritual, code rules, forbidden section)
- **Cline:** Create `.clinerules` referencing AGENTS.md as the rulebook to read at session start

If no tool indicators are found, ask the user: "Which AI coding tool(s) are you using? (e.g., Claude Code, Cursor, Windsurf, GitHub Copilot, Cline)"

Note: The wrapper is for the tool agent (Claude Code, Copilot, Cursor, etc.), not the underlying model. A user running Copilot with Sonnet selected still needs a Copilot wrapper, not a CLAUDE.md — the tool reads its own config file regardless of which model it's using.

Format this as a clean markdown file I can save as AGENTS.md in the project root.

> **Tool-specific wrappers:** The prompt above instructs the AI to scan for tool indicators and create the appropriate wrapper automatically. If you're using multiple tools (e.g., Claude Code and Cursor on the same repo), it will create a wrapper for each. If you add a new tool later, re-run this step — or manually create the wrapper using the table below as reference.

**What good output looks like:** A rulebook that feels incredibly familiar. It should document your exact naming conventions and architectural choices.

**Watch out for:** The AI enforcing framework defaults over your custom patterns. Ensure the agent successfully extracted your custom patterns instead of falling back to its training data defaults.

---

### 0.4 — The Delta Audit (Alignment Check)

**When to use it:** As the final step of onboarding an existing app. You have your documentation, but you need to know where your actual code is currently violating the rules you just established. This gives you your first clean-up sprint.

**Model:** Deep/extended reasoning tier

**The prompt:**
I have just generated foundational documentation for this existing
codebase (docs/SPEC.md, AGENTS.md, docs/DECISIONS.md).

I need you to run a Delta Audit: compare the actual codebase against
these newly generated standards to find where the code contradicts
itself or its implicit rules.

Scan the codebase and flag:

1. Rogue Patterns: Where does the code violate the rules established
   in AGENTS.md? (e.g., 90% of routes use the standard error handler,
   but these 3 routes use raw try/catch blocks).
2. Spec Drift: Are there endpoints, models, or components in the code
   that serve no discernible purpose and aren't in the docs/SPEC.md?
   (Dead code).
3. Security/Architecture Gaps: Based on the stack defined in docs/DECISIONS.md,
   are there any glaring omissions? (e.g., "You chose JWT for auth,
   but there is no token expiration or refresh logic implemented").

For each finding, tell me:

- Location (File/Line)
- The discrepancy
- Severity (Critical/High/Medium/Low)
- The smallest required change to bring it into alignment

Do not propose rewrites. Provide an actionable list of alignment fixes.

**What good output looks like:** A highly targeted list of inconsistencies. A great Delta Audit finds the single controller that an old developer wrote before the team settled on a unified pattern, or identifies dead database tables that aren't hooked up to any models.

**Watch out for:** Deep/extended models getting too pedantic about minor stylistic differences. If it starts flagging whitespace or minor syntax variations that don't impact architecture or maintainability, tell it to strictly focus on structural and behavioral drift.

---

## Section 1: Project Setup (New Apps)

---

### 1.1 — Spec File Generator

**When to use it:** Before writing a single line of code. This is a conversation, not a one-shot prompt. You provide your idea, your background, and your stack preference — then the agent interviews you before writing anything. Answer the questions thoroughly. The spec quality is directly proportional to how much you give the agent during the interview. Expect this to take 2-3 back-and-forth exchanges before you get the final spec.

**Model:** Deep/extended reasoning tier — you want deep thinking here. A weak spec causes every downstream problem.

**The prompt:**
I want to build [describe your app idea in 2-3 sentences].

My background: [describe your technical skill level — e.g. "I'm a frontend
dev comfortable with React but new to backend" or "I'm a non-technical
founder who can follow tutorials but can't debug from scratch"]

My stack: [paste your stack, or write "help me choose" and I'll suggest
strong defaults for AI-assisted development]

Your job is to help me turn this into a complete SPEC.md file, but don't
write it yet.

First, interview me. Ask 3-5 questions focused on areas where two
reasonable developers might interpret my description differently. Ask about
things like: who the users are, what the core workflows look like, what
should be explicitly out of scope for v1, and anything else that's unclear
or ambiguous.

After I've answered your questions, write the SPEC.md with this structure:

1. What this app is — a 2-3 sentence plain English description
2. Who the users are and how they authenticate
3. Each major feature, broken into bullet points describing exact behavior
4. The 2-3 main user journeys — the exact sequence of screens and actions
   a user walks through to accomplish a core task. For example:
   "New user lands on homepage → clicks Sign Up → enters email and password
   → is redirected to an empty dashboard → clicks 'Create Project' → fills
   in project name and description → sees the new project appear in their
   project list"
5. The data models (what entities exist and what fields they have)
6. API endpoints needed (method, path, what it does)
7. A clear "What this is NOT" section — at least 5 things explicitly out
   of scope for v1
8. Tech stack decisions with brief reasoning for each choice

Format this as a clean markdown file I can save as docs/SPEC.md.

**What good output looks like:**

First response: 3-5 specific questions that show the agent understood your idea. If the questions are generic ("what features do you want?" or "who is your target audience?"), your initial description wasn't specific enough — add more detail and resend.

After you answer: A complete SPEC.md that would let a developer build the app without asking you a single question. Each feature is specific enough to be testable. The user journeys read like someone actually clicking through the app. The "What this is NOT" section has at least 5 items and should include things you might be tempted to add later. If any section feels vague or generic, push back on that section specifically — don't accept the whole spec just because most of it is good.

**Watch out for:** Deep/extended models sometimes over-engineer v1. If the spec includes things like real-time websockets, advanced analytics, or mobile apps and you didn't ask for them, push back and trim. Also watch for the interview questions being too shallow — if the agent only asks 2 questions or they're all yes/no, tell it to go deeper.

---

### 1.2 — PROGRESS.md Initializer

**When to use it:** After your SPEC.md is written using Prompt 1.1. This generates your build plan before any code is written.

**Model:** Fast/code-focused tier

**The prompt:**
Using the SPEC.md in this project (see docs/SPEC.md), create an initial
PROGRESS.md file that serves as both a checklist and a build plan.

Structure it with these sections:

```markdown
## Implemented
(empty to start)

## In Progress
(empty to start)

## Not Started
Every feature from the spec, broken into items that each represent
roughly one working session of effort. A working session is approximately
1-2 hours of a developer working with an AI coding assistant — not a
human working alone. Tasks that would take a solo developer a full day
might be a single session with AI assistance. Use your judgment to size
items accordingly.

Group and order items so that:
- Dependencies are built before the things that need them
- Each session ends with something the user can see or manually test.
  For example: after building auth, the user should be able to register
  and log in. After building the project creation API, the user should
  be able to create a project and see it returned from the endpoint.
- The order maps back to the user journeys in the spec — prioritize
  completing the first full user journey end-to-end before starting
  features that belong to secondary flows

For each item include:
- A checkbox
- A short description of what gets built
- An estimated session count (1 session, 2 sessions, etc.)
- A note if this item blocks other items
- A note indicating what the user will be able to verify once it's done

## Blocked / Open Questions
(empty to start — this section is for items that come up during
development that can't be resolved without a decision, an external
dependency, or more information. During any working session, if something
can't move forward, it goes here instead of getting lost in conversation.)
```

Format this as a clean markdown file I can save as docs/PROGRESS.md.

**What good output looks like:** A build plan where you can look at the first 3-5 items and picture your first few sessions clearly. Each item has a testable outcome — you know what "done" looks like before you start. The order should let you complete at least one full user journey within the first few sessions so you have something real to click through early. If the total session count exceeds 20-25 sessions, your spec is probably too ambitious for v1 — consider trimming the spec before starting.

**Watch out for:** Session estimates that feel like human-only estimates. Push back and ask the agent to recalibrate with the assumption that an AI assistant is writing most of the code and the developer is reviewing, testing, and steering.

---

### 1.3 — DECISIONS.md Initializer

**When to use it:** After your SPEC.md is written using Prompt 1.1. This documents the founding architectural decisions before any code is written. New decisions will be appended during development as part of the end-of-session process in Prompt 7.1.

**Model:** Fast/code-focused tier

**The prompt:**
Using the SPEC.md in this project (see docs/SPEC.md), create an initial
docs/DECISIONS.md file that documents the key architectural decisions already
made for this project.

For each decision, use this format:

```markdown
## [Decision title]
- **Date:** [today's date]
- **Status:** Active
- **Decision:** What was decided
- **Why:** The honest reasoning — if this choice was driven by the
  developer's skill level, time constraints, or simplicity needs, say so.
  Not every decision is a pure technical evaluation and that's fine.
- **Alternatives considered:** If this decision was actively evaluated
  against other options, describe what was considered and why it wasn't
  chosen. If the developer brought this choice in as a given (e.g. "I
  already know React"), say "Brought in as a prior choice" rather than
  fabricating a comparison that never happened.
- **Consequences:** What this decision means for the project going forward
- **Revisit if:** The specific conditions under which this decision should
  be reconsidered. For example: "Revisit if we need real-time updates" or
  "Revisit if auth requirements expand beyond email/password." If there's
  no realistic scenario where this changes for v1, write "Stable for v1."
```

Cover at minimum:

- Framework choices (backend and frontend)
- Database and ORM choice
- Auth strategy
- Deployment target

At the end of the file, include this section:

```markdown
## Decisions Made During Development
_This section is for architectural decisions made after the project has
started. Each decision should follow the same format above. New entries
are appended here during the end-of-session process (see Prompt 7.1)
whenever a meaningful technical choice is made during a working session._

(empty to start)
```

**Meaningful technical choice (addendum):** Auth/storage/DB/schema changes, framework/library adoption or removal, security posture changes (tokens, hashing, session strategy), deployment/infra shifts, or pattern/architecture decisions that affect multiple features. Routine bugfixes or style tweaks do not require an entry.

Format this as a clean markdown file I can save as docs/DECISIONS.md.

**What good output looks like:** A document where the "Why" sections are specific and honest. The "Alternatives considered" section should never read like the AI is inventing a debate that didn't happen. And every decision should have a "Revisit if" that gives you a concrete trigger, not a vague "revisit if requirements change."

**Watch out for:** Two things. First, the agent writing "Revisit if requirements change" on every decision — that's meaningless. Push back and ask for specific triggers tied to your app. Second, the "Alternatives considered" section sounding like a blog post comparing frameworks. Honesty in this file saves you from second-guessing yourself later.

---

### 1.4 — AGENTS.md Scaffolding

**When to use it:** After your SPEC.md (Prompt 1.1), PROGRESS.md (Prompt 1.2), and DECISIONS.md (Prompt 1.3) are all created. This creates the instruction file your AI agent reads at the start of every session. It references all three files, so they need to exist first. Your SPEC.md must include your tech stack and developer background — this prompt depends on both.

**Model:** Fast/code-focused tier

**The prompt:**
Using the SPEC.md in this project (see docs/SPEC.md), create an AGENTS.md
file for this project. This file contains the rules and context your AI
agent reads at the start of every session. Include:

1. A project context section that references the following files (use
   your tool's import/include syntax if available, otherwise list them
   as files to read at session start):
   - docs/SPEC.md
   - docs/PROGRESS.md
   - docs/DECISIONS.md

2. A session start ritual — steps the agent should follow before writing
   any code:
   - Run git log --oneline -10 to understand recent work
   - Run git status to check current state
   - Review PROGRESS.md to confirm what's built and what's next
   - Ask the user what we're working on before touching any files

3. The tech stack clearly listed (pulled from the spec) with a note to
   never deviate without explicit approval

4. Code rules — generate these based on the tech stack in the spec.
   Follow established best practices and conventions for each language
   and framework in the stack. Include at minimum:
   - Type safety rules appropriate to the language
   - Error handling conventions for the framework
   - Testing expectations (every new function needs a corresponding test)
   - Security basics (secrets never hardcoded, sensitive data never logged,
     environment variables validated at startup)

5. Testing configuration: Based on the tech stack in the spec, document
   the test framework appropriate to the stack, the expected commands to
   run the full suite and a single test file, and any environment setup
   that will be needed (test database, env vars, seed data). Note that
   this section should be updated once the test infrastructure is
   actually in place — at AGENTS.md creation time, tests don't exist yet.

6. A forbidden section:
   - Before installing any new package or dependency, explain why it's
     needed, what problem it solves, and whether the same thing could be
     done with what's already in the stack. Let the user approve before
     installing.
   - Do not change the database schema without confirming with the user
   - Do not rewrite working code to use a different pattern unprompted
   - Do not use frameworks or libraries outside the tech stack defined
     in the spec
   - CI/CD & secrets: Never echo, print, or persist secrets in logs/artifacts. Use the platform's secret store; do not hardcode or inline secrets in build configs. Fail the pipeline if required secrets are missing. Validate env vars at startup/build time.

7. Developer experience — read the developer background in the spec and
   calibrate accordingly. If the developer is early-career or learning,
   explain what you're doing and why as you go. If the developer is
   experienced, be concise and just execute.

8. Disagreements — if the user asks you to do something that would
   violate a rule in this file, introduce a security risk, create
   technical debt, or go against best practices for the stack, say so
   before proceeding. Explain why you disagree and what you'd recommend
   instead. Then let the user decide. Do not silently comply with
   something you think is a mistake.

9. End-of-session — when the user says "we're done", follow this
   process:
   - Update PROGRESS.md with what was completed, what's in progress,
     and any new items discovered
   - Suggest a git commit message with type prefix and bullet points
   - Update DECISIONS.md if any architectural decisions were made
   - Recommend what to tackle first in the next session

After generating AGENTS.md, detect which AI coding tools are configured
for this project by scanning the project root for these indicators:

- `.claude/` directory or existing `CLAUDE.md` → Claude Code
- `.cursor/` directory → Cursor
- `.windsurfrules` file → Windsurf
- `.github/copilot-instructions.md` or `.github/workflows/` directory → GitHub Copilot
- `.clinerules` file → Cline

For each detected tool, create the appropriate wrapper file:

- **Claude Code:** Create `CLAUDE.md` in the project root containing only `@agents.md`
- **Cursor:** Create `.cursor/rules/agents.mdc` with frontmatter `alwaysApply: true` and a note to read AGENTS.md at session start
- **Windsurf:** Create `.windsurfrules` containing `@agents.md`
- **GitHub Copilot:** Copilot cannot import files — create `.github/copilot-instructions.md` by inlining the core rules from AGENTS.md (session ritual, code rules, forbidden section)
- **Cline:** Create `.clinerules` referencing AGENTS.md as the rulebook to read at session start

If no tool indicators are found, ask the user: "Which AI coding tool(s) are you using? (e.g., Claude Code, Cursor, Windsurf, GitHub Copilot, Cline)"

Note: The wrapper is for the tool agent (Claude Code, Copilot, Cursor, etc.), not the underlying model. A user running Copilot with Sonnet selected still needs a Copilot wrapper, not a CLAUDE.md — the tool reads its own config file regardless of which model it's using.

Format this as a clean markdown file I can save as AGENTS.md in the
project root.

**Tool-specific wrappers:** The prompt above instructs the AI to scan for tool indicators and create the appropriate wrapper automatically. If you're using multiple tools (e.g., Claude Code and Cursor on the same repo), it will create a wrapper for each. If you add a new tool later, re-run this step — or manually create the wrapper using the table below as reference.

**What good output looks like:** An AGENTS.md that reads like a confident brief tailored to your specific stack. The code rules should reference your actual frameworks and languages, not generic boilerplate. The developer experience section should reflect your skill level accurately — if you said you're a beginner in the spec and the AGENTS.md doesn't mention explanations, push back.

**Watch out for:** The forbidden section being too generic. It should include at least one rule specific to your stack. If everything in the forbidden section could apply to any project, ask the agent to add stack-specific guardrails.

---

### 1.5 — Tech Stack Decision Helper

**When to use it:** When you selected "help me choose" in Prompt 1.1, or before committing to a stack when you're unsure what fits your specific app and skill level. Run this before finalizing your SPEC.md — the stack decision feeds back into the spec.

**Model:** Deep/extended reasoning tier — this is a decision you'll live with for the whole project.

**The prompt:**
I need help choosing a tech stack for my project.

Here is my app description (or see docs/SPEC.md if the spec is already
started):
[describe your app in 2-3 sentences, or reference your spec file]

My background: [describe your technical skill level]

What I already know and have used before:

- Languages: [e.g. JavaScript, Python, C#, none]
- Frameworks: [e.g. React, Django, .NET, none]
- Databases: [e.g. PostgreSQL, MongoDB, SQL Server, none]
- Deployment: [e.g. Vercel, Azure, never deployed anything]

My constraints:

- [e.g. "I want to stay in one language for frontend and backend" or
  "I need this deployed for free or very cheap" or "I want managed auth
  so I don't have to build login from scratch"]
- I'll be using AI assistance to write most of the code, so I
  need a stack where AI tools can be most effective
- [add any other constraints]

For each layer (backend framework, frontend framework, database, ORM or
database client, auth approach, deployment), give me:

1. Your recommendation — and why it fits this specific app and my
   skill level. Prioritize tools I can debug myself when AI gets stuck
   over tools that are technically optimal but unfamiliar to me.
2. Why AI assistance will be effective with this choice — not whether
   the AI "has training data on it" but whether the tool has mature
   documentation, a large ecosystem of examples, and well-established
   patterns that AI tools can draw from.
3. Two alternatives, each with a clear tradeoff:
   - One that's simpler but with limitations I'd hit as the app grows
   - One that's more powerful but with a steeper learning curve or
     more complexity
4. Any known gotchas I should be aware of — especially things that are
   fine in tutorials but cause problems in real projects.

After presenting your recommendations, give me a one-paragraph summary
of the full stack as a coherent whole — explain how the pieces fit
together, not just why each piece was chosen individually.

Do not recommend anything obscure, bleeding-edge, or with sparse
documentation. If my existing experience points clearly toward a
particular tool and that tool is a reasonable choice, prefer it over
something marginally better that I've never used.

**What good output looks like:** A recommendation where every choice references your specific app and your specific experience. The two alternatives per layer should feel like real choices with real tradeoffs, not a winner and two strawmen. The summary paragraph at the end should make you feel like the stack is a coherent system, not six independent decisions.

**Watch out for:** Three things. First, the agent recommending the "best" tool over the tool you already know. In AI-assisted development, your ability to debug when AI gets stuck matters more than technical optimality. Second, vague AI-effectiveness reasoning like "it's popular" or "there's a lot of content." Push for specifics. Third, Deep/extended models sometimes recommend newer tools they find interesting. Ask for a more established alternative if needed.

---

## Section 2: Building Features

---

### 2.1 — Feature Implementation

**When to use it:** Every time you want to build a new feature. This is your workhorse prompt. Use plan mode first to have the agent propose its approach without touching files, then review and approve before implementation begins.

**Model:** Fast/code-focused tier

**The prompt:**
I need to implement [feature name].

Before writing any code, orient yourself:

- Read the spec (see docs/SPEC.md) and find the section describing
  this feature
- Check PROGRESS.md (see docs/PROGRESS.md) to confirm that any
  dependencies this feature requires are marked as Implemented. If
  anything this feature depends on isn't built yet, stop and tell me
  before going further.
- Scan the existing codebase to understand what files exist, what
  patterns are established, and what you can reuse
- Review AGENTS.md for any rules that are especially relevant to this
  feature — if this touches auth, payments, or sensitive data, call
  out the specific rules that apply

Then give me your plan:

- What files you'll create or modify, and why
- How this connects to existing code and patterns in the codebase
- Anything that concerns you or feels ambiguous about this feature
- If my skill level (as described in the spec) means I'd benefit from
  understanding your reasoning, explain the key decisions in the plan.
  If I'm experienced, keep the plan concise.

After I approve the plan and switch to act mode:

1. Implement the feature following the rules in AGENTS.md
2. Before running tests, ensure required setup (migrations, seed data, env vars) is applied. If setup is missing, stop and ask whether to run migrations/seed in this session; don't mark tests as failed due to missing setup.
3. Run existing tests to confirm nothing is broken by the changes
4. If no tests exist yet, explicitly call this out and propose a minimal smoke/regression test set before or alongside implementation. Do not skip the "run tests" step; instead, note absence and recommend creating baseline tests.
5. Write new tests for what you just built — test behavior, not
   implementation
6. Tell me what to manually test to confirm it works, and note which
   user journey from the spec this feature advances

Do not modify any files outside the scope of this feature. If you discover something outside this feature's scope that needs fixing, do not fix it — add it to the 'In Progress' or 'Not Started' section of PROGRESS.md instead.

**What good output looks like:**

In plan mode: A clear, scoped proposal that shows the agent understands the codebase context. It should reference specific existing files it intends to work with, not just describe things generically. You should be able to read the plan and know exactly what's going to change before anything changes.

In act mode: Clean implementation that follows the plan. Existing tests still pass. New tests cover the happy path, invalid inputs, and edge cases. The manual testing instructions should let you verify the feature works in under five minutes.

**Watch out for:** Three things. First, the agent skipping the orient step and jumping straight to a plan based on the feature name alone. Second, the plan being too ambitious — if it proposes modifying more than 5-6 files for a single feature, the feature might need to be broken into smaller pieces. Third, after implementation, the agent skipping the step of running existing tests.

---

### 2.2 — Database Schema Design

**When to use it:** When designing your database schema before writing any API routes or business logic. This is a foundational decision — get it right early. Use plan mode to review the schema before the agent writes any migration files.

**Model:** Deep/extended reasoning tier — schema decisions are hard to undo.

**The prompt:**
I need to design the database schema for this project.

Before designing anything, orient yourself:

- Read the spec (see docs/SPEC.md) for the data models, features, and
  tech stack — the database engine and ORM are defined there
- Read the user journeys in the spec carefully. The schema needs to
  support every step of every journey efficiently
- Check PROGRESS.md (see docs/PROGRESS.md) to understand build order
  — which models will be used first and which won't be touched for
  several sessions

Design the schema following the conventions and best practices for the
ORM and database defined in the spec. Include:

- Appropriate ID strategy for the ORM (cuid, uuid, auto-increment —
  whatever is conventional)
- Timestamp fields handled the way the ORM recommends
- Explicit relation names on all foreign keys
- Database-level constraints where appropriate (unique, not null)
- Indexes on any field that will be frequently queried, filtered, or
  used in joins

For each model, explain:

- Why it's structured this way
- Any non-obvious decisions
- Which user journeys depend on this model

After designing the schema, validate it yourself:

- Walk through each user journey from the spec step by step. For each
  step, describe the query that would be needed and confirm the schema
  supports it efficiently. If any step requires an awkward join, a
  missing field, or a query that would be slow without an index, fix
  the schema before presenting it.
- Flag any models that exist only to support features in later sessions
  — mark these as "provisional" and note that they may change as those
  features are built

Do not include any models for features marked as out of scope in the
spec.

**What good output looks like:**

In plan mode: A schema where every model traces back to a feature or user journey in the spec. The validation walkthrough should show you the actual queries the agent expects to run against each model. Models marked as provisional should be clearly separated from the ones you'll use immediately.

In act mode: The schema written as a migration file in whatever ORM your project uses, ready to run. It should follow that tool's conventions — not generic SQL patterns forced into an ORM.

**Watch out for:** Four things. First, the agent adding models for features that are out of scope. Second, missing indexes on foreign keys and fields used in filtering. Third, the schema being designed in isolation from queries. Fourth, the agent defaulting to patterns from a different ORM than the one in your spec.

---

### 2.3 — API Endpoint Implementation

**When to use it:** When building a specific API route or group of related routes. This builds on the workflow established in Prompt 2.1 with API-specific guidance. Use plan mode to review the route design before the agent writes any code.

**Model:** Fast/code-focused tier

**The prompt:**
I need to implement the following API endpoint(s): [e.g. POST /api/projects,
GET /api/projects, GET /api/projects/:id]

Before writing any code, orient yourself:

- Read the spec (see docs/SPEC.md) to understand what these endpoints
  need to do and which user journeys they support
- Check PROGRESS.md (see docs/PROGRESS.md) to confirm that
  dependencies for these routes are built — especially the database
  schema and any auth middleware
- If the database schema was designed with query validation in Prompt
  2.2, reference those expected queries. The routes should execute the
  queries the schema was designed to support.
- Scan the existing codebase for established patterns. If routes already
  exist in this project, match their structure exactly. Consistency
  matters more than any single route being "better."
- Review AGENTS.md for rules relevant to API development

Then give me your plan:

- What routes you'll create, what middleware each one uses, and what
  the request/response shapes look like
- If this is the first set of routes in the project, propose the
  conventions that all future routes will follow: error response shape,
  validation library, file organization, naming patterns. These become
  the standard — so I want to approve them explicitly.
- How you'll handle authorization — not just "is the user logged in"
  but "can THIS user access THIS resource." For any route that returns
  or modifies a specific record, explain how you'll verify the
  requesting user has permission to touch that record.

After I approve the plan and switch to act mode:

1. Implement the endpoints following the conventions in AGENTS.md and
   any patterns already established in the codebase
2. Use the validation and error handling approach appropriate to the
   framework in the spec — do not introduce a different library or
   pattern than what's already in use
3. Input validation must run before any database call
4. No raw queries unless the ORM genuinely can't express what's needed
   — and if so, flag it and explain why
5. Before running tests, ensure required setup (migrations, seed data, env vars) is applied. If setup is missing, stop and ask whether to run migrations/seed in this session; don't mark tests as failed due to missing setup.
6. Run existing tests to confirm nothing is broken
7. If no tests exist yet, explicitly call this out and propose a minimal smoke/regression test set before or alongside implementation. Do not skip the "run tests" step; instead, note absence and recommend creating baseline tests.
8. Write tests covering at minimum:
   - Happy path
   - Missing or invalid input fields
   - Unauthorized access (missing or invalid credentials)
   - Forbidden access (valid user, wrong resource — user A trying to
     access user B's data)
   - Resource not found
9. Tell me what to manually test and which user journey these
   endpoints advance

**What good output looks like:**

In plan mode: A route-by-route breakdown showing the request shape, response shape, middleware chain, and authorization logic for each endpoint. The authorization plan should specifically address object-level access — "we'll check that the project's ownerId matches the authenticated user's id".

In act mode: Routes that are indistinguishable in style from any existing routes in the codebase. Error responses are identical in shape across every endpoint. Tests include the forbidden access case.

**Watch out for:** Five things. First, the agent introducing a different validation or error handling library than what's already in the project. Second, missing object-level authorization. Third, inconsistency with existing routes. Fourth, the agent wrapping every line in try/catch rather than using a centralized error handling middleware. Fifth, tests that only check status codes without checking response bodies.

---

### 2.4 — UI Component Implementation

**When to use it:** When building a UI component or a full page. This builds on the workflow established in Prompt 2.1 with frontend-specific guidance. Use plan mode to review the component design before the agent writes any code.

**Model:** Fast/code-focused tier

**The prompt:**
I need to build a [component or page name].

What it does: [describe the UI and behavior]

Before writing any code, orient yourself:

- Read the spec (see docs/SPEC.md) to understand where this component
  fits. Find which user journey it belongs to — what screen comes before
  it, what user action leads here, and where the user goes next.
- Check PROGRESS.md (see docs/PROGRESS.md) to confirm that any API
  endpoints or data this component depends on are already built
- Scan the existing codebase for established frontend patterns. If
  components already exist, match their structure exactly — same file
  organization, same naming conventions, same styling approach, same
  state management patterns. Do not introduce a new pattern alongside
  an existing one.
- Review AGENTS.md for any frontend-specific rules

Then give me your plan:

- What files you'll create or modify
- Where this component sits in the user journey — what data it expects
  to already have, what it needs to fetch, and what action or navigation
  it triggers when the user completes the main interaction
- If this is the first component in the project, propose the conventions
  that all future components will follow: file structure, styling
  approach, state management patterns, how data fetching works. These
  become the standard — I want to approve them explicitly.
- How you'll handle all UI states:
  - Loading (data is being fetched)
  - Error (something went wrong)
  - Empty (no data exists yet — e.g. "You don't have any projects yet")
  - Success (the happy path)

Design considerations — address each of these in your plan:

- Does this need to match an existing page layout or component style
  in the project? If so, which one?
- Is this responsive? How will it adapt to mobile vs. desktop?
- Accessibility: semantic HTML elements, keyboard navigation for
  interactive elements, ARIA labels where needed, sufficient color
  contrast. Follow accessibility best practices for the framework in
  the spec.
- Ask whether to include localization/RTL and color-contrast validation. If the developer is less experienced, briefly explain why (accessibility, international users) and propose minimal opt-in steps. At minimum, ensure keyboard nav + ARIA; optionally add contrast/RTL if approved.

After I approve the plan and switch to act mode:

1. Implement the component following the conventions in AGENTS.md and
   any patterns already established in the codebase
2. Use the framework, styling approach, and state management appropriate
   to the tech stack in the spec — do not use patterns or terminology
   from other frameworks
3. Handle all four UI states (loading, error, empty, success)
4. Before running tests, ensure required setup (migrations, seed data, env vars) is applied if relevant; don't mark tests as failed due to missing setup.
5. Run existing tests to confirm nothing is broken
6. If no tests exist yet, explicitly call this out and propose a minimal smoke/regression test set before or alongside implementation. Do not skip the "run tests" step; instead, note absence and recommend creating baseline tests.
7. Write tests covering:
   - The component renders correctly in each state (loading, error,
     empty, success)
   - The main user interaction works (e.g. form submission, button
     click, navigation)
   - Accessibility basics — interactive elements are keyboard
     accessible, critical elements have appropriate labels
8. Tell me what to manually test and which user journey step this
   component completes

**What good output looks like:**

In plan mode: A proposal that clearly places the component in the context of a user journey, not just as an isolated piece of UI. The state handling plan should cover all four states with specific descriptions of what each looks like. The accessibility section should name specific elements.

In act mode: A component that is visually and structurally consistent with any existing components in the project. All four states are handled and look intentional, not like afterthoughts. The empty state in particular should feel like a designed part of the app, not a blank page. Tests verify real user interactions, not just that the component mounts without crashing.

**Watch out for:** Five things. First, the agent using patterns from a different framework than the one in your spec. Second, the empty state being ignored or treated as just a paragraph of text. Third, accessibility being treated as optional or skipped entirely. Fourth, styling that doesn't match existing components. Fifth, components that fetch data in a way that doesn't match the framework's conventions.

---

### 2.5 — Authentication Implementation

**When to use it:** When building the auth system. This should be one of the first features you build, since almost everything else depends on it. Use plan mode to review the entire auth design before the agent writes any code. Auth is not the place to skip the plan step.

**Model:** Deep/extended reasoning tier — auth is not the place to cut corners.

**The prompt:**
I need to implement authentication for this project.

Before writing any code, orient yourself:

- Read the spec (see docs/SPEC.md) for the auth strategy defined in
  the tech stack. Understand whether this project uses a managed auth
  service (like Supabase Auth, Clerk, NextAuth, Lucia) or requires a
  custom implementation (like JWT with bcrypt).
- Read the developer background in the spec. If the developer is
  early-career and the spec calls for custom auth, flag this as a
  risk in your plan and recommend whether a managed service would be
  more appropriate. Explain the tradeoff honestly — managed auth is
  less flexible but dramatically less likely to have security holes.
- Check the database schema (from Prompt 2.2) to confirm the user
  model has everything auth needs. If fields are missing, list what
  needs to be added before proceeding.
- Check PROGRESS.md (see docs/PROGRESS.md) to confirm the database
  schema is built and migrated
- Scan the existing codebase for any patterns already established —
  error response shapes, validation approach, file organization. Auth
  routes should be consistent with any existing routes from 2.3.
- Review AGENTS.md for security-related rules

Then give me your plan. Cover all of these:

Auth approach:

- If using a managed service: what needs to be configured, what
  environment variables are required, how the service integrates with
  the existing codebase and database
- If building custom auth: what routes, what token strategy (JWT,
  session, etc.), what hashing algorithm and cost factor, token
  expiration policy, and whether you recommend refresh tokens or
  single long-lived tokens. Explain the reasoning for each choice —
  these will be recorded in DECISIONS.md.

Security plan — address each of these explicitly:

- How passwords are hashed and at what cost factor
- How tokens/sessions are stored (cookies vs. headers, secure flags,
  httpOnly, sameSite)
- How error messages are kept consistent to avoid leaking whether an
  email exists
- Rate limiting strategy on auth routes
- CSRF protection approach if using cookies
- Account lockout or throttling after repeated failed attempts
- What gets logged on failed attempts (and confirmation that
  credentials are never logged)
- How the password field is excluded from every response and query
  that returns user data
- CI/CD: Ensure secrets are only injected via the platform's secret manager; redact secrets in logs; disable artifact bundling of `.env`; forbid committing `.env` or real-looking values.

Middleware design:

- How the auth middleware interface works — what it adds to the
  request, what types it exports, how other route files import and
  use it
- Confirm that this interface is compatible with the route patterns
  established in 2.3 (or will establish the pattern if auth is built
  first)

After I approve the plan and switch to act mode:

1. Implement auth following the approved plan and AGENTS.md rules
2. Every security decision you make should be explained in a comment
   next to the code — not what the code does, but why the security
   choice was made
3. Before running tests, ensure required setup (migrations, seed data, env vars) is applied. If setup is missing, stop and ask whether to run migrations/seed in this session; don't mark tests as failed due to missing setup.
4. Run existing tests to confirm nothing is broken
5. If no tests exist yet, explicitly call this out and propose a minimal smoke/regression test set before or alongside implementation. Do not skip the "run tests" step; instead, note absence and recommend creating baseline tests.
6. Write tests covering at minimum:
   - Successful registration
   - Duplicate email registration (should fail gracefully)
   - Successful login
   - Wrong password (error message must be identical to wrong email)
   - Wrong email (error message must be identical to wrong password)
   - Expired token/session rejection
   - Missing token/session rejection
   - Malformed token/session rejection
   - Rate limiting triggers after repeated failures
   - Password field is never present in any response
7. Tell me what to manually test — I should be able to register, log
   in, access a protected resource, and confirm that accessing it
   without auth fails

Explain any security decisions you make that aren't covered in the
plan — if something came up during implementation that required a
judgment call, document it.

**What good output looks like:**

In plan mode: The first thing you should see is the agent confirming whether this is a managed service integration or a custom implementation, based on what's in the spec. Every item in the security plan should have a specific answer, not "will be handled."

In act mode: Auth code where the security comments explain *why*, not *what*. "// bcrypt cost 12: balances security with response time under 500ms" is useful. The tests should include both wrong-password and wrong-email cases with an assertion that the error messages are identical.

**Watch out for:** Six things. First, the agent building custom JWT auth when the spec says to use a managed service. Second, error messages that differ between "wrong password" and "email not found". Third, the password field appearing in any API response. Fourth, tokens stored in localStorage without any mention of XSS risk, or cookies without httpOnly and secure flags. Fifth, missing rate limiting. Sixth, if using custom auth, the absence of a refresh token strategy.

---

### 2.6 — "Explain Before You Touch"

**When to use it:** When you're about to modify existing code that is complex, fragile, or that you've lost context on. This is not for normal feature development — Prompt 2.1 already has an orient step for that. Use this when your gut says "this feels risky." Use plan mode — this prompt's entire purpose is "think before you touch."

**Model:** Fast/code-focused tier

**The prompt:**
I need to make a change to existing code, but before you touch anything
I need you to prove you understand what you're working with.

The change I want to make: [describe the change]

Before proposing any modifications:

1. Read the file (see @[filepath]) and explain in plain English what it
   does — not just technically, but in the context of this app. What
   feature does it support? Which user journeys from the spec depend on
   it? What would break from the user's perspective if this file stopped
   working?

2. Trace the dependencies in both directions by scanning the codebase:
   - What does this file import and rely on?
   - What other files import and depend on this one?
   - If you change a function signature or remove an export, what
     specifically breaks?

3. Check whether tests exist for this file:
   - If tests exist, summarize what behavior they verify — these define
     the contract you must not break
   - If no tests exist, say so explicitly. That means this change is
     higher risk because there's no automated safety net.

4. Identify the fragile parts — what in this file is most likely to
   break if modified? What assumptions is the code making that aren't
   obvious from reading it?

5. Now give me your plan for the change:
   - What specifically you'll modify
   - What you will NOT modify and why
   - How you'll verify the change doesn't break anything that depends
     on this file
   - If no tests exist, whether you recommend writing tests for the
     existing behavior before making the change

Calibrate your explanation to my skill level as described in the spec.
If I'm still learning, explain the concepts and why things are
structured this way. If I'm experienced, keep it focused on the
dependency map and risk assessment.

Do not make any changes until I've reviewed your analysis and approved
the plan.

**What good output looks like:**

In plan mode: An explanation that makes you feel like the agent genuinely understands the file, not just that it read it. The dependency trace should name specific files and functions. The fragility assessment should identify concrete risks. The plan itself should be conservative.

**Watch out for:** Four things. First, the agent rushing through the explanation to get to the plan. Second, the agent saying "this file is imported by several other files" without naming them. Third, the agent proposing to refactor the file while making your change. Fourth, if no tests exist and the agent doesn't mention that, it's a red flag.

---

## Section 3: Code Review & The AI Review Loop

---

### 3.1 — Standard Code Review

**When to use it:** After completing a feature, before committing. Run this in a fresh conversation — not the same session that wrote the code. The model that wrote the code is biased toward its own output and will overlook the same edge cases it missed when generating.

**Model:** Deep/extended reasoning tier in a **fresh conversation.**

**The prompt:**
You are a senior code reviewer. The code you're about to review was
written by an AI assistant in a previous session. Your job is to find
issues the original author might have missed. You have no attachment to
this code — be ruthless.

Before reviewing, read:

- The project rules (see AGENTS.md)
- The spec (see docs/SPEC.md) — specifically the section describing
  the feature this code implements
- The code to review: (see @[list the files or directory to review])

Review in this priority order:

1. Security vulnerabilities:
   - Can any input be used to exploit the system?
   - Can user A access, modify, or delete user B's data by manipulating
     IDs, parameters, or request bodies? Check every query that uses a
     user-supplied ID.
   - Are auth and authorization checks present on every route that
     needs them?
   - Are secrets, tokens, or passwords exposed anywhere — in responses,
     logs, or error messages?

2. Spec compliance:
   - Does the code actually implement what the spec describes?
   - Are there edge cases described in the spec that the code doesn't
     handle?
   - Does the code do anything the spec doesn't ask for?

3. AGENTS.md rule violations:
   - Check every rule in AGENTS.md against the code
   - Flag any violation with the specific rule being broken

4. Error handling:
   - Unhandled promise rejections, missing try/catch, unvalidated inputs
   - Error responses that expose stack traces or internal details
   - Error shapes that are inconsistent with the rest of the codebase

5. Language and framework issues — based on the tech stack in the spec:
   - Type safety problems appropriate to the language (e.g. `any` types
     in TypeScript, missing null checks)
   - Framework anti-patterns (e.g. data fetching that bypasses framework
     conventions)
   - Anything that works now but will cause problems as the codebase
     grows

6. Logic errors:
   - Code that will break on null, undefined, empty arrays, or
     boundary conditions
   - Off-by-one errors, race conditions, or assumptions about data
     that aren't validated

7. Pattern consistency:
   - Does this match the patterns established in the rest of the
     codebase?
   - If it deviates, is the deviation justified or accidental?

Rules for your review:

- Do not compliment the code
- Be specific — "line 34 in auth.ts will throw if user is undefined
  because the query doesn't check for null before accessing .id" not
  "needs better error handling"
- For each issue, rate severity: Critical / High / Medium / Low
- If you find zero Critical or High issues, say so explicitly — that's
  meaningful signal
- At the end, give a clear verdict: APPROVE, APPROVE WITH CHANGES
  (list what must change), or REQUEST CHANGES (list blockers)

**What good output looks like:** A numbered list of specific findings with file names, line references, and severity ratings. The security section should explicitly confirm whether user A can access user B's data — not just say "auth looks fine." The verdict at the end should be clear and actionable.

**Watch out for:** Four things. First, Deep/extended models being too polite despite the "do not compliment" instruction. Second, the review skipping the object-level authorization check. Third, findings that are vague. Fourth, the review only checking code quality without checking against the spec.

---

### 3.2 — Cross-Model Security Review

**When to use it:** Before deploying anything with authentication, payments, or sensitive user data. This is the one prompt in the library you deliberately run outside of your primary AI agent. The point is to get a genuinely different perspective from a different model family.

**How to run this practically:** Open ChatGPT, Gemini, or another AI tool in your browser. Copy the prompt below, paste in the relevant code files and the context block, and send it.

**Model:** Use a different model family than your primary agent (e.g., if you primarily use Claude, try GPT-4o or Gemini, and vice versa).

**The prompt:**
You are a security engineer reviewing code written by an AI assistant.

Context about this application (read this before reviewing the code):

- What the app does: [one sentence from your spec]
- Tech stack: [backend framework, database, ORM, auth approach from
  your spec]
- Who the users are: [from your spec — e.g. "individual users with
  personal accounts" or "team members within organizations"]
- What data is sensitive: [e.g. "passwords, email addresses, payment
  info" or "user-generated content that is private to each account"]

Your job is to find security vulnerabilities. Nothing else. Do not
suggest code quality improvements, performance optimizations, or
stylistic changes. Only security issues.

Focus on:

- OWASP Top 10 issues
- Injection attacks appropriate to the stack
- Authentication flaws (weak hashing, token/session mismanagement,
  credential leakage in errors or logs)
- Authorization flaws — specifically: can user A access, modify, or
  delete user B's data by manipulating IDs, parameters, or request
  bodies?
- Sensitive data exposure (API keys, passwords, PII in logs,
  responses, or error messages)
- Missing input validation or sanitization
- Rate limiting gaps on auth and sensitive routes
- CORS misconfiguration
- Cookie/token security (missing httpOnly, secure, sameSite flags
  if applicable)
- Any vulnerability specific to the framework and auth approach
  described above

For each vulnerability found:

1. Name the vulnerability type (e.g. "Broken Object-Level
   Authorization — OWASP A01")
2. Describe where it is — the file name and the function or route
   where it occurs
3. Explain how an attacker could exploit it in plain language —
   "an authenticated user could change the project ID in the URL
   to access another user's project"
4. Rate severity: Critical / High / Medium / Low
5. Provide a fixed version of the affected code

If you find no issues in a category, say so explicitly — "No injection
vulnerabilities found" is useful signal.

Here is the code:
[paste the relevant files — focus on auth, API routes, and any code
that handles sensitive data. You don't need to paste the entire
codebase, just the security-critical parts.]

**What good output looks like:** A focused list of findings organized by severity, with each finding naming a specific location and explaining a concrete exploit scenario. The authorization checks should specifically confirm or deny whether user A can access user B's resources.

**Watch out for:** Four things. First, false positives around ORM usage. Second, the other model ignoring the "only security issues" instruction and padding the review with code quality suggestions. Third, generic findings that don't reference your actual code. Fourth, the reviewer missing authorization checks.

---

### 3.3 — Architecture Review

**When to use it:** At the end of each major milestone — after auth is done, after the core API is done, before building the frontend, before deployment. Not for individual files or features. This is a review-only exercise — the agent should not modify any files during this process.

**Model:** Deep/extended reasoning tier with extended thinking enabled, in a **fresh conversation.**

**The prompt:**
I've completed a major milestone on this project and I want an
architectural review before moving to the next phase.

The milestone I just completed: [e.g. "the backend API", "the auth
system and core data model", "the full backend and I'm about to start
the frontend"]

The next phase I'm about to start: [e.g. "building the frontend",
"adding payment processing", "preparing for deployment"]

Before reviewing, read the full project context:

- The spec (see docs/SPEC.md) — understand what this app does, who
  the users are, and what the remaining user journeys require
- The project rules (see AGENTS.md)
- The decision log (see docs/DECISIONS.md) — understand what was
  decided and the reasoning behind each choice
- The build progress (see docs/PROGRESS.md) — understand what's
  been built, what's next, and whether any patterns are emerging
- The project structure — scan the full codebase to understand how
  files are organized, where logic lives, and how the pieces connect

Do not modify any files. This is a review only.

Review the architecture at the project level, not the file level.
Evaluate in this order:

1. Does the architecture serve the app's goals?
   - Can the remaining user journeys from the spec be built cleanly
     on top of what exists, or will the current structure force
     workarounds?
   - Is the current architecture ready for the next phase
     specifically? If I'm about to build a frontend, is the API
     surface clean enough to consume without hacks? If I'm about to
     add payments, is there a clean place for that logic to live?

2. Separation of concerns:
   - Is business logic in the right place, or has it leaked into
     route handlers, components, or the data layer?
   - Are there files doing too many things that should be split?
   - Are there abstractions that exist but aren't being used
     consistently?

3. Pattern consistency:
   - Do all files of the same type follow the same patterns?
   - Are there places where two different approaches exist for the
     same problem?
   - Are naming conventions consistent across the codebase?

4. AGENTS.md compliance at scale:
   - Check the project rules against the codebase systemically.
   - Are rules being followed everywhere, or has drift crept in?
   - If rules are being violated consistently, is it because the
     rules are wrong or because they're not being enforced?

5. Decision log validation:
   - Review each decision in DECISIONS.md. Based on what you can see
     in the codebase, have any decisions already proven to be wrong
     or costly? Should any "Revisit if" triggers be updated?

6. Missing pieces:
   - What should exist at this stage that doesn't? Common gaps:
     centralized error handling middleware, request logging, input
     validation middleware, environment variable validation at
     startup, consistent response formatting.
   - Are there utilities or helpers that are being reinvented in
     multiple places instead of shared?

7. Tech debt:
   - What shortcuts were taken that will hurt later?
   - What is cheap to fix now but will be expensive to fix after
     the next phase is built on top of it?
   - What is equally easy to fix at any point and can safely wait?

8. Scalability concerns:
   - What will break first as the app grows — more users, more data,
     more features?
   - Are there any architectural bottlenecks that would require a
     significant rewrite to address?

For each finding, tell me:

- Severity: Critical / High / Medium / Low
- Fix now or fix later — with this reasoning: if fixing it later
  will be significantly harder or more expensive because the next
  phase will be built on top of it, say "fix now." If fixing it
  later is roughly the same cost, say "fix later" and I'll add it
  to PROGRESS.md as tech debt.
- The smallest change that addresses the concern. Do not recommend
  rewrites unless the problem genuinely cannot be solved incrementally.

At the end, give me:

- A clear verdict: is the architecture ready for the next phase,
  or are there blockers?
- A prioritized list of at most 5 things to fix before moving on
- Any updates you'd recommend to DECISIONS.md based on what you found

**What good output looks like:** A review that evaluates the architecture in the context of what the app needs to do, not just whether the code is clean in isolation. The verdict at the end should be honest and actionable. "Ready with changes" and a list of 3-5 concrete things to fix is the most common healthy outcome.

**Watch out for:** Four things. First, Deep/extended models recommending a complete rewrite or major restructuring. Second, the review being too abstract. Third, the review ignoring the next phase entirely. Fourth, findings that contradict decisions in DECISIONS.md without acknowledging them.

---

### 3.4 — Quick Sanity Check

**When to use it:** Quick spot-check on a single file or function before committing. Faster than a full review (3.1), good for catching obvious issues.

**Model:** Fast/code-focused tier, fresh conversation.

**The prompt:**
This code was written by an AI in a previous session. Quick review —
only flag problems, don't explain things that are fine.

Read the file (see @[filepath]) and tell me:

1. Does it do what the function/file name says it does?
2. Will it break on null, undefined, empty inputs, or missing data?
3. Does it follow the patterns established in the rest of the codebase?
4. Is there anything a developer would flag in a code review?
5. One thing you'd change if this were your codebase

Check the project rules (see AGENTS.md) for anything obviously violated.

Keep it short. 3-5 specific observations max.

**What good output looks like:** 3-5 short, specific observations. If it comes back clean, that's useful signal too. Each observation should be actionable.

**Watch out for:** Two things. First, using this as a substitute for the full Deep/extended review (3.1) on important code. Second, the agent giving generic praise instead of flagging real issues.

---

## Section 4: Testing

---

### 4.1 — Write Tests for Code Just Built

**When to use it:** Immediately after building a feature, in the same session. The agent still has the full context of what it just wrote.

**Model:** Fast/code-focused tier

**The prompt:**
You just built [feature name]. Now write tests for it.

Before writing tests, orient yourself:

- Review the spec (see docs/SPEC.md) for the expected behavior of
  this feature — the tests should verify the spec, not just the
  implementation
- Check what testing framework and patterns are already established
  in this project. If tests already exist, match their style exactly.
- Review the user journey this feature belongs to — tests should
  verify the user-facing behavior, not just internal functions

Testing requirements:

- Use the testing framework defined in the project (check existing
  tests or AGENTS.md for the standard)
- Test behavior, not implementation — tests should pass even if the
  internal implementation changes
- Every test name should complete the sentence "it should..."
- Cover these cases for every function/endpoint:
  - Happy path (it works correctly with valid input)
  - Invalid input (it rejects gracefully with useful errors)
  - Edge cases (empty arrays, null values, boundary conditions,
    zero-length strings, maximum values)
  - Error states (dependency failures, not found, unauthorized)
  - For any route with auth: forbidden access (valid user, wrong
    resource)
- If no tests exist yet, explicitly call this out and propose a minimal smoke/regression test set before or alongside implementation. Do not skip the "run tests" step; instead, note absence and recommend creating baseline tests.
- Before running tests, ensure required setup (migrations, seed data, env vars) is applied. If setup is missing, stop and ask whether to run migrations/seed in this session; don't mark tests as failed due to missing setup.

Do not write tests that just verify the code doesn't throw. Write
tests that verify it returns the right thing with the right shape.

After writing the tests, run them and fix any failures before handing
back to me. If a test reveals a bug in the implementation, fix the
implementation too and tell me what you found.

**What good output looks like:** Test names that read like specifications. A test file where you could delete the implementation and rebuild it correctly using only the tests as a guide.

**Watch out for:** Three things. First, tests that mock everything and test nothing. Second, tests that only check status codes or "truthy" values without verifying the actual response shape and data. Third, the agent using a different testing framework or pattern than what's already established in the project.

---

### 4.2 — Fix Tests That Pass for the Wrong Reason

**When to use it:** When you want to verify that existing tests are actually protecting you, not just showing green. Use this when tests feel too easy to pass, when a feature was built and tested in the same session with no failures at all, or when you inherit tests from a previous session and want to confirm they would catch real regressions — not just that they run without errors.

**Model:** Fast/code-focused tier

**The prompt:**
These tests all pass, but I don't trust them. I think they might be
testing the wrong things.

Read the test file (see @[test filepath]) and the implementation it
tests (see @[implementation filepath]).

For each test:

1. Explain what would have to go wrong in the implementation for this
   test to fail
2. If the answer is "nothing reasonable" — the test is not useful.
   Rewrite it to test something that could actually break.
3. If the test mocks a dependency, verify that the mock behaves like
   the real dependency. A mock that always returns success doesn't test
   anything.

Then check for missing tests:
4. What behavior in the implementation has NO test covering it?
5. What edge cases (null, empty, duplicate, unauthorized, forbidden)
   are untested?

I want tests that would catch real bugs, not tests that exist to show
green checkmarks.

**What good output looks like:** The agent identifying at least 2-3 tests that need to be strengthened, and rewriting them with meaningful assertions. The "missing tests" section should identify at least one case you hadn't thought of.

**Watch out for:** The agent defending all the tests as fine. If you want to pressure-test this, ask directly: "Describe a bug in this implementation that these tests would NOT catch." If it can easily name several, the tests need work.

---

### 4.3 — End-to-End Test

**When to use it:** After a full user journey is working — registration through core feature usage. Tests the real app in a browser. Map this directly to the user journeys defined in your spec.

**Model:** Fast/code-focused tier

**The prompt:**
Write an end-to-end test for the following user journey from the spec.

Read the spec (see docs/SPEC.md) and find user journey: [name or
number of the journey, or describe it]

Before writing the test:

- Check what e2e testing framework is established in this project.
  If none exists yet, recommend one appropriate to the frontend
  framework in the spec and wait for my approval.
- Scan the existing components to identify what selectors are
  available (data-testid attributes, form labels, headings). If
  critical elements are missing test selectors, list what needs to
  be added to the components before the test can work.

Requirements:

- Use the e2e testing framework established in the project
- The test should follow the user journey step by step, exactly as
  described in the spec
- Use the page object model pattern — keep selectors and actions separate
  from test logic so they're reusable across tests
- Wait for network requests to complete — do not use arbitrary
  timeouts like waitForTimeout(2000). Use the framework's built-in
  wait mechanisms (waitForSelector, waitForResponse, etc.)
- Take a screenshot on failure for debugging
- Each step in the test should have a clear assertion — not just
  "navigate to page" but "navigate to page and verify the expected
  content is visible"
- Before running tests, ensure required setup (migrations, seed data, env vars) is applied. If setup is missing, stop and ask whether to run migrations/seed in this session; don't mark tests as failed due to missing setup.
- If no tests exist yet, explicitly call this out and propose a minimal smoke/regression test set before or alongside implementation. Do not skip the "run tests" step; instead, note absence and recommend creating baseline tests.

After writing:

- List any data-testid attributes or selectors that need to be added
  to existing components for this test to work
- Tell me how to run the test and what I should see when it passes

**What good output looks like:** A test that reads like a user story. Each step maps to a step in the spec's user journey. If it fails, you know exactly which step in the flow broke and what the user would have seen.

**Watch out for:** Three things. First, hardcoded `waitForTimeout()` calls. These are brittle and will fail on slow machines or CI environments. Second, tests that don't assert anything meaningful at each step. Third, missing setup/teardown — the test should create its own test user and clean up afterward, not depend on existing data.

---

### 4.4 — Test Coverage Audit

**When to use it:** Before a milestone review (Prompt 3.3), to find untested code paths. Run this so you can close gaps before the architecture review finds them.

**Model:** Fast/code-focused tier

**The prompt:**
Audit the test coverage for this project.

Read:

- The source files in the codebase
- The existing test files
- The spec (see docs/SPEC.md) for what behavior should be tested
- PROGRESS.md (see docs/PROGRESS.md) for what features are marked
  as implemented

For each implemented feature, tell me:

1. What source files implement this feature
2. What test files cover it (if any)
3. What behavior is tested vs. untested
4. Priority rating for the untested behavior:
   - Critical: auth logic, authorization checks, data mutation,
     payment processing, any code that writes to the database or
     handles sensitive data
   - Should have: API endpoint happy paths, component rendering with
     all states, input validation
   - Nice to have: edge cases, error message formatting, UI-only
     logic

Focus especially on:

- Auth and authorization — is every protected route tested for both
  unauthorized AND forbidden access?
- Data mutations — is every create, update, and delete operation tested?
- The user journeys — could you run through each journey's steps using
  only the existing tests as proof that each step works?

Give me a prioritized list of tests to write, starting with Critical
untested paths. For each, tell me what kind of test it needs (unit,
integration, or e2e) and roughly how many tests are needed.

**What good output looks like:** A clear picture of where you're covered and where you're exposed. The Critical untested paths should be surprising — if you already knew they were untested, you probably would have written them.

**Watch out for:** The agent listing every possible test without prioritizing. The point isn't 100% coverage — it's knowing where the dangerous gaps are. If the list has 50 items and they're all "Should have," the prioritization isn't working. Push back and ask: "Which 5 untested paths are most likely to cause a production bug?"

---

## Section 5: Security

---

### 5.1 — Pre-Deploy Security Checklist

**When to use it:** Before every deployment. Run this on the full codebase, not just new code. This is a quick sweep — the OWASP audit (5.2) goes deeper. Run this in a **fresh conversation** — the same reasoning that applies to code reviews applies here. A model that worked on this codebase has blind spots toward its own output.

**Model:** Deep/extended reasoning tier, **fresh conversation.**

**The prompt:**
Run a pre-deployment security checklist on this project.

Read the full codebase, the spec (see docs/SPEC.md) for the tech
stack and auth approach, and AGENTS.md for security-related rules.

Check for these issues, organized by severity. Adapt the checks to
the specific framework and database in the spec:

CRITICAL (do not deploy if any of these exist):

- [ ] Hardcoded API keys, secrets, or passwords anywhere in the code
      (including comments and example values that look real)
- [ ] Environment variables referenced without validation at startup
- [ ] Auth middleware missing on routes that should be protected
- [ ] User input used directly in database queries without the ORM's
      parameterization (injection risk)
- [ ] Passwords, tokens, or sensitive data logged anywhere
- [ ] User A can access User B's resources (broken object-level
      authorization)

HIGH (fix before shipping):

- [ ] Missing input validation on any route that accepts user data
- [ ] CORS configured to allow `*` for browser-facing APIs with auth/PII. `*` is only acceptable for non-auth, purely public, read-only assets. Otherwise, lock to known origins (env-driven whitelist).
- [ ] Error responses that expose stack traces, file paths, or
      internal details
- [ ] No rate limiting on auth routes
- [ ] Tokens/sessions stored insecurely (missing httpOnly, secure,
      or sameSite flags if using cookies)

MEDIUM (fix soon after launch):

- [ ] Missing HTTPS enforcement
- [ ] No request size limits
- [ ] Dependency vulnerabilities (run the package manager's audit
      command)
- [ ] Sensitive data returned in API responses that shouldn't be
      (e.g. password hash, internal IDs)
- [ ] Missing security headers (Content-Security-Policy,
      X-Content-Type-Options, etc.)

CI/CD & secrets:

- [ ] Secrets only injected via the platform's secret manager; never echoed, printed, or persisted in logs/artifacts
- [ ] `.env` and real-looking values are not committed; artifacts do not bundle `.env`
- [ ] Pipeline fails fast if required secrets/envs are missing

For each issue found:

- Name the file and location
- Explain the risk in one sentence
- Provide a one-line fix or a clear remediation step

For each check that passes, mark it explicitly — I want to see the
full checklist, not just the failures.

**What good output looks like:** A complete checklist where most items are marked as passing, with specific findings on the ones that don't. Any Critical finding is a hard blocker — do not deploy until it's fixed. The checklist should feel like a pre-flight check, not a code review.

**Watch out for:** Three things. First, Deep/extended models flagging things that aren't real issues for your specific ORM or framework. Second, the checklist skipping the object-level authorization check — this is the single most important item and should be checked explicitly for every route that accepts a resource ID. Third, Deep/extended models marking things as passing without actually checking.

---

### 5.2 — OWASP Top 10 Audit

**When to use it:** Once, after the core API is complete. This is a comprehensive, category-by-category security audit. Slower than the checklist (5.1) but much more thorough. Run this in a **fresh conversation** — bias toward your own output is especially dangerous in a security context. After running this, cross-check the results with the cross-model review (Prompt 3.2).

**Model:** Deep/extended reasoning tier, **fresh conversation** — and then cross-check with Prompt 3.2.

**The prompt:**
Audit this project against the OWASP Top 10 web application security
risks.

Read the full codebase, the spec (see docs/SPEC.md), and AGENTS.md
before starting.

For each OWASP category, tell me:

1. Whether this application is vulnerable
2. Where specifically the vulnerability exists (file + function/route)
3. The severity in context of THIS specific app and its data
   sensitivity
4. A concrete fix with code

OWASP Top 10 categories to check:

- A01: Broken Access Control — this is the #1 vulnerability in
  AI-generated code. For every route that accepts a resource ID,
  verify that the query scopes to the authenticated user. Show me
  the specific query for each route.
- A02: Cryptographic Failures
- A03: Injection
- A04: Insecure Design
- A05: Security Misconfiguration
- A06: Vulnerable and Outdated Components
- A07: Identification and Authentication Failures
- A08: Software and Data Integrity Failures
- A09: Security Logging and Monitoring Failures
- A10: Server-Side Request Forgery

For any category where the check passes, explain briefly why — "Not
vulnerable because [specific reason]."

For any category where you cannot fully assess without additional
context, tell me what you need.

At the end, give me a summary: how many Critical, High, Medium, and
Low findings, and a recommended fix order.

**What good output looks like:** A structured report organized by OWASP category where every category gets a clear verdict. The A01 (Broken Access Control) section should show you the actual queries for each route and confirm they scope to the authenticated user — this should be the longest section. Most well-built apps will have no Critical findings but several Medium ones. That's normal and expected.

**Watch out for:** Three things. First, the audit skipping or rushing through A01 (Broken Access Control). Second, false positives on A03 (Injection) when using an ORM. Third, A06 (Vulnerable Components) being hand-waved — the agent should recommend running the package manager's audit command rather than guessing about dependency vulnerabilities.

---

### 5.3 — Environment Variable Audit

**When to use it:** Before first deployment and any time you add a new integration or third-party service.

**Model:** Fast/code-focused tier

**The prompt:**
Audit this project for environment variable usage.

Scan the full codebase and:

1. List every environment variable referenced anywhere in the code
2. Check that each one is validated at application startup — not just
   when a route happens to use it. If the app can start without a
   required variable and only fails when a specific route is hit,
   that's a bug.
3. Check that none are hardcoded anywhere, including in comments,
   example code, or test files
4. Check that a .env.example file exists and includes ALL of them
   with safe placeholder values
5. Check that .env is in .gitignore
6. Check the frontend code specifically — based on the framework in
   the spec, identify any environment variables being exposed to the
   client. Only non-sensitive values should be accessible on the
   frontend. Flag any secrets, API keys, or database URLs that could
   be exposed to the browser.
7. Check for environment variables that are used but never defined
   anywhere — these will cause runtime errors in production

Provide:

- A list of all issues found with file locations
- A complete .env.example file with safe placeholder values and a
  comment explaining what each variable is for, ready to commit to
  the repo
- Frontend exposure rules: apply framework-specific prefixes (e.g., Next.js `NEXT_PUBLIC_`, Vite `VITE_`). Flag any secret used in client bundles as Critical.

**What good output looks like:** A clean .env.example file you can commit immediately. The frontend exposure check should be specific to your framework. Any secrets accessible on the frontend should be treated as a Critical issue. The startup validation check should identify any variables that would cause a silent failure if missing.

**Watch out for:** Three things. First, the agent missing variables that are only referenced through a config file or utility. Second, the .env.example including actual values that look like real credentials (even if they're not). Third, framework-specific exposure rules being wrong.

---

## Section 6: Debugging

---

### 6.1 — The Debugging Formula

**When to use it:** Every time you hit an error. Fill in all the sections before sending — the quality of the debug output scales directly with the quality of the input you provide. Don't summarize the error — paste the full thing.

**Model:** Fast/code-focused tier — switch to Deep/extended if stuck.

**The prompt:**
I have a bug I need help debugging.

**The error:**
[Paste the full error message and stack trace — do not summarize it]

**What I was trying to do:**
[One sentence: what user action or code path triggers this]

**What I already tried:**
[List anything you've already attempted. "Nothing" is a valid answer.]

**Relevant code:**
(see [filepath] — point the agent to the specific file(s) involved,
not the entire codebase)

**Environment:**

- Tech stack: [from your spec, or say "see docs/SPEC.md"]
- Dev or prod: [dev/prod]
- Did this ever work, or has it never worked? [answer]
- What changed right before it broke? [last thing you built or
  modified, if known]

Before suggesting a fix:

1. Explain what you think is causing the error
2. Tell me if there are multiple possible causes, ranked by likelihood
3. Give me the fix for the most likely cause first
4. Keep the fix as small as possible — change only what's necessary
   to resolve the error

**What good output looks like:** An explanation of root cause before any code. If the agent gives you a fix without explaining why the error happened, you'll hit the same pattern again next week. The explanation should be specific enough that you learn something.

**Watch out for:** Three things. First, the agent suggesting fixes that change a lot of code when the error is pointing to one specific place. Second, the agent guessing without reading the actual file. Third, circular debugging — if you've tried the agent's fix and it didn't work, don't just re-explain the same error. Tell the agent specifically what you tried, what happened, and ask for alternative hypotheses.

---

### 6.2 — Fresh Context Debug

**When to use it:** When you've been debugging for 30+ minutes in the same session and you're going in circles. Context collapse is probably making things worse — the agent is stuck in the same mental model that created the bug. Start a fresh conversation.

**Model:** Fast/code-focused tier — **fresh conversation.**

**The prompt:**
I've been stuck on a bug and I'm starting fresh with a clean context.

Project context: (see docs/SPEC.md for what this app does,
see AGENTS.md for the tech stack and rules)

The bug:

- Error: [paste full error message and stack trace]
- Happens when: [what user action or code path triggers this]
- First appeared: [when — after a specific change, or always been there]

I have NOT been able to fix it by: [list what you tried — be specific
so the agent doesn't suggest the same things]

I think the problem might be in: [your best guess at which file or
layer — or "I have no idea"]

Approach this like you've never seen this codebase before. Before
telling me what's wrong:

1. Read the files I've pointed to and any files they depend on
2. Ask me for any additional files you need to see
3. Form your own hypothesis before looking at my guess

Do not suggest anything I've already tried (listed above).

**What good output looks like:** The agent asking for specific files rather than guessing. A fresh perspective often spots what you've been staring past. The hypothesis should be different from what you've been investigating.

**Watch out for:** Two things. First, the agent jumping into the same failed approach you already tried. The "I have NOT been able to fix it by" section exists specifically to prevent this — if the agent ignores it, push back. Second, the agent making assumptions about the code without reading it.

---

### 6.3 — Database Debug

**When to use it:** When you have database errors, unexpected query results, or migration issues. This applies to any database and ORM.

**Model:** Fast/code-focused tier

**The prompt:**
I have a database issue.

**The error or unexpected behavior:**
[Paste exact error, or describe what you expected vs. what you got]

**The query causing the problem:**
(see @[filepath] and point to the specific query or function)

**The relevant schema/models:**
(see @[schema filepath] — point to just the models involved, not
the whole schema)

**What I've already tried:**
[List attempts]

**Context:**

- ORM and database: [from your spec, or say "see docs/SPEC.md"]
- Is this a migration issue, a query issue, or a data issue? [your
  best guess, or "not sure"]
- Did this query ever work? [yes/no/first time running it]

Please:

1. Identify whether this is a schema issue, a query issue, or a data
   issue
2. Show me the SQL the ORM would generate for this query — so I can
   verify it's doing what I think it's doing
3. If it's a migration issue, explain what state the database is in
   vs. what the schema expects
4. Give me the fix, scoped to the smallest change needed
5. If the fix involves a schema change, warn me about the impact on
   existing data and any migration steps required

**What good output looks like:** The agent identifying which layer the problem is at (schema, query, or data) before suggesting fixes. The generated SQL is the most useful output here — if you can see the actual SQL, you can often spot the issue yourself. For migration issues, the agent should explain the gap between the schema and the database state clearly.

**Watch out for:** Three things. First, the agent suggesting schema changes for what's actually a query problem, or vice versa. Second, migration fixes that don't account for existing data. Third, the agent suggesting raw SQL to fix an ORM issue — the fix should use the ORM unless there's a genuine reason it can't.

---

## Section 7: Context Management

---

### 7.1 — End-of-Session Handoff

**When to use it:** At the end of every working session, before closing your AI agent. This is referenced by AGENTS.md and multiple other prompts in this library — it's the standard process for cleanly closing a session so the next one can pick up without losing context.

**Model:** Fast/code-focused tier

**The prompt:**
We're wrapping up this session. Please do the following:

1. Update PROGRESS.md (see docs/PROGRESS.md):
   - Move anything we completed today from "Not Started" or "In
     Progress" to "Implemented"
   - Move anything we started but didn't finish to "In Progress"
     with a note on where we left off and what remains
   - Add any new items we discovered need to be built
   - If anything is blocked, move it to the "Blocked / Open
     Questions" section with context on what's needed to unblock it
   - Update session estimates if any items turned out to be larger
     or smaller than originally estimated

2. Suggest a git commit message that:
   - Starts with a type (feat/fix/refactor/test/chore)
   - Summarizes what was built in one line
   - Has 3-5 bullet points listing the specific changes
   - Includes a TODO comment for anything left unfinished

3. Update DECISIONS.md (see docs/DECISIONS.md) if we made any
   architectural decisions today. Use the standard format:
   - Decision, Why, Alternatives considered, Consequences, Revisit if
   - Be honest about the reasoning — if a decision was driven by
     time pressure or skill level, say so
   - Only log a "meaningful technical choice": auth/storage/DB/schema changes, framework/library adoption or removal, security posture changes (tokens, hashing, session strategy), deployment/infra shifts, or pattern/architecture decisions that affect multiple features. Routine bugfixes or style tweaks do not require an entry.

4. Give me a brief summary:
   - What we accomplished this session
   - What to tackle first in the next session
   - Any risks or concerns I should be thinking about before next time

**What good output looks like:** A PROGRESS.md update you can commit alongside the code. A commit message that would tell a developer in 6 months exactly what happened in this session. The next-session recommendation should be specific.

**Watch out for:** Three things. First, PROGRESS.md updates that are vague about where an in-progress item was left off. Second, the agent skipping the DECISIONS.md update. Third, the commit message having a single vague bullet point.

---

### 7.2 — Session Start Orientation

**When to use it:** At the start of a new session when you haven't touched the project in a few days and need to reorient quickly. If you set up AGENTS.md with Prompt 1.4, the session start ritual happens automatically — this prompt is for when you need more context than the automatic ritual provides, or when starting a fresh conversation.

**Model:** Fast/code-focused tier

**The prompt:**
I'm starting a new session on this project. Before we write any code,
orient me.

Read the full project context:

- SPEC.md (see docs/SPEC.md)
- PROGRESS.md (see docs/PROGRESS.md)
- DECISIONS.md (see docs/DECISIONS.md)
- AGENTS.md

Run:

- git log --oneline -10
- git status

Then tell me:

1. What was the last thing completed?
2. What is currently in progress, and where was it left off?
3. What are the next 3 things to build, in priority order from
   PROGRESS.md?
4. Are there any items in "Blocked / Open Questions" that need
   to be resolved?
5. Are there any decisions from DECISIONS.md with "Revisit if"
   triggers that might have been hit based on current progress?
6. Roughly how many sessions remain based on the current estimates
   in PROGRESS.md?

Then ask me what I want to work on today.

Do not write any code until I tell you what we're doing.

**What good output looks like:** A clear picture of where the project stands in under 60 seconds. The agent should be asking you what to work on, not proposing its own agenda.

**Watch out for:** Two things. First, the agent immediately suggesting what to work on instead of asking you. The last line exists for a reason — you should be driving the session, not the agent. Second, the orientation being so long it takes five minutes to read. This should be a quick brief, not a comprehensive report.

---

### 7.3 — Context Collapse Recovery

**When to use it:** When the agent starts suggesting the wrong framework, contradicting earlier decisions, duplicating code that already exists, or generally acting like it's forgotten the project. These are signs of context collapse — the agent has lost track of the project's accumulated context. Start a completely fresh conversation.

**Model:** Fast/code-focused tier — **fresh conversation.**

**The prompt:**
I'm resuming a project after a context collapse. Start fresh.

Read the full project context before responding:

- SPEC.md (see docs/SPEC.md)
- AGENTS.md
- PROGRESS.md (see docs/PROGRESS.md)
- DECISIONS.md (see docs/DECISIONS.md)

Also run:

- git log --oneline -20
- git status

The specific task I was in the middle of:
[describe what you were building]

The last thing that was working before context went sideways:
[describe last known good state]

Now: tell me what you understand about this project and the current
task before we proceed. Specifically:

1. What is the tech stack?
2. What has been built so far?
3. What are the project rules from AGENTS.md?
4. What is the current task and where was it left off?
5. What are the key decisions from DECISIONS.md that affect
   current work?

If your summary is wrong about anything, I'll correct you before
we continue.

**What good output looks like:** The agent demonstrating it has absorbed the project context correctly — naming the specific stack, acknowledging specific features that have been built, quoting rules from AGENTS.md, and understanding the current task with enough detail to resume. If the summary is wrong on any point, correct it before proceeding.

**Watch out for:** Two things. First, the agent giving a generic summary that could apply to any project. Push it to be specific. Second, the agent eager to start coding before you've verified its understanding. The whole point of this prompt is to rebuild context before any code is written.

---

### 7.4 — Audit Against AGENTS.md

**When to use it:** Periodically — every few sessions — to catch drift from your own standards. Also useful before a milestone review (Prompt 3.3) as preparation.

**Model:** Fast/code-focused tier

**The prompt:**
Audit the current codebase against our project rules.

Read AGENTS.md and then scan the codebase.

For each rule in AGENTS.md, check whether the existing code follows
it. Report back as:

- [x] Rule is followed consistently across the codebase
- [~] Rule is followed mostly but with exceptions (list every
  exception with file name and location)
- [ ] Rule is violated (list every violation with file name and
  location)

Pay special attention to:

- Type safety rules (any shortcuts or workarounds)
- Error handling consistency
- Naming conventions
- Any items in the "Forbidden" section — these should have zero
  violations
- Security rules — these should also have zero violations

For any rule that is consistently violated ([~] or [ ]):

- Is the code wrong, or is the rule impractical?
- If the rule is impractical, suggest a revised version that's
  more realistic while still protecting what the rule was designed
  to protect

At the end, tell me:

- Total rules followed consistently
- Total rules with exceptions
- Total rules violated
- The top 3 most important violations to fix first

**What good output looks like:** At least a few `[~]` items — it's rare to be fully consistent across an entire codebase. The findings become your technical debt list. The most valuable output is the "is the code wrong or is the rule impractical" assessment, because it helps you evolve your AGENTS.md over time.

**Watch out for:** Two things. First, the agent marking everything as `[x]` without actually scanning files. Second, the agent suggesting you remove rules rather than fix the code. The default assumption should be that violations need to be fixed — only revise the rule if the violation pattern reveals that the rule genuinely doesn't make sense for your project.

---

## Section 8: Refactoring & Code Health

---

### 8.1 — Targeted Refactor

**When to use it:** When a specific file or function has grown too complex. Use plan mode to review the refactoring plan before the agent changes anything.

**Model:** Fast/code-focused tier

**The prompt:**
This code needs to be refactored. It works correctly — do not change
behavior.

The file: (see @[filepath])

What's wrong with it: [describe the problem — too long, doing too
many things, hard to read, inconsistent with the rest of the codebase,
flagged in architecture review]

Before refactoring, orient yourself:

- Read the file and understand what it does in the context of the app
- Trace its dependencies — what imports it, what does it import
- Check whether tests exist for this file (see @[test filepath] if
  known, otherwise scan the test directory)
- Review the patterns used in the rest of the codebase — the
  refactored version should match those patterns, not introduce
  new ones

Constraints:

- All existing tests must still pass after the refactor
- Do not change function signatures or exports that other files
  depend on — check the dependency trace first
- Do not introduce new dependencies during refactor unless strictly necessary to fix correctness/security. If you believe a new dep is required, pause and propose: why it's needed, alternatives using the standard library/existing deps, security/maintenance impact, and size the change to the developer's skill level. Never add on your own without explicit user approval.
- The refactored version must be readable without comments
  explaining what it does
- If no tests exist for this file, recommend writing tests BEFORE
  refactoring so we have a safety net. Wait for my approval on
  whether to write tests first or proceed without them.

Give me your refactoring plan:

1. List the specific problems you see
2. Describe what you'll change and why
3. Describe what you will NOT change
4. Wait for my approval before proceeding

After I approve and switch to act mode:

- Implement the refactor
- Before running tests, ensure required setup (migrations, seed data, env vars) is applied if relevant; don't mark tests as failed due to missing setup.
- Run all existing tests to confirm nothing broke
- If the refactor created any new functions or modules, write tests
  for them
- Tell me what changed so I can verify the behavior is preserved

**What good output looks like:** Code you can read top to bottom without getting lost. The refactored version should be shorter or the same length as the original. The dependency trace should confirm that nothing outside the file was affected. If tests existed, they all still pass.

**Watch out for:** Four things. First, the agent changing behavior during a refactor. "Refactor" means restructure, not rewrite. Second, the agent introducing new patterns that don't exist elsewhere in the codebase. Third, the refactor growing in scope — "while I was in there, I also improved..." means scope creep. Fourth, the agent skipping the test check. Refactoring without tests is flying blind — if no tests exist, seriously consider writing them first.

---

### 8.2 — Dependency Audit

**When to use it:** Before deployment and any time you notice the project's dependency list has grown unexpectedly. AI assistants are notoriously eager to add packages — this prompt catches the unnecessary ones.

**Model:** Fast/code-focused tier

**The prompt:**
Audit the dependencies in this project.

Read the dependency file (package.json, requirements.txt, Cargo.toml,
go.mod — whatever is appropriate for the stack defined in the spec)
and scan the codebase for actual usage.

For each dependency:

1. Is it actually imported and used somewhere in the codebase? Search
   for actual imports, not just the dependency listing.
2. Is it doing something that could be done natively with the language
   or standard library instead?
3. Is there a lighter alternative that would do the same job? Only
   flag this if the current package is significantly larger than needed
   for what we're using it for.
4. Are there multiple packages doing similar things?

Flag any dependency that was probably added by an AI assistant without
asking — these are often unnecessary. Common signs: the package does
one small thing that could be done in a few lines of code, or the
package is only used in one file for one function.

Also check:

- Run the package manager's audit command for known vulnerabilities
- Are dev dependencies correctly separated from production dependencies?
- Are there any dependencies pinned to specific versions that should
  be updated?

Give me a final recommendation:

- Dependencies to remove (with reasoning)
- Dependencies to replace (with what and why)
- Dependencies that are fine and should stay
- Any security vulnerabilities found in the audit

**What good output looks like:** At least 2-3 dependencies flagged as unnecessary or replaceable. The best finding is a package that's imported but only used for one function that could be replaced with 5 lines of native code — removing it reduces your attack surface and bundle size.

**Watch out for:** Three things. First, the agent recommending removal of packages that ARE necessary but are imported indirectly (through a config file or framework convention rather than a direct import statement). Second, the agent recommending lighter alternatives that don't actually have the same feature coverage. Third, the audit missing indirect dependencies that have vulnerabilities. The package manager's audit command catches these — make sure the agent actually runs it rather than guessing.

---

## Quick Reference Card

| Task | Prompt # | Model |
| ------ | ---------- | ------- |
| Reverse-engineer SPEC.md | 0.1 | Deep/extended |
| Reverse-engineer AGENTS.md | 0.2 | Fast/code-focused |
| State of the Union (PROGRESS/DECISIONS) | 0.3 | Fast/code-focused |
| Delta Audit (Alignment Check) | 0.4 | Deep/extended |
| Turn idea into spec | 1.1 | Deep/extended |
| Create build plan | 1.2 | Fast/code-focused |
| Document decisions | 1.3 | Fast/code-focused |
| Create AGENTS.md | 1.4 | Fast/code-focused |
| Choose tech stack | 1.5 | Deep/extended |
| Build any feature | 2.1 | Fast/code-focused (plan → act) |
| Design database schema | 2.2 | Deep/extended (plan → act) |
| Build API endpoints | 2.3 | Fast/code-focused (plan → act) |
| Build UI components | 2.4 | Fast/code-focused (plan → act) |
| Build auth system | 2.5 | Deep/extended (plan → act) |
| Modify complex code safely | 2.6 | Fast/code-focused (plan mode) |
| Standard code review | 3.1 | Deep/extended (fresh session) |
| Security review | 3.2 | Cross-model (different model family) |
| Architecture review | 3.3 | Deep/extended + extended thinking (fresh session) |
| Quick sanity check | 3.4 | Fast/code-focused (fresh session) |
| Write tests | 4.1 | Fast/code-focused |
| Validate test quality | 4.2 | Fast/code-focused |
| End-to-end tests | 4.3 | Fast/code-focused |
| Test coverage audit | 4.4 | Fast/code-focused |
| Pre-deploy checklist | 5.1 | Deep/extended (fresh session) |
| OWASP audit | 5.2 | Deep/extended + cross-model (fresh session) |
| Environment variable audit | 5.3 | Fast/code-focused |
| Debug an error | 6.1 | Fast/code-focused → Deep/extended if stuck |
| Fresh context debug | 6.2 | Fast/code-focused (fresh session) |
| Database debug | 6.3 | Fast/code-focused |
| End of session | 7.1 | Fast/code-focused |
| Session start | 7.2 | Fast/code-focused |
| Context collapse recovery | 7.3 | Fast/code-focused (fresh session) |
| AGENTS.md audit | 7.4 | Fast/code-focused |
| Refactor messy code | 8.1 | Fast/code-focused (plan → act) |
| Dependency audit | 8.2 | Fast/code-focused |

---

*Version 2.4 — Refined for agent-agnostic usage across any AI coding tool, plan/act workflow, and existing codebase onboarding. AGENTS.md replaces CLAUDE.md as the canonical rulebook; tool-specific wrappers bridge the gap. Expand this library as you discover prompts that work. When you find a prompt that reliably produces better output than what's here, replace it.*
