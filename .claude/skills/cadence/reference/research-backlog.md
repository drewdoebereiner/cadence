# Research Backlog

## Overview

For each backlog ticket, dispatch a parallel research subagent that investigates the technical domain and posts a structured comment with findings directly on the ticket.

## Pre-flight: Confirm API Key

```bash
echo $LINEAR_API_KEY
```

If empty, ask the user now. Do not proceed without a working key.

## CRITICAL: curl only, never MCP

All Linear reads and writes use curl + GraphQL. Never use MCP Linear tools for mutations.

## Step 1: Identify Target Team

Ask the user which Linear team to research, or default to the team from `$LINEAR_TEAM_NAME`.

Fetch the team ID:

```bash
curl -s -X POST https://api.linear.app/graphql \
  -H "Authorization: $LINEAR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"query": "{ teams { nodes { id name } } }"}'
```

## Step 2: Fetch Backlog Issues

Fetch issues in `backlog` and `unstarted` state types:

```bash
curl -s -X POST https://api.linear.app/graphql \
  -H "Authorization: $LINEAR_API_KEY" \
  -H "Content-Type: application/json" \
  -d "$(jq -n --arg teamId "$TEAM_ID" \
    '{query: "query($teamId: String!) { issues(filter: { team: { id: { eq: $teamId } }, state: { type: { in: [\"backlog\", \"unstarted\"] } } }) { nodes { id identifier title description } } }", variables: {teamId: $teamId}}')"
```

Skip tickets with no title and no description.

## Step 3: Dispatch Parallel Research Agents

Dispatch one subagent per ticket, all in parallel. Each subagent receives the ticket title and description and returns a structured comment with:

- **What** -- one sentence on what this ticket is asking for
- **Recommended approach** -- concrete technical direction
- **Key resources** -- relevant docs, libraries, prior art
- **Considerations** -- architectural tradeoffs
- **Gotchas** -- known failure modes, edge cases
- **Complexity** -- S / M / L / XL with one sentence justification

## Step 4: Post Comment to Each Ticket

For each ticket, post the research comment using curl:

```bash
curl -s -X POST https://api.linear.app/graphql \
  -H "Authorization: $LINEAR_API_KEY" \
  -H "Content-Type: application/json" \
  -d "$(jq -n --arg issueId "$ISSUE_ID" --arg body "$COMMENT_BODY" \
    '{query: "mutation($issueId: String!, $body: String!) { commentCreate(input: { issueId: $issueId, body: $body }) { success } }", variables: {issueId: $issueId, body: $body}}')"
```

## Step 5: Report to User

Output a summary table: Ticket | Title | Complexity | Comment posted (yes/no)

## Common Mistakes

| Mistake | Correct behavior |
|---|---|
| Sequential research agents | All agents in parallel |
| Using MCP for comment creation | curl + GraphQL mutation only |
| Skipping tickets with no description | Skip only if both title and description are empty |
| Generic "look at the docs" comments | Specific resources, concrete approach |
