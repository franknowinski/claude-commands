# Claude Commands

Slash commands for [Claude Code](https://claude.com/claude-code) that turn ad-hoc work into a repeatable pipeline:

```
Small task   → /context → /implement
Feature      → /context → /spec → /implement
Large/risky  → /context → /spec → /plan → /implement
```

## Install

Claude Code reads slash commands from `~/.claude/commands/`. Drop these `.md` files there:

```bash
git clone https://github.com/franknowinski/claude-commands.git /tmp/claude-commands
mkdir -p ~/.claude/commands
cp /tmp/claude-commands/commands/*.md ~/.claude/commands/
```

Type `/` in any Claude Code session to see them.

## Commands

| Command | Writes | Purpose |
|---|---|---|
| `/context` | `context.md` | Pull the ticket, explore the codebase in parallel subagents |
| `/spec` | `spec.md` | Interview-driven spec: acceptance criteria, scope, edge cases |
| `/plan` | `plan.md` | Break large work into a step-by-step roadmap |
| `/implement` | code | Walks the work; auto-detects mode by which files exist |
| `/status` | — | Reports current tier and the next command |
| `/pr-review` | — | Multi-dimension review of a branch or PR |

`context.md`, `spec.md`, and `plan.md` live in your **project root** — they're how state crosses sessions.

## Best practices

- **Fresh session between commands.** Each step loads context the next doesn't need; the workflow files persist state, so exit and restart between commands. Stay in one session only for tightly coupled follow-ups.
- **Answer one question at a time.** Commands ask sequentially on purpose. `skip` moves past a section.
- **Approve diffs before edits.** `/implement` proposes the diff and waits per step.
- **Run specs yourself.** `/implement` lists which to run.
- **Lost? Run `/status`** — it re-reads the workflow files and tells you the resume point.

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

These commands assume a Rails / ActiveForce / Salesforce / Datadog / Linear stack. Edit the prompts to match yours — they're just Markdown.
