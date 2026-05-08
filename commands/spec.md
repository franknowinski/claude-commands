---
description: Build a feature spec (acceptance criteria, scope, edge cases) from context.md
argument-hint: "(none)"
allowed-tools: Read, Write, Edit
---

Role: Senior Product Engineer speccing out a task.

## Prerequisites
`.claude/workflow/context.md` must exist. If missing, tell the user to run `/context` first and stop.

## Process
1. Read `.claude/workflow/context.md` to understand the feature.
2. Ask one focused question at a time to build the spec — each question builds on the last.
3. After each answer, note which section you're populating (e.g., "✓ Adding to: Acceptance Criteria").
4. Before finalizing, suggest names for any new tables, services, interactors, organizers, or jobs — wait for confirmation.
5. "skip" moves to the next section.

## Sections to cover (as relevant to task complexity)
- **Problem/Goal** — the "why"
- **Acceptance Criteria** — how we know it's done
- **Scope boundaries** — what's explicitly OUT
- **Data/State changes** — new tables, columns, migrations
- **Feature flag strategy** — gradual rollout, kill switch
- **Edge cases & error handling**
- **Observability** — Datadog signals to emit (`add_meta` for request-scoped facts, `ReportCustomEvent` for errors/business events)
- **Dependencies/blockers**
- **Testing approach**

## Output
Compile into `.claude/workflow/spec.md`.

## Next step
After `spec.md` is written, recommend the next command based on task tier:
- **Large/risky/multi-day** → `/plan` to break the work into steps
- **Feature-sized** → `/implement` (drafts step list in-session)
- **Small** → `/implement` directly
Suggest a fresh session before the next command to keep context lean.

## Rules
- If the user suggests something that isn't best practice, explain what best practice is and why — let them decide.
