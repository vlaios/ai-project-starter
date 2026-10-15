---
project: <project-slug>
version: 0.1.0
status: draft
owner: <one named person>
updated: <YYYY-MM-DD>
---

# Project Brief: <Project Name>

<!--
How to use this document:
- The constitution says WHY. This brief says WHAT. Never restate the
  constitution here: link to it.
- Update it whenever a decision is resolved or scope changes, and bump
  the version (minor for scope or decisions, patch for wording).
- Write UNKNOWN instead of guessing.
-->

## 1. Summary

<!-- Two or three sentences: what gets built and for whom.
     The why lives in CONSTITUTION.md. -->

UNKNOWN

See [CONSTITUTION.md](CONSTITUTION.md) for purpose, boundaries and non-negotiables.

## 2. Scope

<!-- Feature level, more concrete than the constitution's boundaries. -->

**In scope (v1):**

- UNKNOWN

**Out of scope (v1):**

- UNKNOWN

## 3. Key use cases

<!-- Concrete flows: who does what, and what happens.
     One line each. These become the basis for tests. -->

1. UNKNOWN

## 4. Architecture

<!-- Main components, how data flows between them, and every external
     service. Each storage layer and external service sits behind a thin
     abstraction, so it can be swapped later. Prefer fewer moving parts:
     justify every component. -->

UNKNOWN

## 5. Stack

<!-- Verify every version live (PyPI, npm, Docker Hub, official release
     notes) on the date you write it down. Never from memory. -->

| Layer | Choice | Version | Verified on | Why |
|---|---|---|---|---|
| UNKNOWN | | | | |

## 6. Agents

<!-- Agents are part of the project from day one. List what each one checks
     and when it runs. Remove a row only with a reason. -->

| Agent | Checks | Runs when |
|---|---|---|
| code-reviewer | Correctness, clarity, adherence to the coding style skills | Before every commit of a logical unit |
| debugger | Root cause, with evidence, before any fix | Whenever something fails |
| security-auditor | Secrets, injection, auth, dependencies, insecure defaults | Before every merge and every release |

## 7. Decisions

<!-- Every open question that affects the build. When resolved with real
     consequences, write an ADR and link it. Resolved rows stay as history. -->

| ID | Question | Status | Answer | ADR |
|---|---|---|---|---|
| D-01 | UNKNOWN | open | | |

## 8. Risks

| Risk | Impact | Mitigation |
|---|---|---|
| UNKNOWN | | |

## 9. Status

**Current phase:** not started. See [IMPLEMENTATION-PLAN.md](IMPLEMENTATION-PLAN.md).

## Changelog

| Version | Date | Change |
|---|---|---|
| 0.1.0 | <YYYY-MM-DD> | Initial draft |
