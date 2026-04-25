# claude-code-plugins

Custom [Claude Code](https://docs.claude.com/en/docs/claude-code) plugins, distributed as a marketplace.

## Repository layout

```
.
├── .claude-plugin/
│   └── marketplace.json     # marketplace manifest — lists plugins
└── plugins/
    └── git-commands/        # plugin: opinionated git workflow commands
        ├── .claude-plugin/
        │   └── plugin.json
        └── commands/
            ├── git.branch.md
            ├── git.commit.md
            ├── git.prepare-pr.md
            └── git.status.md
```

## Plugins

### `git-commands`

Opinionated slash commands for a clean Git workflow. Each command guides Claude through inspection, confirmation, and execution.

| Command | Purpose |
|---------|---------|
| `/git.branch` | Create a typed branch (`feature/*`, `fix/*`, `hotfix/*`, `chore/*`, `release/*`) from a fresh `main`. |
| `/git.commit` | Group changes into logical conventional commits, confirm the plan, then commit. |
| `/git.prepare-pr` | Rebase onto `main`, force-with-lease push, and generate a ready-to-paste PR title and description. |
| `/git.status` | Surface useful inspection commands for branches, release state, and unbackported fixes. |

Conventions enforced:

- Conventional Commits (`<type>(<scope>): <description>`, imperative mood, ≤72 chars).
- `main` is always deployable; branches start from a fresh `main`.
- Force-pushes use `--force-with-lease`, never `--force`.
- PR merges use **Merge (no fast-forward)** so `git log --first-parent main` shows one entry per PR.

## Installation

Add the marketplace, then install the plugin:

```bash
/plugin marketplace add teian/claude-plugins
/plugin install git-commands@teian-claude-code-plugins
```

Replace `teian/claude-plugins` with the GitHub path to this repository.

## Adding a new plugin

1. Create `plugins/<name>/.claude-plugin/plugin.json` with `name`, `description`, `version`.
2. Add commands under `plugins/<name>/commands/*.md` (each with frontmatter declaring `allowed-tools` and `description`).
3. Register the plugin in [`.claude-plugin/marketplace.json`](.claude-plugin/marketplace.json).
