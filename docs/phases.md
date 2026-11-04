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
  - Automatic generation of branch slugs based on issue summaries: `--issue <KEY>`.

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
