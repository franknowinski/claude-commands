---
description: Break large/risky work into a step-by-step implementation roadmap
argument-hint: "(none)"
allowed-tools: Read, Write, Edit, Agent
---

Role: Senior Staff Engineer creating an implementation roadmap.

## When to use
Only for **large/risky/multi-day work** or tasks that hand off across sessions or people. Feature-sized work should skip `/plan` — `/implement` drafts the step list in-session.

## Prerequisites
`context.md` and `spec.md` must exist. If either is missing, tell the user which to create first (`/context` → `/spec`) and stop.

## Process
1. Read `context.md` and `spec.md`.
2. Draft a high-level blueprint of the solution architecture. Present it and wait for approval before detailing steps.
3. Break into phases aligned with this codebase:
   - Schema/Migrations → Models → Services/Interactors/Organizers → Jobs → Controllers (if needed) → Specs → Integration wiring
4. Break each phase into small, iterative steps that build on each other. Use subagents to find blueprint files in the codebase — keeps main context lean.
5. Convert each step into a standalone prompt for `/implement`.
6. Before writing `plan.md`, verify every acceptance criterion in `spec.md` is covered by ≥1 step. Flag any AC not covered.

## For each step include
- **Phase & Step** (e.g., "Phase 2, Step 3")
- **File(s):** exact path(s) to create or modify
- **Objective:** one-sentence technical goal
- **Blueprint:** existing file to reference for patterns (if applicable)
- **Edge cases:** spec.md edge cases this step handles (if any)
- **Verification:** how `/implement` knows the step is done — test cases, expected output, or behavior to observe
- **Depends on:** previous step(s) that must be complete
- **Prompt:** the prompt for the implementing agent (in a code block)

## Constraints
- No orphaned code — each step integrates with previous work
- Order to minimize merge conflicts and circular dependencies
- End with wiring/integration steps
- No actual code — prompts only

## Output
Write to `plan.md` in the project root.

## Next step
After `plan.md` is written, recommend `/implement` (Large mode) and suggest starting a fresh session so the implementing agent walks `plan.md` with a clean context.

## Rules
- If the user suggests something that isn't best practice, explain what best practice is and why — let them decide.
