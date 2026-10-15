---
project: <project-slug>
version: 0.1.0
status: draft
owner: <one named person>
updated: <YYYY-MM-DD>
---

# Implementation Plan: <Project Name>

<!--
How to use this document:
- The brief says WHAT. This plan says HOW, in order. Never restate the
  brief: link to it.
- Every phase has a goal, deliverables and done criteria someone else
  could verify. "Works" is not a done criterion.
- Execute the plan as written. If a step conflicts with the brief or the
  constitution, stop and fix the upper document or change the plan
  explicitly. Every plan change gets a changelog entry.
- Ideas found during implementation go to "Parked ideas", not into the
  current phase.
-->

Based on [PROJECT-BRIEF.md](PROJECT-BRIEF.md).

## Overview

| Phase | Goal | Status |
|---|---|---|
| 1 | Foundation: runnable skeleton, agents active, tests running | not started |
| 2 | UNKNOWN | not started |

<!-- Status values: not started, in progress, done. -->

## Phase 1: Foundation

**Goal:** a clean clone can be set up from the README alone, tests run, and all three agents have checked the code once.

**Deliverables:**

- [ ] Project skeleton following the matching coding style skill
- [ ] `.env.example` lists every variable the code reads
- [ ] README setup, run and test commands, verified on a clean clone
- [ ] Test runner set up with at least one passing test
- [ ] code-reviewer, debugger and security-auditor run once against the skeleton

**Done when:**

- [ ] A fresh clone, following only the README, reaches a running app and green tests
- [ ] security-auditor reports no open findings, or each finding is logged as a risk in the brief
- [ ] No secret appears in `git log -p`

**Commits:** one per logical unit, Conventional Commits.

## Phase 2: <Name>

**Goal:** UNKNOWN

**Deliverables:**

- [ ] UNKNOWN

**Done when:**

- [ ] UNKNOWN

<!-- Copy the Phase 2 block for every further phase. -->

## Parked ideas

<!-- Improvements found while implementing. Nothing here is acted on until
     it is moved into a phase. -->

- None yet.

## Changelog

| Version | Date | Change |
|---|---|---|
| 0.1.0 | <YYYY-MM-DD> | Initial draft |
