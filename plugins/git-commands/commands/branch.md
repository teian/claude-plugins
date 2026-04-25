---
allowed-tools: Bash(git checkout:*)
description: create a branch
---

Infer the branch type from the user's description and fill in the branch name.
Show the user what will be executed:

```bash
git checkout main && git pull origin main
git checkout -b <type>/<short-description>
```

Then use the `AskUserQuestion` tool to confirm:

```
question: "Create branch <type>/<short-description>?"
header: "New branch"
options:
  - label: "Execute"
    description: "Switch to main, pull, and create the branch"
  - label: "Adjust"
    description: "Change the branch name or type first"
```

If the user selects "Execute", immediately run the commands.
If they select "Adjust" or provide changes via "Other", update the plan and prompt again.

**Branch types:**

| Type | Lifetime | Use for |
|------|----------|---------|
| `feature/*` | Days | New functionality |
| `fix/*` | Hours–days | Non-urgent bug fixes |
| `hotfix/*` | Hours | Urgent production issues |
| `chore/*` | Hours | Deps, tooling, refactors |
| `release/*` | Permanent | Versioned artefact snapshots |

Naming: `kebab-case`, 2–4 words, no ticket numbers (those go in the PR).

## Rules
- `main` is always deployable — all branches start from a fresh `main`.
- Use `--force-with-lease`, never `--force`.
