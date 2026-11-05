# 🌱 Sprout

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)
[![Git Worktree](https://img.shields.io/badge/Git-Worktree-orange.svg)](https://git-scm.com/docs/git-worktree)
[![Multiplexer: Herdr](https://img.shields.io/badge/Multiplexer-Herdr-6c5ce7.svg)](https://herdr.dev)
[![Agents: Multi--Agent](https://img.shields.io/badge/Agents-Claude%20%7C%20AGY%20%7C%20Codex-success.svg)](docs/workflows.md#supported-agents)

> **One command from issue ticket to active coding agent in a focused Git worktree.**

Starting a task usually means a dozen repetitive, context-draining steps: find the ticket, name the branch, set up a worktree, open a terminal, launch an agent, paste in the ticket context. **Sprout** collapses all of that into one command:

```bash
sprout fix --issue FT-123
```

...and leaves you in an active terminal with the agent already briefed on the ticket, ready to pair.

---

## Quick Start

### Before you start

* **Git** (>= 2.30, with `worktree` support), **Python 3.11+**, **curl** & **jq**.
* A **Jira API Token** *(required for Jira issue lookups)* — see [Getting Your Jira API Token](#-getting-your-jira-api-token) below.
* **[Herdr](https://herdr.dev)** installed and running (`herdr status`) — required for workspace & agent orchestration.

### 1. Install

```bash
git clone https://github.com/your-org/sprout.git
cd sprout
cp bin/sprout ~/.local/bin/sprout
```

### 2. Initialize your project

Run `sprout init` from within your **canonical base Git repository**:

```bash
cd ~/workspace/my-project
sprout init
```

The interactive wizard configures:
* **Project metadata:** Project name and GitHub repository (saved under shared Git metadata at `<git-common-dir>/sprout/config.toml`).
* **Defaults:** Default agent (`claude`, `agy`, `codex`) and workflow method (`bmad`, `scpe`, `none`).
* **Jira credentials:** Site URL, account email, and API token (saved securely in `~/.sprout/auth.toml` with `chmod 600`).

#### 🔑 Getting Your Jira API Token

1. Log into your Atlassian account and navigate to **[Atlassian Security: API Tokens](https://id.atlassian.com/manage-profile/security/api-tokens)**.
2. Click **Create API token**, give it a descriptive label (e.g., `sprout-cli`), and copy the generated token.
3. **Required scopes & permissions:** your account needs **Browse Projects** on the relevant Jira projects, and the token needs the `read:jira-work` scope (or its granular issue-read equivalent) if it's a Scoped API Token.
4. Run `sprout init` and paste the token when prompted (input is hidden).

> [!TIP]
> Need to rotate your token later? Run `sprout --configure-jira` any time to refresh credentials without touching project settings.

### 3. Create your first worktree

```bash
sprout fix --issue FT-123
```

Sprout fetches the issue, creates a `fix/FT-123` worktree tracking `origin/main`, drops a normalized `.sprout/issue.json` + `.sprout/issue.md` into it, opens a Herdr workspace, and launches your agent with the ticket already loaded — ready for you to review and continue pair programming.

---

## Usage

```bash
# Standard workflow: Jira issue + default agent (Claude) + default method (BMad)
sprout fix --issue FT-123

# Choose a different agent (e.g., Antigravity CLI)
sprout feat --issue FT-456 --agent agy

# Combine a specific agent with the SCPE methodology
sprout feat --issue FT-456 -a agy -m scpe

# Launch Codex in interactive mode without an automated prompt
sprout fix --issue FT-789 -a codex -m none

# Create worktree and Herdr workspace without starting an agent
sprout feat --issue FT-101 --no-agent

# Create worktree only without creating a Herdr workspace
sprout feat --issue FT-102 --no-herdr

# Create worktree without an issue key (manual feature name and slug)
sprout chore --feature cleanup_logs --slug prune_old_entries
```

### Command Options Reference

| Option | Short | Description |
| :--- | :--- | :--- |
| `<type>` | — | Positional Conventional Commit type (`feat`, `fix`, `docs`, `refactor`, `perf`, `test`, `build`, `ci`, `chore`, `revert`). |
| `--issue <KEY>` | — | Issue key (e.g. `FT-123`). Fetches details, derives branch slug, and creates `.sprout/issue.json`. |
| `--agent <NAME>` | `-a` | Agent runtime to start in Herdr (`claude`, `agy`, `codex`, `gemini`). |
| `--method <NAME>`| `-m` | Initial methodology prompt template (`bmad`, `scpe`, `none`). |
| `--feature <NAME>`| — | Directory name for the worktree (defaults to issue key; lowercased with hyphens converted to underscores). |
| `--slug <SLUG>` | — | Branch name slug (defaults to the issue key when `--issue` is used, otherwise `--feature`; lowercased with hyphens converted to underscores). |
| `--base <PATH>` | — | Base repository path (defaults to current Git repo, then `BASE_REPO`). |
| `--dest <PATH>` | — | Explicit destination path for the worktree. |
| `--no-agent` | — | Open Herdr workspace but do not start an agent. |
| `--no-herdr` | — | Skip Herdr workspace creation altogether. |
| `--configure-jira`| — | Reconfigure Jira credentials in `~/.sprout/auth.toml`. |

---

## How It Works

```mermaid
flowchart LR
    A["sprout fix --issue FT-123"] --> B["1. Fetch Issue Data\n(Jira / Trackers)"]
    B --> C["2. Create Git Worktree\n(origin/main tracking)"]
    C --> D["3. Generate Normalized Contract\n(.sprout/issue.json & issue.md)"]
    D --> E["4. Open Workspace in Herdr\n(herdr workspace create)"]
    E --> F["5. Dispatch Agent & Prompt\n(Claude / AGY / Codex + BMad / SCPE)"]
    F --> G["6. Developer in Flow\n(Active interactive terminal)"]
```

Sprout is built around pluggable issue trackers (Jira today; Linear/GitHub Issues planned), pluggable methodologies (BMad, SCPE, or none), and multi-agent support via [Herdr](https://herdr.dev) (Claude Code, Antigravity, Codex, Gemini CLI). See the [Architecture Guide](docs/architecture.md) for the full design, including the zero-secrets-in-repo configuration model.

---

## Documentation

* **[Architecture Guide](docs/architecture.md)**: Deep dive into the 3-tier design, configuration/secrets model, data flows, and extensibility.
* **[Workflows & Methodologies](docs/workflows.md)**: Details on configuring agents, prompt templates, and project defaults.
* **[Phases & Evolution](docs/phases.md)**: The engineering milestones that shaped Sprout.

---

## License

MIT © [Marcos Araújo](https://github.com/mraraujo)
