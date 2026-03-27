# Skill: End-of-Session Handoff

## Purpose

Close out a working session cleanly by updating project docs, drafting a commit message, and leaving a clear starting point for the next session.

## When the agent should use this

- The user says "we're done for today", "let's wrap up", "end of session", "that's it for now", or similar
- The user asks the agent to update PROGRESS.md at the end of a session
- Referenced by AGENTS.md or other skills as the standard session-close process

## Opening interview

This skill requires no interview. All the information needed — what was built, what was started, what decisions were made — is available from the current conversation, the codebase, and the docs. Proceed directly to the protocol.

## Protocol

1. Update [docs/PROGRESS.md](docs/PROGRESS.md):
   - Move everything completed this session from "Not Started" or "In Progress" to "Implemented"
   - Move anything started but not finished to "In Progress" with a specific note on where it was left off and what remains to complete it
   - Add any new items discovered during the session that need to be built
   - Move any blocked items to "Blocked / Open Questions" with context on what is needed to unblock them
   - Update session estimates for any items that turned out to be larger or smaller than originally estimated

2. Suggest a git commit message that:
   - Starts with a type prefix: `feat`, `fix`, `refactor`, `test`, or `chore`
   - Summarizes what was built in one line
   - Lists 3–5 bullet points of specific changes made
   - Includes a TODO comment for anything left unfinished

3. Update [docs/DECISIONS.md](docs/DECISIONS.md) if any architectural decisions were made this session. Only log a meaningful technical choice — auth, storage, database or schema changes, framework or library adoption or removal, security posture changes (tokens, hashing, session strategy), deployment or infrastructure shifts, or patterns that affect multiple features. Routine bugfixes and style tweaks do not need an entry. For each decision that qualifies, use the standard format:
   - **Decision** — what was decided
   - **Why** — the reasoning, including if it was driven by time pressure or skill level
   - **Alternatives considered**
   - **Consequences**
   - **Revisit if** — the condition under which this decision should be reconsidered

4. Give a brief summary:
   - What was accomplished this session
   - What to tackle first in the next session (be specific, not generic)
   - Any risks or concerns worth thinking about before next time

## What good output looks like

A PROGRESS.md update ready to commit alongside the code. A commit message that tells a developer six months from now exactly what happened in this session. The next-session recommendation names a specific task, not just a category of work.

## Watch out for

- **Vague in-progress notes.** "Started auth" is not enough — the note should say exactly where work stopped and what the next concrete step is.
- **Skipping DECISIONS.md.** If an architectural decision was made, it must be logged even if the session ran long.
- **Thin commit messages.** A single vague bullet point is not acceptable — the message should be reconstructable from the bullet points alone without reading the diff.
