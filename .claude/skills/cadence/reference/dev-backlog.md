# Dev Backlog

## Overview

For each Todo ticket: implement on a feature branch, open a PR targeting `dev`, move ticket to In Review. Max 5 tickets per run.

## Configuration

Set these before running:
- `PROJECT_ROOT` -- absolute path to the repo
- `LINEAR_TEAM_NAME` -- your Linear team name

## Pre-flight: Confirm API Key

```bash
echo $LINEAR_API_KEY
```

If empty, stop and ask the user.

## CRITICAL: curl only, never MCP

All Linear reads and writes use curl + GraphQL.

## Step 1: Sync Repo

```bash
git status  # fail if dirty working tree
git fetch upstream
git checkout dev
git merge --ff-only upstream/dev
```

Fail on dirty tree. Fast-forward only -- never merge.

## Step 2: Fetch Team ID

```bash
curl -s -X POST https://api.linear.app/graphql \
  -H "Authorization: $LINEAR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"query": "{ teams { nodes { id name } } }"}'
```

## Step 3: Fetch Todo Tickets

```bash
curl -s -X POST https://api.linear.app/graphql \
  -H "Authorization: $LINEAR_API_KEY" \
  -H "Content-Type: application/json" \
  -d "$(jq -n --arg teamId "$TEAM_ID" \
    '{query: "query($teamId: String!) { issues(filter: { team: { id: { eq: $teamId } }, state: { type: { eq: \"unstarted\" } } }, first: 5, orderBy: priority) { nodes { id identifier title description branchName } } }", variables: {teamId: $teamId}}')"
```

## Step 4: Read Full Ticket Details

For each ticket, fetch the full description and any research comments posted by `/cadence research-backlog`. Read comments to understand prior research before implementing.

## Step 5: Reference Repo Docs

Read in order:
1. `CLAUDE.md` or `AGENTS.md`
2. `tasks/lessons.md` if present
3. Relevant source files for the feature area

## Step 6: Create Feature Branch

```bash
# Use feat/<ID>-<slug> or bug/<ID>-<slug>
# NEVER use the Linear branchName field -- it may contain garbage
git checkout -b feat/<LINEAR-ID>-<short-slug>
```

Slug: lowercase, hyphens, max 5 words from the title.

## Step 7: Implement

- Use `/cadence write-unit-tests` for test coverage
- Stage specific files only -- never `git add -A` or `git add .`
- Never include "Co-Authored-By: Claude" in commits
- Schema changes require both Alembic and Supabase migrations

## Step 8: Push and Open PR

```bash
git push upstream feat/<ID>-<slug>
gh pr create --base dev --title "<title>" --body "..."
```

Always push to `upstream`, never `origin`. Always target `--base dev`.

PR body template:
```
## Summary
<what this implements>

## Linear ticket
<identifier and URL>

## Test plan
<checklist>
```

## Step 9: Move Ticket to In Review

```bash
curl -s -X POST https://api.linear.app/graphql \
  -H "Authorization: $LINEAR_API_KEY" \
  -H "Content-Type: application/json" \
  -d "$(jq -n --arg issueId "$ISSUE_ID" --arg stateId "$IN_REVIEW_STATE_ID" \
    '{query: "mutation($issueId: String!, $stateId: String!) { issueUpdate(id: $issueId, input: { stateId: $stateId }) { success } }", variables: {issueId: $issueId, stateId: $stateId}}')"
```

## Step 10: Report

Output a table: Ticket | Branch | PR URL | Status

## Run Limits and Stopping Rules

- Hard stop at 5 tickets per run
- Stop and ask if any ticket requires a database schema change
- Stop and ask if a ticket's description is empty and no research comments exist

## Common Mistakes

| Mistake | Correct behavior |
|---|---|
| Using Linear branchName field | Construct branch name from ID + title slug |
| `git add -A` | Stage specific files by name |
| Push to origin | Always push to upstream |
| PR targeting main | Always `--base dev` |
| Skipping tests | Use write-unit-tests for all new code |
| "Co-Authored-By: Claude" in commits | Never include this |
