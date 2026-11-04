# JDW (Jira Driven Worktree)

A local developer & agent CLI that reads Jira issues, automates Git worktree creation, spins up dedicated workspaces in [Herdr](https://herdr.dev), and dispatches coding agents (Claude, Antigravity, Codex) using decoupled methodologies (BMad, SCPE).

---

## Quick Start

### 1. Initialize project and credentials
Run inside your canonical Git repository:
```bash
jdw init
```
This prompts for:
* Project name and GitHub repo (saved in `<git-common-dir>/jdw/config.toml`)
* Default agent (`claude`, `agy`, `codex`) and workflow method (`bmad`, `scpe`)
* Jira base URL, Atlassian email, and API token (saved securely in `~/.jdw/auth.toml` with `chmod 600`)

### 2. Create worktrees & launch agents

```bash
# Standard workflow: creates worktree, Herdr workspace, launches Claude with BMad
jdw fix --issue FT-123

# Choose a different agent (e.g. Antigravity) and methodology (e.g. SCPE):
jdw feat --issue FT-456 --agent agy --method scpe

# Use Codex without pre-submitting a prompt:
jdw fix --issue FT-789 -a codex -m none

# Create worktree only without launching an agent:
jdw feat --issue FT-101 --no-agent

# Without Jira issue:
jdw chore --feature refactor-cache --slug cache-cleanup
```

---

## Documentation

* **[Architecture](docs/architecture.md)**: System overview, layers, data flow, and normalized issue contract (`.jdw/issue.json`).
* **[Workflows & Methodologies](docs/workflows.md)**: Agent configuration (`claude`, `agy`, `codex`) and workflow integration (`bmad`, `scpe`).
* **[Phases & Milestones](docs/phases.md)**: Evolution log from manual worktrees to multi-agent automation.
* **[Handoff Context](HANDOFF.md)**: Historical session notes and roadmap.
