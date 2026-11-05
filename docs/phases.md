# Evolution Phases of Sprout

This document records the developmental milestones and capabilities introduced across the phases of Sprout (initially prototyped as JDW).

---

## Phase 1: Git Worktree Automation
**Commit:** `2211e47 feat(worktree): inicializacao do CLI para criacao automatizada de git worktrees`

- **Goal:** Eliminate manual setup overhead when creating Git worktrees for features and bug fixes.
- **Key Capabilities:**
  - Positional conventional commit types: `feat`, `fix`, `docs`, `refactor`, `perf`, `test`, `chore`, etc.
  - Automatic detection of the canonical Git repository and common directory via `git rev-parse --path-format=absolute --git-common-dir`.
  - Automatic creation of sibling worktree directories.
  - Creation and upstream tracking of `origin/main`.
  - Arguments: `--feature`, `--slug`, `--base`, `--dest`.

---

## Phase 2: Jira Cloud Integration & Scoped Tokens
**Commit:** `162a902 feat(jira): integracao com api do jira, scoped tokens e comando jdw init`

- **Goal:** Connect worktree creation directly to Jira issues without leaking credentials.
- **Key Capabilities:**
  - Automatic resolution of Jira Cloud ID via `https://<site>/_edge/tenant_info`.
  - Support for Atlassian Scoped API tokens routed through the API Gateway (`https://api.atlassian.com/ex/jira/<cloudId>`).
  - Temporary Netrc file lifecycle (created and purged safely in memory/tempfile).
  - Secure credential storage:
    - User secrets: `$HOME/.sprout/auth.toml` (chmod 600, dir chmod 700).
    - Project metadata: `<git-common-dir>/sprout/config.toml` (shared across worktrees, untracked by source control).
  - Interactive onboarding wizard via `sprout init`.
  - Automatic branch slug derived from the issue key: `--issue <KEY>`.

---

## Phase 3: Herdr Workspace, Multi-Agent Dispatch & Methodology Decoupling
**Commit:** `276f663 feat(herdr): integracao com herdr, despacho multi-agente e desacoplamento de metodologias`

- **Goal:** Complete end-to-end automation from issue key to active coding agent session in a dedicated terminal workspace.
- **Key Capabilities:**
  - **Full Issue Extraction:** Fetches `summary`, `description`, `issuetype`, `status`, `priority`, and `assignee`.
  - **Normalized Issue Contract:** Writes `.sprout/issue.json` (machine-readable) and `.sprout/issue.md` (markdown) inside each worktree.
  - **Herdr Workspace Management:** Automatically creates and focuses a dedicated terminal workspace in Herdr (`herdr workspace create --cwd "$target" --label "$label"`).
  - **Multi-Agent Runtime Selection:** Supports choosing which agent to run per session (`-a claude`, `-a agy`, `-a codex`, `-a gemini`).
  - **Methodology Decoupling:** Pluggable prompt templates for BMad (`-m bmad`), SCPE (`-m scpe`), or interactive (`-m none`).
  - **Interactive Terminal Readiness:** Prompts are injected non-blockingly, allowing the developer to continue conversing immediately.

---

## Phase 4: Open-Source Rebranding as Sprout
- Rebranded from JDW to **Sprout** to reflect a tracker-agnostic, developer-flow-first identity.
- Added backwards compatibility aliases and directory fallbacks.
- Documentation and CLI polished for open-source publication.

---

## Phase 5: Smart Agent Router & Telemetry
- **Goal:** Intelligent agent selection based on remaining session budget, cooldown tracking, and live Herdr concurrency.
- **Key Capabilities:**
  - Repository-wide telemetry store in `<git-common-dir>/sprout/agents_status.json`.
  - Subcommands: `sprout agent status`, `sprout agent cooldown`, and `sprout agent reset`.
  - `--agent auto` smart selection balancing Herdr load and cooldown timers.
  - Transparent developer feedback on routing decisions.

---

## Phase 6: Task-Type Model Routing & Warm-Start Session Recycling
- **Goal:** Optimize token economics and eliminate context window degradation across extended agent sessions.
- **Key Capabilities:**
  - **Deterministic Task-Type Model Tiering:** Automatically resolves model complexity (`frontier`, `balanced`, `fast`) based on task type (`feat`, `fix`, `docs`, `chore`, etc.) and maps directly to provider models (`claude`, `agy`, `codex`).
  - **Provider Flag Propagation:** Injects `--model` flags into Herdr agent startup invocations.
  - **Warm-Start Session Recycling (`sprout session recycle`):** Snapshots Git diffs, modified files, and task context to `.sprout/handoff.md`, interrupts the bloated agent process in Herdr, and relaunches a fresh session with a pristine context window (~3k tokens) and warm-start prompt.
  - **Session Lifecycle Inspection (`sprout session status`):** Displays active agent, Herdr pane, worktree mapping, and last handoff timestamp.


