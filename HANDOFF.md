# JDW — context handoff

Date: 2026-11-04

## What this project is

JDW currently means “Jira Driven Worktree.” It began as a personal shell command for creating Git worktrees from Jira issues. The longer-term intent is to make it a distributable tool that can work with multiple project-management systems (Jira first, potentially Linear/GitHub Issues and others) and multiple agent harnesses. Because of that direction, reconsider the product name: “JDW” is Jira-specific even if it remains a useful command alias.

This repository was created as `/home/mraraujo/workspace/jdw` to continue the work. The current installed command remains at `/home/mraraujo/.local/bin/jdw`; `bin/jdw` is a copy of that version for source control. No credentials were copied.

## Current command behavior

Invocation examples:

```bash
jdw init
jdw fix --issue FT-123
jdw feat --issue FT-456 --feature ft_456 --slug loading-progress
jdw docs --feature guide-refresh
```

- The only positional argument for worktree creation is the Conventional Commit type. Supported types: `feat`, `fix`, `doc`, `docs`, `style`, `refactor`, `perf`, `test`, `build`, `ci`, `chore`, `revert`.
- `--issue KEY` fetches the Jira issue summary and derives a kebab-case branch slug. The issue key is the default worktree directory name.
- `--feature` overrides the worktree directory name; `--slug` overrides the branch slug.
- `--base` overrides the source repo. Otherwise precedence is `BASE_REPO`, current Git checkout, then `~/workspace/finance_tool`.
- `--dest` supplies the full worktree destination. By default, a sibling directory named after the feature/issue key is created beside the canonical repo.
- The worktree branch is created from `origin/main` with tracking configured.
- `jdw init` must be run inside the canonical base repo. It detects the base from Git metadata; it does not save the base path.

## Current configuration and secrets

- Jira credentials: `$HOME/.jdw/auth.toml`, TOML table `[jira]` with `base_url`, `email`, and `api_token`.
- Project metadata: `<git-common-dir>/jdw/config.toml`, TOML table `[project]` with `name` and `github_repo`. Storing this under the shared Git metadata makes it available from linked worktrees without adding local settings to the source tree.
- `jdw init` prompts for project name, GitHub repository, Jira site URL, Atlassian account email, and API token. The token is entered with hidden input. Config files are written with mode `600`; the `$HOME/.jdw` directory is mode `700`.
- Never commit or copy actual credentials. The user's real config exists outside this repo and is intentionally not included.

## Jira authentication: scoped API tokens

The user created a scoped API token. A previous direct request to `https://<site>.atlassian.net/rest/api/3/myself` returned 401 because scoped API tokens use the Atlassian API gateway with the site's Cloud ID.

The current CLI now:
1. Reads the site URL from `auth.toml`.
2. Fetches `https://<site>/_edge/tenant_info` to get `cloudId`.
3. Calls `https://api.atlassian.com/ex/jira/<cloudId>/rest/api/3/issue/<KEY>?fields=summary` using Basic auth with account email and scoped API token via a temporary netrc file.
4. Removes the temporary netrc and response files on exit.

The token needs issue-read scope (`read:jira-work` or equivalent granular read scopes), and the Atlassian account needs Browse Projects / issue visibility. Atlassian sources reviewed:
- Scoped token gateway and Cloud ID: https://support.atlassian.com/atlassian-cloud/kb/401-unauthorized-error-when-service-account-accesses-jira-or-confluence-api/
- Get issue endpoint and recommended scope: https://developer.atlassian.com/cloud/jira/platform/rest/v3/api-group-issues/
- Cloud ID discovery: https://support.atlassian.com/jira/kb/retrieve-my-atlassian-sites-cloud-id/

## Work completed in the prior session

- Created the local `jdw` command (renamed from `new-worktree`) in `$HOME/.local/bin`.
- Added named options, conventional commit branch types, optional branch slug, dynamic base selection, destination override, Jira issue lookup, slug generation, and secure local configuration.
- Added `jdw init` to save project/Jira configuration. It detects the canonical repo and stores per-project settings in shared Git metadata rather than saving the repo path.
- Fixed interactive prompts to read from `/dev/tty`; before that, Python read its heredoc as stdin and `jdw init` failed with `EOFError`.
- Added Jira scoped-token support through Atlassian's API gateway and Cloud ID discovery.
- Added clearer HTTP 401/403/404 diagnostics.

## Verification status and known next step

- `jdw --help` was checked after the init work.
- `bash -n` passed after the interactive prompt fix and after the scoped-token routing changes.
- `jdw init` completed before scoped-token routing was added.
- **The scoped-token API request has not yet been validated end-to-end after the gateway change.** The next step is to run this from the Finance Tool canonical repo with an issue the account can browse:

```bash
cd ~/workspace/finance_tool
jdw fix --issue FT-123
```

That command creates the worktree and branch if the Jira lookup succeeds; choose an issue whose target directory and branch do not already exist. If the API still fails, verify the token scopes include issue read access and inspect the new status-specific message. Do not print or share the token.

The installed command and repo copy can diverge until an installer/symlink is created. Compare `$HOME/.local/bin/jdw` and `~/workspace/jdw/bin/jdw` before continuing development.

## Product architecture discussed

Recommendation was to begin with one monorepo, not three repositories, and split only if separate teams/release cadences justify it. Keep the layers independent:

- CLI/core: auth, Git worktrees, normalized issue model, stable JSON output.
- Tracker providers: Jira first, later other trackers such as Linear or GitHub Issues.
- Agent skill(s): orchestration instructions that call the CLI; they should not handle credentials or make raw Jira requests.
- Harness adapters: BMad workflow integration, Codex/Claude/etc. integrations as needed.
- Shared briefing schema/template: tracker-neutral input/output contract.

BMad is better treated as a framework/methodology with its own skills, agents, and workflows, rather than as an agent harness. A BMad workflow can consume the normalized issue JSON and produce a reviewed brief. For broader agent interoperability, expose a stable command such as `jdw issue show --key FT-123 --json`; consider MCP later if multiple harnesses need a common callable interface.

Suggested future repo structure:

```text
bin/jdw
src/jdw/
  core/
  providers/
    jira/
    ...
agent-skills/
  issue-brief/SKILL.md
integrations/
  bmad/
  ...
schemas/
  issue.json
  brief.json
```

The current shell prototype is growing. Consider migrating the implementation to a small Python CLI package before adding several providers, while preserving its command-line UX. Keep credentials outside the repository and project-specific local metadata under Git common metadata (or another clearly documented per-project config location).
