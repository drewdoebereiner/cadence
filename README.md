# cadence

Agentic SDLC automation for the full development lifecycle.

> **Install:** `cp -r .claude/skills/cadence ~/.claude/skills/` then use `/cadence <sub-command>` in Claude Code.

---

## Why

Your AI assistant handles individual tasks. Cadence handles the lifecycle -- from roadmap to merged PR -- without you managing the handoffs.

Each sub-command runs as a bulk pass: it picks up everything in your backlog or PR queue and advances it in one shot. Run it manually or drop it into cron and walk away.

Schedule `research-backlog` to enrich new tickets overnight. Schedule `dev-backlog` to implement five of them. Schedule `bulk-pr-review` so PRs don't sit. The whole loop runs without you.

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

Run `/cadence` with no sub-command to see the full list.

---

## Designed Workflow

The sub-commands form a full SDLC loop:

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

### Run it on a schedule

Each sub-command is stateless and idempotent. Running one twice leaves no duplicate work. Use Claude Code's `/schedule` command to wire the loop into cron:

```
# Enrich new backlog tickets every morning
/schedule "0 8 * * 1-5" /cadence research-backlog

# Implement up to 5 tickets every weeknight
/schedule "0 21 * * 1-5" /cadence dev-backlog

# Review all open PRs each afternoon
/schedule "0 14 * * 1-5" /cadence bulk-pr-review

# Fix review comments before standup
/schedule "0 9 * * 1-5" /cadence fix-pr-comments
```

`research-backlog` skips tickets it has already commented on. `dev-backlog` only picks up unstarted tickets. `fix-pr-comments` only touches PRs with open feedback.

---

## Installation

**Claude Code:**

```bash
cp -r .claude/skills/cadence ~/.claude/skills/
```

Then use `/cadence <sub-command>` in any Claude Code session.

**From this repo:**

```bash
git clone https://github.com/drewdoebereiner/cadence.git
cp -r cadence/.claude/skills/cadence ~/.claude/skills/
```

---

## Why Linear

Three of the six sub-commands read and write Linear. Its GraphQL API is clean and well-documented. Ticket states (backlog, unstarted, in review) map directly to the stages cadence moves work through. Comment threads persist between agent runs, so research findings are already on the ticket when the implementer picks it up. For agents running on a schedule, that combination means full programmatic control with no UI work required.

Linear was built with developer tooling in mind. That shows in the API. It's why cadence uses it rather than a more generic project management tool.

---

## Requirements

Environment variables required per sub-command:

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
