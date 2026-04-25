---
allowed-tools: Bash(git status:*), Bash(git diff:*), Bash(git add:*), Bash(git commit:*), Bash(diff:*)
description: stage and commit changes
---

Review the staged/unstaged changes and produce a conventional commit.

## Step 1 — inspect changes

```bash
git status
git diff          # unstaged
git diff --staged # staged
```

Understand what changed before writing the message.

## Step 2 — group changes into logical commits

Analyse the full diff and identify logical groups. Each group = one commit.
Common split boundaries:

- Production code change vs. its tests → separate commits
- Unrelated modules touched in the same session → separate commits
- Dependency bump vs. code using it → separate commits
- Refactor vs. behaviour change → separate commits

For each group, decide:
1. Which files belong to it
2. What type/scope/message it gets

Present the proposed commit plan to the user before staging anything:

```
Proposed commits:
1. feat(portfolio): add creation endpoint with ownership guard
   → src/main/.../PortfolioController.java, PortfolioService.java, ...

2. test(portfolio): add integration tests for creation flow
   → src/test/.../PortfolioControllerTest.java
```

Then use the `AskUserQuestion` tool to confirm:

```
question: "Execute this commit plan?"
header: "Commit plan"
options:
  - label: "Execute"
    description: "Run the commits as proposed"
  - label: "Adjust"
    description: "I want to change the groupings or messages first"
```

If the user selects "Execute", immediately run the commits without waiting for further input.
If they select "Adjust" or provide changes via "Other", update the plan and prompt again.

Prefer `git add <specific-files>` over `git add .` to avoid committing
unintended files (e.g. `.env`, generated artefacts).

## Step 3 — commit message format

```
<type>(<scope>): <short description>

<body — what changed and why, wrapped at 72 chars>

<optional footer — BREAKING CHANGE, Refs #123, Closes #456>
```

### Subject line

- Imperative mood — "add", "fix", "remove" not "added" or "fixes"
- Max 72 characters
- No full stop at the end
- No `Co-Authored-By` or other trailers

**Types:**

| Type | Use for |
|------|---------|
| `feat` | New functionality |
| `fix` | Bug fix |
| `test` | Adding or updating tests |
| `chore` | Deps, tooling, config, refactors |
| `docs` | Documentation only |
| `refactor` | Code restructure without behaviour change |
| `perf` | Performance improvement |
| `ci` | CI/CD pipeline changes |
| `revert` | Reverting a previous commit |

**Scope:** the module, layer, or domain area affected — e.g. `auth`, `payments`, `portfolio`, `graphql`, `db`.

### Body

Separated from the subject by a blank line. Wrap at 72 characters.
Explain **why** and **what changed at a high level** — not the literal diff
(the diff already shows that).

Include a body when any of these apply:

- The change is non-trivial or non-obvious from the subject
- There is a motivation, ticket, or incident worth recording
- A trade-off, alternative considered, or constraint shaped the approach
- Behaviour changed in a way reviewers or future readers need to know about
- Breaking change, migration step, or follow-up required

Skip the body for trivial commits (typo fix, formatting, version bump) where
the subject is fully self-explanatory.

Suggested structure (use only the parts that apply):

```
<one-paragraph summary of what changed and why>

<optional: details, trade-offs, alternatives considered>

<optional: side effects, migrations, follow-ups>
```

### Footer (optional)

- `BREAKING CHANGE: <description>` for any incompatible change
- `Refs #123`, `Closes #456` to link tickets/issues
- One trailer per line, blank line above the footer block

### Examples

Subject only (trivial change):

```
chore(deps): upgrade Spring Boot to 4.1.2
```

With body (non-trivial change):

```
feat(portfolio): add creation endpoint with ownership guard

New `POST /portfolios` accepts a name + currency and creates a
portfolio owned by the authenticated user. The ownership guard
rejects requests where the principal does not match the body's
owner field, preventing horizontal privilege escalation.

Chose a service-layer guard over an `@PreAuthorize` annotation
because the check needs the request body, which Spring Security
expressions cannot access cleanly.

Refs #482
```

With breaking change:

```
refactor(auth): replace session cookies with JWT bearer tokens

Sessions were stored server-side in Redis, which became the main
bottleneck under load. JWTs move state to the client and let us
scale the auth service horizontally.

BREAKING CHANGE: clients must send `Authorization: Bearer <token>`
instead of relying on the `SID` cookie. The `/auth/session`
endpoint is removed; use `/auth/token` instead.

Closes #611
```

## Step 4 — execute each commit in sequence

After the user confirms, run each commit without waiting for further input.

For subject-only commits:

```bash
git add <files-for-this-group>
git commit -m "<type>(<scope>): <short description>"
```

For commits with a body, pass each paragraph as a separate `-m` (each `-m`
becomes a paragraph separated by a blank line):

```bash
git add <files-for-this-group>
git commit \
  -m "<type>(<scope>): <short description>" \
  -m "<body paragraph 1>" \
  -m "<body paragraph 2 — optional>" \
  -m "Refs #123"
```

Do not embed literal `\n` in a single `-m` string — it will not be
interpreted as a newline.