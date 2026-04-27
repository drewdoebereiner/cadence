# Bulk PR Review

## Overview

Review all open PRs sequentially by dependency layer, using parallel subagents within each layer. Two phases: map then review. Never skip to reviewing.

## Core Principle

Build a complete picture of how PRs relate to each other before reviewing any of them. A review that ignores cross-PR dependencies produces wrong verdicts.

## Phase 1: Map

### Step 1: Fetch All Open PRs and CI Status

```bash
gh pr list --state open --json number,title,headRefName,baseRefName,files,statusCheckRollup
```

Mark any PR where CI is failing as BLOCKED-CI immediately.

### Step 2: Build Dependency Layers

Assign each PR to a layer based on what it depends on:

- **Layer 0** -- Foundation: no dependencies on other open PRs (models, migrations, core interfaces)
- **Layer 1** -- Services that depend on Layer 0 changes
- **Layer 2** -- Features that depend on Layer 1 services
- **Layer 3** -- Integration or cross-cutting changes
- **Layer 4** -- Tests that depend on Layer 3
- **Layer 5** -- Release prep, docs, version bumps

### Step 3: Build Cross-PR Conflict Registry

For any file touched by 2+ open PRs, note:
- Which PRs touch it
- What each PR does to it
- Whether the changes conflict or compose cleanly

Also check for:
- Model field divergence (two PRs change the same field differently)
- Interface gaps (PR A adds a method PR B calls, but A hasn't merged)

## Phase 2: Review

Dispatch one subagent per PR within each layer, all in parallel. Wait for all Layer N verdicts before starting Layer N+1.

Each reviewer subagent receives:
- PR diff (`gh pr diff <number>`)
- The cross-PR conflict registry entries relevant to this PR
- The list of PRs in lower layers it depends on

Verdict options:
- **APPROVE** -- ready to merge
- **APPROVE WITH COMMENTS** -- minor issues, can merge after addressing
- **REQUEST CHANGES** -- must fix before merging
- **BLOCKED-CI** -- CI failing, fix first
- **CLOSE** -- not worth merging as-is

## Decisive Decision Rules

| Situation | Verdict |
|---|---|
| Same file in 2+ PRs with conflicting changes | REQUEST CHANGES on the later one |
| Type mismatch between PRs | REQUEST CHANGES on both, note the conflict |
| Missing method one PR depends on | BLOCKED until dependency merges |
| Omnibus PR (unrelated changes bundled) | REQUEST CHANGES -- ask to split |
| CI failing | BLOCKED-CI |
| Style issues only | APPROVE WITH COMMENTS |
| Safety-critical path changed without tests | REQUEST CHANGES |

## Phase 3: Post Results to GitHub

For each verdict:

```bash
# Approve
gh pr review <number> --approve --body "<verdict body>"

# Request changes
gh pr review <number> --request-changes --body "<verdict body>"

# Comment only
gh pr comment <number> --body "<verdict body>"

# Close
gh pr close <number> --comment "<verdict body>"
```

Verdict body format:
```
**Verdict:** <APPROVE / REQUEST CHANGES / BLOCKED-CI / CLOSE>

**Reasoning:** <1-3 sentences>

**Required changes:**
- <specific item>

**Cross-PR impacts:**
- <any conflicts with other open PRs>

**Notes:**
<anything else>
```

## Final Output

Print:
- Summary counts: approved / changes requested / blocked / closed
- Conflict list with affected PRs
- Recommended merge order (Layer 0 first, then Layer 1, etc.)

## Red Flags -- Stop If You See These

1. A PR that changes a public API without a version bump
2. A migration that drops a column without a backfill
3. Two PRs that both claim to be the "canonical" version of a module
4. A PR that disables tests or skips CI checks
5. A PR with >20 files changed and no clear scope boundary
6. A PR that adds a secret or credential in plaintext

## Common Rationalizations

| Rationalization | Correct behavior |
|---|---|
| "The conflict is minor" | Flag it. Let the author decide. |
| "CI will probably pass after a rebase" | BLOCKED-CI until it actually passes |
| "The tests are just flaky" | Still flag it |
| "I don't want to block this PR" | Your job is accurate verdicts, not approval |
