# Claude Commands

Slash commands for [Claude Code](https://claude.com/claude-code) that turn ad-hoc work into a repeatable pipeline:

```
Small task   → /context (inline spec) → /implement
Feature      → /context → /spec → /implement
Large/risky  → /context → /spec → /plan → /implement
Open the PR  → /pr → gh pr create
```

## Install

Claude Code reads slash commands from `~/.claude/commands/`. Drop these `.md` files there:

```bash
git clone https://github.com/franknowinski/claude-commands.git /tmp/claude-commands
mkdir -p ~/.claude/commands
cp /tmp/claude-commands/commands/*.md ~/.claude/commands/
```

Type `/` in any Claude Code session to see them.

Add this to your repo's `.gitignore` so workflow files don't get committed:

```
.claude/workflow/
```

## Commands

| Command | Writes | Purpose |
|---|---|---|
| `/context` | `context.md` (+ `spec.md` on Small tier) | Pull the ticket, explore the codebase in parallel subagents; on Small tier, also runs an inline spec interview |
| `/spec` | `.claude/workflow/spec.md` | Interview-driven spec: acceptance criteria, scope, edge cases |
| `/plan` | `.claude/workflow/plan.md` | Break large work into a step-by-step roadmap |
| `/implement` | code | Walks the work; auto-detects mode by which files exist |
| `/pr` | `.claude/workflow/pr-body.md` | Fill the repo's PR template from `spec.md`, ready for `gh pr create` |
| `/status` | — | Reports current tier and the next command |
| `/pr-review` | — | Multi-dimension review of a branch or PR |

Workflow files live in `.claude/workflow/` (project-local, gitignored) — they're how state crosses sessions.

## Best practices

- **Fresh session between commands.** Each step loads context the next doesn't need; the workflow files persist state, so exit and restart between commands. Stay in one session only for tightly coupled follow-ups.
- **Answer one question at a time.** Commands ask sequentially on purpose. `skip` moves past a section.
- **Approve diffs before edits.** `/implement` proposes the diff and waits per step.
- **Run specs yourself.** `/implement` lists which to run.
- **Lost? Run `/status`** — it re-reads the workflow files and tells you the resume point.
- **Regenerate `/pr` per task.** `.claude/workflow/pr-body.md` persists across tasks until overwritten — always re-run `/pr` for the current branch before opening a PR.

## Walkthrough: your first task

Say you're picking up `MAR-1234 — Add daily statement email`. A Feature-sized task.

```
# 1. Branch
git checkout -b mar-1234-daily-statement-email

# 2. Gather context (in Claude Code)
/context
# → paste the Linear card, answer constraints, /context explores the codebase
# → recommends "Feature" tier, suggests a fresh session

# 3. Spec (fresh session)
/spec
# → interview-driven; covers AC, scope, edge cases, observability

# 4. Implement (fresh session)
/implement
# → drafts the step list, you approve
# → walks each step: proposes diff → you approve → it implements + writes specs
# → run the listed specs yourself

# 5. PR
/pr
# → fills .github/pull_request_template.md from spec.md, writes to .claude/workflow/pr-body.md
gh pr create --body-file .claude/workflow/pr-body.md --web
# → opens the GitHub web UI with the body pre-filled
```

For a Small task (one-line fix), `/context` runs an inline spec interview and writes both files — go straight to `/implement` after, no fresh session needed.
For a Large task (multi-day, multi-service), add `/plan` between `/spec` and `/implement`.

## Authoring your own command

Drop a Markdown file in `~/.claude/commands/`:

```markdown
---
description: One-line summary
argument-hint: "[arg] or (none)"
allowed-tools: Bash, Read, Write, Edit, Grep, Glob, Agent
---

Role: ...

## Process
1. ...
```

These commands assume a Rails / ActiveForce / Salesforce / Datadog / Linear stack — the standard Beyond stack. The `sf-go-proxy` repo is an exception; edit the prompts when working there.
