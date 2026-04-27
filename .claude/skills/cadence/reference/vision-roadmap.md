# Vision Roadmap

## Overview

Act as a product visionary: read the repo and all documentation, ask clarifying questions, then produce a strategic roadmap document and create Linear backlog tickets via the Linear REST/GraphQL API.

The goal is not bug hunting. The goal is imagining what this product needs to become.

## Before You Begin: Check the Linear API Key

**Do this before reading a single file.** You cannot create tickets without it, and discovering a missing key after building the entire roadmap wastes everyone's time.

```bash
echo $LINEAR_API_KEY
```

If the variable is empty or unset, ask the user to provide the key now. Do not proceed to Phase 1 until you have confirmed a working key.

## CRITICAL: Linear tickets use curl, never MCP

All Linear mutations (create issue, update issue) must use `curl` with the GraphQL API endpoint. Never use MCP Linear tools for writes -- they have caused silent failures and data loss.

## Phase 1: Context Gathering

Read the following in order:
1. `README.md`
2. All files in `docs/` if present
3. `CLAUDE.md` or `AGENTS.md`
4. `git log --oneline -50` to understand recent work
5. Open Linear tickets (use curl to fetch)

## Phase 2: Clarifying Questions (never skip)

Ask exactly these 7 questions before generating anything:

1. Who is the primary user of this product right now?
2. What is the single most important problem it solves today?
3. What would "10x better" look like in 6 months?
4. Are there any hard constraints (tech stack, team size, budget)?
5. What has already been tried and abandoned?
6. Which competitors or comparable products should we be aware of?
7. Is there a deadline or milestone driving the next phase?

Do not proceed to Phase 3 until all 7 questions are answered.

## Phase 3: Vision Generation

Produce a table of 3-5 strategic themes. For each theme:
- One sentence describing the theme
- 2-3 initiatives under it
- Rough effort (S/M/L/XL)
- Why it matters now

## Phase 4: Roadmap Document

Write `ROADMAP-<YYYY-MM>.md` to the repo root. Structure:
- Executive summary (3 sentences max)
- Strategic themes table from Phase 3
- Quarter-by-quarter initiative plan
- Dependencies and risks

## Phase 5: Linear Ticket Creation

For each initiative in Phase 3, create a Linear ticket using curl:

```bash
curl -s -X POST https://api.linear.app/graphql \
  -H "Authorization: $LINEAR_API_KEY" \
  -H "Content-Type: application/json" \
  -d "$(jq -n --arg title "$TITLE" --arg desc "$DESC" --arg teamId "$TEAM_ID" \
    '{query: "mutation { issueCreate(input: { title: $title, description: $desc, teamId: $teamId }) { success issue { id identifier } } }", variables: {}}')"
```

Priority mapping: Core infrastructure -> Urgent, Growth features -> High, Nice-to-have -> Medium.

Issue template:
```
## Context
<why this initiative matters>

## Success criteria
<how we know it's done>

## Open questions
<unknowns to resolve during implementation>
```

## Phase 6: Summary Report

Output a table: Ticket identifier | Title | Theme | Priority | Linear URL

Then print total counts: tickets created, themes covered, estimated total effort.

## Common Mistakes

| Mistake | Correct behavior |
|---|---|
| Starting roadmap before confirming API key | Check key first, always |
| Skipping clarifying questions | All 7 questions, no exceptions |
| Creating tickets with MCP tools | Use curl + GraphQL only |
| Bug list instead of strategic vision | Focus on what the product needs to become |
| Single mega-ticket per theme | One ticket per initiative |
| Relative dates in tickets | Use absolute dates (YYYY-MM-DD) |
