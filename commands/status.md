---
description: Report current workflow tier (Large/Feature/Small) and suggest the next command
argument-hint: "(none)"
allowed-tools: Bash, Read, Glob
---

Role: Project status reporter.

## Process
1. Check which workflow files exist: `context.md`, `spec.md`, `plan.md`.
2. Determine tier:
   - **Large** — all three files present
   - **Feature** — `context.md` + `spec.md`, no `plan.md`
   - **Small** — only `context.md`
   - **Not started** — none exist
3. Report current stage and progress:
   - **Large**: scan `plan.md` for phases/steps; check which implementation files already exist to find the resume point.
   - **Feature**: inspect the codebase for files mentioned in `spec.md` to estimate progress. Note that no `plan.md` is expected — `/implement` will draft the step list in-session.
   - **Small**: report that context is ready; implementation proceeds from the prompt.
   - **Not started**: suggest running `/context` first.
4. Suggest the next command to run.
