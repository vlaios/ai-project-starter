# CLAUDE.md

Instructions for Claude Code in this repository. Keep this file short: it is loaded into every session.

## Project

<!-- One sentence: what this project is. -->

UNKNOWN

Read before any non-trivial work:

- [CONSTITUTION.md](CONSTITUTION.md): why, boundaries, non-negotiables
- [PROJECT-BRIEF.md](PROJECT-BRIEF.md): what, stack, open decisions
- [IMPLEMENTATION-PLAN.md](IMPLEMENTATION-PLAN.md): current phase and next step
- [decisions/](decisions/): accepted decisions, do not reopen without a new ADR

## Commands

<!-- Real commands only. State the shell for each. -->

| Task | Command | Shell |
|---|---|---|
| Setup | UNKNOWN | |
| Run | UNKNOWN | |
| Test | UNKNOWN | |
| Lint | UNKNOWN | |

## How we work

1. For anything bigger than a small fix, propose a short plan (files, structure, approach) and wait for approval.
2. Change one file at a time. Before editing, state the file, the location (function, class or method, not line numbers), what changes and why. Wait for approval.
3. Never guess what a file contains. Read it first.
4. No silent changes. Every change, even a config value, is named and justified.
5. After each change, report: what changed, why, files touched, how to test, open risks, parked ideas.

## Two modes

- **Implementation (default):** execute the agreed plan as written. No new designs or alternatives mixed in.
- **Problem-solving:** on any gap, contradiction, risk or error, stop. Start with "Pause: problem" and state what it is, where, why it matters, and what the plan currently says. Wait for a decision, then return to implementation.
- A plan change is always its own question, never bundled with routine confirmations.
- "ok" or "go" approves only what was explicitly asked.
- If unsure whether something is a problem, treat it as one.
- Improvement ideas go into the report as parked ideas. Nothing is acted on until picked up.

## Architecture

- Prefer fewer moving parts. Remove a component before adding one.
- Hide storage and external services behind thin abstraction layers.
- Follow the matching coding style skill in `.claude/skills/`.

## Agents

- **code-reviewer:** before every commit of a logical unit.
- **debugger:** whenever something fails. Root cause with evidence before any fix.
- **security-auditor:** before every merge and every release.

## Versions

Before pinning any dependency, image or runtime, verify the latest stable version live from the official source. Never from memory. Record it in the brief's stack table.

## Git

- Conventional Commits, one line: `type(scope): short description`. Lowercase, imperative, no trailing period, under 72 characters.
- Suggest a commit after each logical unit. Remind to check `git status` before committing.

## Security

- Secrets never go into code or commits. Use `.env`, keep `.env.example` with placeholders.
- Default to secure settings: least privilege, short-lived tokens. If told otherwise, say it once.

## Writing

- Docs, comments and READMEs in English.
- Deploy and setup commands in docs are the real ones, no placeholders.
- Never use the em dash. Use commas, colons, parentheses or separate sentences.
