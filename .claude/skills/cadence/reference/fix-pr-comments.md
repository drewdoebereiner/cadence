# Fix PR Comments

## Overview

For each open PR with unresolved review comments or CHANGES_REQUESTED: check out the branch, fix every issue, push, reply to each comment thread with the fixing commit SHA. One PR at a time.

## Pre-flight: Clean Working Tree

```bash
git status  # fail if dirty
git fetch origin
```

Do not proceed on a dirty working tree.

## Step 1: Find PRs with Pending Feedback

```bash
gh pr list --state open --json number,title,reviewDecision,headRefName
```

Filter for:
- `reviewDecision == "CHANGES_REQUESTED"`
- PRs with unresolved inline review comments

Also check for CodeRabbit comments: look for the AI Agents `<details>` block in review bodies.

## Step 2: For Each PR -- Sync Branch

```bash
git checkout <branch>
git fetch origin
git merge --ff-only origin/<branch>
```

Fast-forward only. If the branch has diverged, skip it and note it in the report.

## Step 3: Fetch All Review Comments

```bash
gh pr view <number> --json reviews,comments
gh api repos/{owner}/{repo}/pulls/<number>/comments
```

Collect:
- Inline comments (file + line + body)
- Review-level comments (body of each CHANGES_REQUESTED review)
- Issue-level comments

For CodeRabbit reviews: extract the actionable items from the AI Agents `<details>` block, not the summary.

## Step 4: Read Affected Files Before Touching Anything

For each comment, read the current state of the referenced file and line. Confirm the issue still exists before writing a fix. If it's already resolved, mark it as skip.

## Step 5: Fix All Issues

- Make the minimal change that resolves the comment
- Follow `CLAUDE.md` conventions
- Stage specific files only -- never `git add -A`
- One commit per logical fix group, or one commit for all fixes if they're small
- Never include "Co-Authored-By: Claude" in commits

## Step 6: Push

```bash
git push upstream <branch>
```

Always push to `upstream`, never `origin`.

## Step 7: Reply to Each Comment Thread

For each resolved comment, reply with the commit SHA:

```bash
gh api repos/{owner}/{repo}/pulls/<number>/comments/<comment-id>/replies \
  -f body="Fixed in <commit-sha>."
```

Resolve threads via GraphQL if the API supports it. If not, reply to confirm.

## Step 8: Report

Output a table: PR | Branch | Issues fixed | Issues skipped | Commit | Status

## Stopping Rules

- Stop if the branch has merge conflicts -- report and skip
- Stop if a fix would require a breaking API change -- ask the user
- Stop if the comment is ambiguous and could mean multiple things -- ask the user

## Common Mistakes

| Mistake | Correct behavior |
|---|---|
| Fixing without reading the file first | Always read current state before writing |
| Pushing to origin | Always push to upstream |
| `git add -A` | Stage specific files only |
| Skipping reply to comment thread | Reply to every thread with commit SHA |
| One giant commit for all PRs | Commit per PR, sometimes per fix group |
