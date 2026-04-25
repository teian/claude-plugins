---
allowed-tools: Bash(git branch:*), Bash(git log:*)
description: inspect branch and release state
---

Surface the relevant commands based on what the user needs to know.

```bash
# All open branches
git branch -a

# Main history — one entry per PR
git log --oneline --first-parent main

# Commits on main not yet in a release branch
git log release/v<X.Y>..main --oneline --first-parent

# Is a specific commit in a release branch?
git branch --contains <SHA>

# Unbackported fix/security commits
git log release/v<X.Y>..origin/main --oneline --first-parent \
  | grep -E "^[a-f0-9]+ (fix|security|hotfix|revert)"
```
