---
description: Implement an approved task — auto-detects Large/Feature/Small mode by which workflow files exist
argument-hint: "(none)"
allowed-tools: Bash, Read, Write, Edit, Grep, Glob, Agent
---

Role: Staff engineer implementing a task from approved context.

## Mode detection
Operate based on which workflow files exist:

- **Large mode** — `context.md` + `spec.md` + `plan.md` all present. Walk `plan.md`.
- **Feature mode** — `context.md` + `spec.md` present, no `plan.md`. Draft the step list in-session, get approval, then walk it.
- **Small mode** — only `context.md` present. Implement directly from the prompt as a single pass.

If `context.md` is missing, tell the user to run `/context` first and stop.

## Setup (varies by mode)
- **Large**: read `context.md`, `spec.md`, `plan.md`. Find first un-done step in `plan.md`.
- **Feature**: read `context.md`, `spec.md`. Draft a step list using the codebase phase order (Schema/Migrations → Models → Services/Interactors/Organizers → Jobs → Controllers → Specs → Integration wiring). For each step: file path(s), one-line objective, blueprint file to reference. Present and wait for approval before any code.
- **Small**: read `context.md` and the user's prompt.

## Per-step loop
1. Read any blueprint/reference files for this step. If the step touches schema, also read `db/structure.sql`.
2. Propose the diff — do NOT edit yet. Wait for approval.
3. After approval, implement and write specs.
4. Report results, list the spec files for the user to run, and wait for confirmation before the next step.

## Re-planning (Feature mode only)
If a later step shows the in-session plan is wrong, re-plan from that step. Show the revised list and get approval before continuing.

## Constraints
- One step at a time — do not skip ahead.
- Flag any conflicts between plan and spec — ask how to proceed.
- Do not run specs — list them for the user.
- Propose changes and get explicit approval before editing — show the diff.
- If the user suggests something that isn't best practice, explain it and let them decide.

## Resuming
- **Large**: check which files from `plan.md` already exist; resume there.
- **Feature**: inspect what's been built, re-draft the remaining step list, get approval.
- **Small**: ask the user where you left off.
