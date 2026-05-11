---
description: Generate a PR body by filling the repo's pull_request_template with spec.md content (paired with `makepr`)
argument-hint: "(none)"
allowed-tools: Bash, Read, Write
---

Role: Engineer preparing a PR body for completed work.

## Pairing
Writes `.claude/workflow/pr-body.md` only. Does NOT create the PR.
After approval, the user creates the PR with `gh pr create --body-file .claude/workflow/pr-body.md --web` (or their preferred PR tool).

## Prerequisites
- `.claude/workflow/spec.md` must exist. If missing, tell the user to run `/spec` first and stop.
- A repo PR template must exist at one of: `.github/pull_request_template.md`, `PULL_REQUEST_TEMPLATE.md`, `.github/PULL_REQUEST_TEMPLATE.md`. If none exist, tell the user and stop.

## Process
1. Read `.claude/workflow/spec.md` (and `.claude/workflow/context.md` if present).
2. Read the repo's PR template (first match in the fallback list above).
3. Run in parallel for PR context:
   - `git branch --show-current`
   - `git log main..HEAD --oneline`
   - `git diff main...HEAD --stat`
4. Fill the template's sections from `spec.md` using best-effort matching on section headers:
   - Headers like "Summary", "Description", "What", "Why", "Overview" → spec.md Problem/Goal
   - Headers like "Acceptance Criteria", "AC", "Requirements" → spec.md Acceptance Criteria as unchecked `- [ ]`
   - Headers like "Out of Scope", "Non-goals" → spec.md Scope boundaries
   - Headers like "Test Plan", "Testing", "How to Test", "QA" → spec.md Testing approach as unchecked `- [ ]`
   - Headers like "Observability", "Monitoring", "Metrics" → spec.md Observability section
   - Any other section → leave the template's existing placeholder/comment text untouched
5. Show the filled body to the user side-by-side with which sections were auto-filled vs. left as-is. Wait for approval or edits.
6. After approval, write to `.claude/workflow/pr-body.md`.
7. Run `/Users/fnowinski/.path_scripts/makepr` via Bash.
8. Tell the user the PR was opened in the browser.

## Rules
- Do not auto-check Acceptance Criteria or Test Plan boxes — reviewers/the user check them.
- Do not invent content for sections that have no spec.md source — keep the template's placeholder.
- Do not delete sections from the template even if they're empty.
- Do not run `gh pr create`. Body generation only — the user runs the PR creation step themselves.
