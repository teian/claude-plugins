---
allowed-tools: Bash(git fetch:*), Bash(git branch:*), Bash(git status:*), Bash(git log:*), Bash(git rebase:*), Bash(git push:*)
description: prepare branch for a PR
---

Rebase, push, then generate a ready-to-paste PR title and description.
Does **not** merge, only prepares the branch for a PR.

## Step 1 — rebase and push

Run these commands to get the current branch name and check status:

```bash
git branch --show-current
git fetch origin
git status
git log origin/main..HEAD --oneline
```

If the rebase will have conflicts, stop and help resolve them first.
Do **not** rebase if the PR has open review comments — wait for approval.

Show what will be executed with the real branch name filled in:

```bash
git rebase origin/main
git push --force-with-lease origin <branch>
```

Use the `AskUserQuestion` tool to confirm:

```
question: "Rebase onto main and push <branch>?"
header: "Rebase & push"
options:
  - label: "Execute"
    description: "Run rebase and push — generate PR title and description after"
  - label: "Skip"
    description: "Branch is already up to date — just generate PR title and description"
```

If the user selects "Execute", immediately run the rebase and push, then continue to Step 2.
If they select "Skip", go directly to Step 2.
If an error occurs, diagnose and fix before continuing.

## Step 2 — read the commits

After the user confirms, read the commits that will be in the PR:

```bash
git log origin/main..HEAD --oneline
```
Use the commit messages to fill in the PR title and description.

## Step 3 — PR title

Format: `PR: <type>(<scope>): <short description>`
- `PR:  ` prefix marks merge commits in `git log --first-parent main`
- Max ~72 characters total
- Imperative mood — "add", "fix", "update" not "added" or "fixes"
- No full stop at the end

Examples:
- `PR: feat(payments): add Stripe checkout with idempotency key support`
- `PR: fix(auth): guard against null session in AuthMiddleware`
- `PR: chore(deps): upgrade Elasticsearch client to v9`

## Step 4 — PR description body

Fill in this template from the commits and branch context:

```markdown
## What
<!-- 1–2 sentences: what does this change do? -->

## Why
<!-- Link to ticket / explain the motivation -->

## How to test
1. <!-- step one -->
2. <!-- step two -->
3. <!-- expected outcome -->

## Screenshots / recordings
<!-- Attach before/after if UI is involved — delete section if not -->

## Checklist
- [x] Branch rebased onto latest `main`
- [ ] Tests pass locally
- [ ] No new lint errors
- [ ] Feature flag added if change is incomplete
- [ ] Migrations are backwards-compatible
```

Drop the screenshots section entirely if there are no UI changes.
Pre-tick the rebase checkbox — step 1 already did it.

## Step 5 — output

Present in order:
1. The rebase + push commands (step 1) with real branch name
2. PR title as a copyable line
3. PR description as a copyable markdown block
4. Reminder: set target branch to `main`, merge type **Merge (no fast-forward)**
