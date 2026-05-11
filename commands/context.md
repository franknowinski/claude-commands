---
description: Gather context (Linear card + codebase exploration) for a new feature
argument-hint: "(none)"
allowed-tools: Bash, Read, Grep, Glob, Agent, Write
---

Role: Senior Product Engineer gathering context for a feature.

## Purpose
Build a `context.md` file that gives downstream prompts (/spec, /plan, /implement) everything they need.

## Process
1. Ask the user to paste the Linear card or ticket description. Wait for response.
2. Ask for any additional context — constraints, stakeholder notes, related work, things to avoid. If they say "skip", move on.
3. Use the **Explore** subagent (`subagent_type=Explore`) to investigate each area below — Explore is read-only and keeps main context lean. Launch all five in a **single message** so they run in parallel:
   - Models involved (ActiveRecord and ActiveForce `app/models/af/`) — associations, validations
   - Entry points — relevant controllers, services, interactors, organizers, jobs
   - Similar features — existing code with patterns this feature could reuse
   - External dependencies — API clients, background jobs, Salesforce objects
   - Existing feature flags that may be relevant
4. Present findings one section at a time for user confirmation.
5. Compile into `.claude/workflow/context.md` (create the directory if needed) using the template below.
6. Estimate task tier (small / feature / large) based on findings. Present your recommendation with reasoning and let the user confirm or override.
7. **If Small** — run a brief inline spec interview, one question at a time, covering:
   - Acceptance criteria
   - Scope boundaries (what's explicitly out)
   - Top 2–3 edge cases
   - Observability — any `add_meta` or `ReportCustomEvent` needed
   - Testing approach (which spec files to write)
   Write `.claude/workflow/spec.md` with these sections. Recommend `/implement` next — no fresh session needed, context is already loaded.
8. **If Feature or Large** — recommend a fresh session before `/spec` (exploration loaded files the spec interview doesn't need). For Large, mention `/plan` follows `/spec`.

## Template

```
# Context for [Feature Name]

## Linear Card
- **Description**: [From the pasted ticket]
- **Acceptance Criteria**:
  - [ ] ...

## Additional Context
[User-provided constraints, clarifications, stakeholder notes]

## Models Involved
<!-- Auto-populated by Claude — verify these are correct. -->

## Entry Points
<!-- Auto-populated by Claude — verify these are correct. -->

## Similar Features
<!-- Auto-populated — existing patterns this feature should follow. -->

## External Dependencies

## Feature Flags
- Existing: ...
- Needed: ...

## Existing Observability
<!-- add_meta keys and ReportCustomEvent calls in similar features — reference for /spec -->

## Constraints
[Anything to avoid, backward compatibility needs, in-progress refactors]
```
