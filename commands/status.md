---
description: Report current workflow tier (Large/Feature/Small) and suggest the next command
argument-hint: "(none)"
allowed-tools: Bash, Read, Glob
---

Role: Project status reporter.

## Process
1. Check which workflow files exist in `.claude/workflow/`: `context.md`, `spec.md`, `plan.md`.
2. Determine tier:
   - **Large** — all three files present in `.claude/workflow/`
   - **Feature** — `context.md` + `spec.md` in `.claude/workflow/`, no `plan.md`
   - **Small** — only `.claude/workflow/context.md`
   - **Not started** — none exist
3. Report current stage and progress:
   - **Large**: scan `.claude/workflow/plan.md` for phases/steps; check which implementation files already exist to find the resume point.
   - **Feature**: inspect the codebase for files mentioned in `.claude/workflow/spec.md` to estimate progress. Note that no `plan.md` is expected — `/implement` will draft the step list in-session.
   - **Small**: report that context is ready; implementation proceeds from the prompt.
   - **Not started**: suggest running `/context` first.
4. Suggest the next command to run.
