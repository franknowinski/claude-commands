# Workflow
- Always present proposed code changes as a diff and get approval before editing files. Present one file (and its spec, if applicable) at a time — do not batch multiple files into a single review.
- Never push to remote or rebase. The user handles all git push and rebase operations.
- Ask one question at a time. If requirements are ambiguous, present options and let me choose — don't guess.
- When working in /Users/fnowinski/Projects/glue, read /Users/fnowinski/.claude/projects/-Users-fnowinski-Projects-glue/CLAUDE.md for additional project context.
- When creating PRs, always read `.github/pull_request_template.md` first (if it exists) and use it as the basis for the `--body` content
passed to `gh pr create`.

# Scope Discipline
- Don't add features, refactoring, or abstractions beyond what I asked for. If you spot something worth changing, mention it — don't do it.
- Don't add error handling, validation, or fallbacks for cases that can't happen. Trust internal callers. Only validate at real system boundaries.
- Default to no comments. Only add one when the *why* is non-obvious (hidden constraint, workaround, surprising behavior).
- Before writing new logic, check whether it already exists in the codebase.

# Style and Conventions
- Match the codebase's mechanical conventions (naming, formatting, file structure) without exception.
- For architectural choices — error handling, abstractions, state flow — if you'd write it differently from existing patterns, surface the tradeoff and let me decide before diverging.

# Error Handling
- Raise specific error types, not catch-all rescues that hide root causes.
- Error messages must include enough context to debug (params, response bodies, status codes).

# Testing
- Write specs/tests alongside the implementation. Test coverage is part of "done."
- **Do not run specs.** I'll run them manually. In your summary, list which spec files I should run before merging.

# Linting
- When running rubocop, always pass `--cache false` (e.g. `bundle exec rubocop --cache false`). The sandbox blocks writes to `tmp/`, so the default cache directory causes an "Operation not permitted" failure.

# Documentation
- If a code change affects behavior described in docs, update the docs in the same pass.

# Debugging
- If a fix fails 3 times in a row, stop. State the problem clearly, share what you've ruled out, and ask how to proceed. Do not keep retrying variations of the same approach.

# Dependencies
- Don't install gems or packages globally. Add them to the project config (Gemfile, package.json) so they're tracked.

# Reporting Completion
- When you claim a task is done, state explicitly what you verified and what you didn't (e.g., "edited 3 files, listed specs to run, did not run specs").

# Observability
- Prefer Datadog over Rails.logger for request-scoped observability. Use `ReportingClient::Current.add_meta(key: value)` for request-scoped facts and `ReportCustomEvent.call` for errors and business events. Reserve `Rails.logger` for infrastructure-level concerns outside the request lifecycle (initializers, rake tasks, boot errors).
- NewRelic is deprecated and no longer used. Do not add NewRelic instrumentation, references, or dependencies.

# Specs
## Spec Values
- In RSpec, always reference `let` variables in expectations instead of repeating literal values. For example, use `eq(monthly_income)` not `eq(5000.0)`, and `eq(monthly_income - monthly_expenses)` not `eq(1550.0)`. Hardcoded literals in expectations create silent inconsistencies when the `let` value changes.

## File Conventions
- When you create a new file in `app/`, add it to `.github/CODEOWNERS` in alphabetical order.

%% Testing command i saw on LinkedIn to produce better results
Codex will review your output once you are done
