---
description: Multi-dimension PR review against ticket acceptance criteria
argument-hint: "[PR number | branch name]  (omit to review current branch)"
allowed-tools: Bash, Read, Grep, Glob, Agent
---

You are conducting a thorough code review. Follow the steps below in order. **Do not skip the confirmation checkpoints.**

## Step 0: Gather Context

Resolve what to review based on `$ARGUMENTS`:

- **No argument** → review the current branch vs `main`. Get the branch name with `git branch --show-current`.
- **Numeric argument** (e.g. `1234`) → treat as PR number. Use `gh pr view $ARGUMENTS --json title,body,headRefName,baseRefName` and `gh pr diff $ARGUMENTS`.
- **Non-numeric argument** → treat as branch name. Diff against `main`.

Then gather:

1. **Diff** — full diff of the changes (`git diff main...<branch>` or `gh pr diff`).
2. **Files changed** — list of touched files.
3. **Ticket ID** — parse from branch name (format: `MAR-NNNN-description` or similar) or PR title.
4. **Acceptance criteria** — from the PR body if available. If reviewing a local branch with no PR, ask the user to paste the ticket description / acceptance criteria.
5. **Commit messages** — `git log main..<branch> --oneline` for intent context.

State the resolved context back to the user in 3–4 lines (branch, ticket, # files changed, base) before proceeding.

## Step 1: Plan the Review

Based on the nature of the changes, determine which dimensions apply. Include only the relevant ones:

| Dimension | When to include |
|---|---|
| **Approach & Technical Design** | Always |
| **Acceptance Criteria Validation** | Always |
| **Regression Bugs** | Changes touch shared logic, alter existing behavior, or modify interfaces |
| **Design Patterns** | New abstractions, services, or architectural decisions |
| **Coding Style** | Always |
| **Test Coverage** | New logic added; check unit, integration, and External Call Count specs |
| **Performance** | Queries, loops, or data-intensive operations |
| **Readability & Maintainability** | Always |

Present the plan and wait for confirmation:

> Here is my plan for reviewing `[BRANCH]` against `[TICKET ID]`:
>
> [List of dimensions + 1-line description of what will be evaluated in each]
>
> Please confirm or let me know what adjustments are needed.

**Stop. Wait for the user's response before continuing.**

## Step 2: Execute Review

Work through the approved dimensions **one at a time**. After each dimension, present findings and wait for the user to acknowledge before moving to the next.

### Approach & Technical Design
- Does the implementation follow a logical approach?
- Are technical decisions sound and justified?
- Are there better approaches that should have been considered?

### Acceptance Criteria Validation
For each criterion: state **fully met / partially met / not addressed**. Cite specific files/lines that satisfy it, or explain what is missing.

### Regression Bugs
Investigate blast radius:
- Trace method calls, callbacks, and side effects affected by the change.
- Check changes to shared utilities, base classes, or public interfaces.
- Identify callers of modified methods (`grep` for usages) and verify they're unaffected.

### Design Patterns
- Right abstractions used (services, interactors, organizers)?
- Existing patterns followed consistently?
- Better patterns that should have been applied?

### Coding Style
- Naming conventions, method length, class organization.
- Idiomatic Ruby / Rails usage; built-in helpers applied where appropriate.
- Avoidance of anti-patterns present elsewhere in the codebase.
- **Observability**: Datadog (`ReportingClient::Current.add_meta`, `ReportCustomEvent.call`) preferred over `Rails.logger` for request-scoped events. Flag any new `Rails.logger` calls in request paths. Flag any NewRelic references (deprecated).

### Test Coverage
- All new code paths covered by unit specs?
- Edge cases and error conditions tested?
- Tests diagnostic when they fail (clear `let` names, no hardcoded literals where a `let` exists)?
- **External Call Count specs** present/updated where external HTTP calls are made?
- Integration specs present/updated for end-to-end scenarios?

### Performance
- N+1 query risks (note them even if non-blocking).
- Inefficient loops or redundant operations.
- Missing indexes or overly broad queries.

### Readability & Maintainability
- Will future engineers understand the intent?
- Variable and method names clear and descriptive?
- Logic decomposed at appropriate level?
- **Dev utilities applied**: new files in `app/` added to `.github/CODEOWNERS` in alphabetical order? Feature map assignments correct?

## Step 3: Summary

After all dimensions are reviewed, present a consolidated summary:

1. **Overall Assessment** — brief verdict on quality.
2. **Required Changes** — must-fix before merge.
3. **Suggested Improvements** — non-blocking recommendations.
4. **Positive Observations** — done well, worth reinforcing.
