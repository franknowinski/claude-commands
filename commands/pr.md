---
description: Generate a PR body by filling the repo's pull_request_template with spec.md content (paired with `makepr`)
argument-hint: "(none)"
allowed-tools: Bash, Read, Write, AskUserQuestion
---

Role: Engineer preparing a PR body for completed work.

## Pairing
Writes `.claude/workflow/pr-body.md`, then opens a browser PR draft via `gh pr create --web`.
The user clicks "Create pull request" (or "Draft") in the browser to finalize.

## Prerequisites
- A repo PR template must exist at one of: `.github/pull_request_template.md`, `PULL_REQUEST_TEMPLATE.md`, `.github/PULL_REQUEST_TEMPLATE.md`. If none exist, tell the user and stop.

## Process
1. Pick the PR content source, in priority order:
   - **Preferred:** `.claude/workflow/spec.md` exists → use it. Also read `.claude/workflow/context.md` if present.
   - **Fallback 1:** `.claude/workflow/plan.md` exists → use it.
   - **Fallback 2:** neither exists → derive content from `git log main..HEAD` (commit messages) and `git diff main...HEAD --stat` (file changes). Inform the user you're falling back to diff-only mode and suggest they consider running `/spec` next time for richer PR bodies.
2. Read the repo's PR template (first match in the fallback list above).
3. Run in parallel for PR context:
   - `git branch --show-current`
   - `git log main..HEAD --oneline`
   - `git diff main...HEAD --stat`
4. Derive Linear metadata from the branch name (no API call needed):
   - Extract the short ID: match `^([a-z]+-\d+)-` and uppercase it (e.g. `mar-1260-add-foo` → `MAR-1260`)
   - Extract the slug: everything after `SHORT_ID-` (e.g. `add-structured-data-nodes-to-home-page`)
   - Format the slug as a title: replace hyphens with spaces and title-case each word
   - Construct the Linear URL: `https://linear.app/beyond-finance/issue/SHORT_ID/slug`
   - Determine ticket type: scan `git log main..HEAD --oneline` for `| FEAT |`, `| FEATURE |`, `| BUG |`, `| CHORE |`, `| SPIKE |` — use the first match, default to `CHORE` if none found
5. Fill the template's sections from the chosen content source, using best-effort matching on section headers:
   - "Summary", "Description", "What", "Why", "Overview" → spec/plan Problem-Goal; if diff-only mode, synthesize 1-3 bullets from the commit messages.
   - "Acceptance Criteria", "AC", "Requirements" → spec/plan AC as unchecked `- [ ]`; if diff-only, leave the template's placeholder.
   - "Out of Scope", "Non-goals" → spec/plan Scope boundaries; if diff-only, leave the template's placeholder.
   - "Test Plan", "Testing", "How to Test", "QA" → spec/plan Testing approach as unchecked `- [ ]`; if diff-only, synthesize a checklist of spec files to run from `git diff --stat` (any `*_spec.rb` in the diff).
   - "Observability", "Monitoring", "Metrics" → spec/plan Observability section; if diff-only, leave the template's placeholder.
   - Any other section → leave the template's existing placeholder/comment text untouched.
6. Prepend the Linear card link as the very first line of the body (before any template sections):
   `![](https://avatars.githubusercontent.com/u/46686594?s=14&v=4) [Formatted Title](https://linear.app/beyond-finance/issue/SHORT_ID/slug)`
7. Show the filled body to the user. Wait for approval or edits.
8. After approval, write to `.claude/workflow/pr-body.md`.
9. Build the PR title: `SHORT_ID | TYPE | Formatted Title` (e.g. `MAR-1260 | FEAT | Add Structured Data Nodes To Home Page`). If no Linear ID could be derived from the branch, fall back to the branch name itself as the title.
10. **Stage changes for commit (skip if `git status --porcelain` is empty):**
    - Run `git status` to surface modified + untracked paths.
    - Filter out anything that looks sensitive: `.env*` (except `.env.example` / `.env.template`), files matching `**/secret*`, `**/*.key`, `**/credentials*`.
    - Show the filtered list and ask the user which to stage (default: all of them; let the user veto specific paths). Never use `git add -A` or `git add .`.
    - After approval, run `git add <each path>` (specific paths, not globs).
    - Run `git diff --staged --stat` and confirm with the user before committing.
11. **Commit the staged changes (skip if step 10 was skipped):**
    - Commit message line 1: the PR title from step 9.
    - Commit message body: 1–2 sentences summarizing the change, derived from the same source used for the PR body's Summary section.
    - Always append `Co-Authored-By: Claude Opus 4.7 <noreply@anthropic.com>`.
    - Pass the message via a HEREDOC per the global commit-style instructions.
    - Do not skip hooks (`--no-verify`). If a pre-commit hook fails, fix the issue and create a NEW commit (do not amend).
12. Check if the branch is pushed: run `git status -sb`. If no remote tracking branch exists, tell the user:
    - Push with: `git push -u origin HEAD`
    - Then re-run `/pr` (it will detect the pushed branch and open the GH PR draft), OR run this manually after pushing:
      ```
      gh pr create --title "<title>" --body-file .claude/workflow/pr-body.md --base main --web
      ```
    Stop here — do not auto-push.
13. If the branch IS already pushed, run `gh pr create --title "SHORT_ID | TYPE | Formatted Title" --body-file .claude/workflow/pr-body.md --base main --web` and tell the user the PR was opened in the browser.

## Rules
- Do not auto-check Acceptance Criteria or Test Plan boxes — reviewers/the user check them.
- Do not invent content for sections that have no source — keep the template's placeholder.
- In diff-only mode, do not fabricate AC/risk/observability content from a diff. Summarize what changed, not why.
- Do not delete sections from the template even if they're empty.
- Never push to remote. The user always pushes.
- Never commit secrets (`.env`, `*.key`, `credentials*`). Filter before staging.
- If `git status --porcelain` is empty, skip staging/commit steps and go straight to the push check.
