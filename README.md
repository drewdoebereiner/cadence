# cadence

Agentic SDLC automation for the full development lifecycle.

> **Install:** `cp -r .claude/skills/cadence ~/.claude/skills/` then use `/cadence <sub-command>` in Claude Code.

---

## Why

AI coding assistants are powerful at the task level but blind to the lifecycle level. Cadence gives your AI harness the context and process it needs to operate across the full SDLC -- from product vision through implementation, testing, and review -- without you coordinating every step.

Each sub-command is a self-contained workflow that knows what to do, when to stop, and when to ask.

---

## What's Included

### The Skill

One skill, six sub-commands. Invoke as `/cadence <sub-command>`.

### Sub-commands

| Sub-command | What it does |
|---|---|
| `vision-roadmap` | Read the repo, ask 7 clarifying questions, produce a strategic roadmap doc, create Linear backlog tickets via GraphQL |
| `research-backlog` | Fetch all backlog tickets, dispatch parallel research agents, post structured findings as comments |
| `dev-backlog` | Pull up to 5 Todo tickets by priority, implement each on its own branch, open PRs targeting dev, move tickets to In Review |
| `write-unit-tests` | Discover test framework from codebase, identify coverage gaps from git diff, write pattern-matched tests |
| `fix-pr-comments` | Find all open PRs with CHANGES_REQUESTED or unresolved threads, fix every issue, push, reply to threads with commit SHA |
| `bulk-pr-review` | Map PR dependency layers, dispatch parallel review subagents per layer, post APPROVE / REQUEST CHANGES / BLOCKED-CI verdicts to GitHub |

---

## Usage

```
/cadence vision-roadmap
/cadence research-backlog
/cadence dev-backlog
/cadence write-unit-tests
/cadence fix-pr-comments
/cadence bulk-pr-review
```

Invoke `/cadence` with no sub-command to see the full list.

---

## Designed Workflow

The sub-commands compose into a full SDLC loop:

```
vision-roadmap      # What should we build?
      |
research-backlog    # What do we need to know before building?
      |
dev-backlog         # Build it.
      |
write-unit-tests    # Cover it.
      |
bulk-pr-review      # Review everything in flight.
      |
fix-pr-comments     # Close the loop on review feedback.
```

---

## Installation

**Claude Code:**

```bash
cp -r .claude/skills/cadence ~/.claude/skills/
```

Then use `/cadence <sub-command>` in any project.

**From this repo:**

```bash
git clone https://github.com/drewdoebereiner/cadence.git
cp -r cadence/.claude/skills/cadence ~/.claude/skills/
```

---

## Requirements

Some sub-commands require environment variables:

| Sub-command | Requires |
|---|---|
| `vision-roadmap` | `LINEAR_API_KEY` |
| `research-backlog` | `LINEAR_API_KEY` |
| `dev-backlog` | `LINEAR_API_KEY`, `gh` CLI authenticated |
| `write-unit-tests` | none |
| `fix-pr-comments` | `gh` CLI authenticated |
| `bulk-pr-review` | `gh` CLI authenticated |

---

## Supported Tools

- Claude Code (Claude CLI)

---

## License

MIT
