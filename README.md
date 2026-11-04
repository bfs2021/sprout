# JDW

Jira Driven Worktree: a local CLI that reads Jira issues and creates Git worktrees for implementation work.

The current prototype is in `bin/jdw`. See [HANDOFF.md](HANDOFF.md) for project context, current behavior, configuration, and planned product direction.

No Jira credentials or local project configuration belong in this repository. Per-user Jira credentials are stored in `$HOME/.jdw/auth.toml`; project configuration is kept under the Git common directory of the configured project.
