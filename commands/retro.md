---
description: Capture a feature retrospective and propose improvements to your workflow commands
argument-hint: "[PR number | branch name]  (omit for current branch)"
allowed-tools: Bash, Read, Write, Edit, AskUserQuestion
---

Role: Engineering coach conducting a feature retrospective.

## Purpose
Learn from completed work. Captures what the workflow missed — spec gaps, implementation surprises, review findings — and proposes specific improvements to your commands after every retro.

## Step 0: Resolve the feature

Based on `$ARGUMENTS`:
- **No argument** → current branch (`git branch --show-current`)
- **Numeric** → PR number. Fetch with `gh pr view $ARGUMENTS --json number,title,body,headRefName,mergedAt`
- **Non-numeric** → treat as branch name. Find PR with `gh pr list --head $ARGUMENTS --json number,title,body,mergedAt --limit 1`

Run in parallel:
- `git log main..<branch> --oneline`
- PR metadata (from above)
- `gh pr view <number> --json reviews,comments` — review comments (if PR found)
- Check for `.claude/workflow/context.md`, `spec.md`, `plan.md`

State back to the user in 2–3 lines: feature name, PR number (if found), whether workflow files exist. Stop if no branch can be resolved.

## Step 1: Read artifacts

- All workflow files present in `.claude/workflow/`
- PR review comments and inline review threads (from Step 0)
- Synthesize: what did the spec promise vs. what reviewers flagged vs. what got built

## Step 2: Interview — one question at a time

Ask each question, wait for the answer, then move to the next. "skip" moves on.

1. **Spec gaps** — What did the spec miss that only became clear during implementation or review?
2. **Implementation surprises** — Any step that took significantly longer than expected, or required a different approach than planned?
3. **Review findings** — Did `/pr-review` flag the important issues, or did human reviewers catch things it missed? What were those things?
4. **What worked** — Any part of the workflow (a command, the spec interview, the step-by-step plan) that was particularly useful for this type of work?
5. **One change** — If you ran this feature again tomorrow, what's the single thing you'd do differently?

## Step 3: Write retro

Determine the branch slug: take the branch name, strip the Linear ID prefix if present (e.g. `mar-1260-`), truncate to 40 chars, replace non-alphanumeric with hyphens.

Save to `~/.claude/retros/YYYYMMDD_<branch-slug>.md`:

```markdown
---
date: YYYY-MM-DD
feature: <PR title or branch name>
pr: <PR number or "none">
branch: <branch name>
commands_used: [list which of /context /spec /plan /implement /pr /pr-review were run]
---

## Spec Gaps
[What the spec missed]

## Implementation Surprises
[Unexpected complexity or direction changes]

## Review Findings
[What human reviewers caught that /pr-review missed — or confirmed it caught]

## What Worked
[Workflow elements worth keeping]

## One Change
[Single most valuable improvement]

## Proposed Command Improvements
[Populated during Step 4 — diffs proposed and their outcomes]
```

Tell the user the file path.

## Step 4: Propose command improvements

Analyze all interview answers and PR review comments for actionable patterns. Group findings by command:

- `/spec` — questions it didn't ask that mattered
- `/plan` — phases it missed or ordered wrong
- `/implement` — approval flow gaps, missing constraints
- `/pr-review` — dimensions it skipped that reviewers caught
- `/context` — codebase areas it didn't explore that were relevant

For each command with a finding, propose a specific diff — show the exact before/after text in the command file. Present one command at a time. Wait for approval or rejection before moving to the next.

After each proposal, the user can:
- **Approve** → apply the edit to `~/.claude/commands/<command>.md`
- **Reject** → note the rejection in the retro file so `/evolve` can weigh it across retros
- **Modify** → incorporate their change, confirm, then apply

If a finding is too vague to produce a specific diff (e.g., "felt slow"), write it to the retro's `## Proposed Command Improvements` section with a note that `/evolve` will revisit it when there are more data points.

## Rules
- Never edit command files without explicit approval on the specific diff.
- If no PR exists yet (feature still in progress), skip the PR fetch in Step 0 and note it in the retro.
- Migration note: when moving to team mode, change the write path from `~/.claude/retros/` to the context repo's `retros/` folder. File format stays the same.
