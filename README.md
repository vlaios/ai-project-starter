# ai-project-starter

Starter files for AI-assisted software projects. Create a new repo from this one, fill the documents in order, and start building with review, debugging and security agents in place from the first commit.

These templates are opinionated. They encode one way of working: decide why before what, decide what before how, record every significant decision, and treat agents as part of the project from day one, not a later phase.

## Quick start

On GitHub: open https://github.com/vlaios/ai-project-starter, click **Use this template**, then **Create a new repository**, and clone your new repo.

Or locally, PowerShell (Windows):

```powershell
$name = "my-project"
git clone https://github.com/vlaios/ai-project-starter.git $name
Set-Location $name
Remove-Item -Recurse -Force .git
git init
```

bash (WSL, Linux):

```bash
name="my-project"
git clone https://github.com/vlaios/ai-project-starter.git "$name"
cd "$name"
rm -rf .git
git init
```

Removing `.git` drops this repo's history, so your project starts clean. `.gitignore` is already in place, so the first commit is safe. Run `git status` before it anyway.

## Fill order

1. `CONSTITUTION.md`: why the project exists and what it must never do.
2. `PROJECT-BRIEF.md`: what we build and which decisions are open.
3. `IMPLEMENTATION-PLAN.md`: how we build it, phase by phase.
4. `CLAUDE.md`: how Claude works inside this repo.
5. `.env.example`, `.gitignore`, skills: adapt to the chosen stack.
6. `README.md` and `LICENSE`: replace them (see the last two sections).
7. `decisions/`: one ADR per significant decision, written when the decision is made.

Each document reads the one above it and never contradicts it. If the plan conflicts with the brief, or the brief with the constitution, fix the upper document first or change the plan.

## What each file does

| File | What it is | Why it is there |
|---|---|---|
| `README.md` | This guide | Explains the starter. Rewritten in every new project. |
| `LICENSE` | MIT license | Lets others reuse the starter. Replaced or removed in new projects. |
| `CONSTITUTION.md` | Ten fixed sections: Identity, Diagnosis, Who It Serves, Guiding Policy, Non-Negotiables, Outcomes, Boundaries, The Test, Open Questions, Governance | Gives every later decision something to be checked against. Without it there is no way to tell drift from progress. |
| `PROJECT-BRIEF.md` | Vision, scope, stack, open and resolved decisions, status | Single place to see what is being built and what is still undecided. |
| `IMPLEMENTATION-PLAN.md` | Phases, each with a goal, deliverables and done criteria | Turns the brief into work that can be executed and verified step by step. |
| `CLAUDE.md` | Stack, commands, conventions and working rules for Claude Code | Loaded at the start of every Claude Code session, so Claude never starts blind. |
| `decisions/ADR-000-template.md` | Architecture Decision Record template | Records why a decision was made, so nobody has to reconstruct it six months later. |
| `.claude/agents/code-reviewer.md` | Subagent that reviews changes for correctness and clarity | Review from the first commit, not after the codebase has grown. |
| `.claude/agents/debugger.md` | Subagent for root-cause analysis | First call when something breaks: evidence before fixes. |
| `.claude/agents/security-auditor.md` | Subagent that checks for secrets, injection, auth and dependency risks | Security issues are cheapest to fix before they ship. |
| `.claude/skills/*-coding-style/SKILL.md` | Coding standards for PHP, Python and TypeScript | Claude writes code in one consistent style per language. Delete the stacks you do not use. |
| `.env.example` | Every environment variable with a placeholder and a comment | Documents configuration without leaking secrets. The real `.env` is never committed. |
| `.gitignore` | Base rules plus sections per stack | Keeps secrets, build output and editor files out of git from the first commit. |

## Rules for the documents

**Constitution.** Project scope, maximum 500 words, one named owner. Every section is filled: `UNKNOWN` is allowed if it names who resolves it and by when, a blank section is not. Set `inherits: null` for a standalone project. If a parent constitution exists, the project may narrow it but never contradict it. Changes are versioned (semver) and always state a reason.

**Brief.** References the constitution, never restates it. When an open decision is resolved and has real consequences, it moves into an ADR and the brief links to it.

**ADRs.** Copy
