---
tracker_id: ""
type: initiative
title: "Smart Agent Router: Telemetry, Cooldown Tracking & Concurrency-Aware Dispatch"
sources: ["bin/sprout", "docs/architecture.md", "docs/workflows.md"]
companions: []
after: []
risk: low
---

# Smart Agent Router: Telemetry, Cooldown Tracking & Concurrency-Aware Dispatch

## Description

Developers working with multiple generative coding agents (Anthropic Claude Code, Google Antigravity `agy`, OpenAI Codex) frequently hit provider rate limits, sliding window quota caps (e.g., Claude's 5-hour window), or local concurrency saturation. When spinning up a new Git worktree for an issue, manually checking which agent has remaining session budget or lowest concurrency introduces friction and causes failed/stalled sessions.

This initiative introduces a **Smart Agent Router** directly into Sprout. By leveraging Git's common administrative directory (`--git-common-dir`) to persist an untracked, shared `agents_status.json` and querying Herdr's live agent session telemetry, Sprout dynamically routes new task worktrees to the agent with the highest availability and remaining session time.

## Outcome

When a developer runs `sprout fix --issue FT-123` with `--agent auto` (now the smart default), Sprout automatically:
1. Evaluates local agent availability against recorded cooldown windows and quotas.
2. Cross-references live active agent processes in Herdr.
3. Selects the optimal agent (e.g., routing to `agy` when Claude is in cooldown).
4. Provides transparent terminal feedback on why the routing decision was made, while always respecting explicit CLI overrides (`-a claude`).
5. Exposes CLI commands (`sprout agent status`, `sprout agent cooldown`, `sprout agent reset`) for fast developer inspection and cooldown management.

## Requirements

### R1: Centralized, Repository-Wide Telemetry Store
**Given** Sprout is executed from any worktree derived from the canonical repository
**When** Sprout checks or updates agent status
**Then** it reads/writes `<git-common-dir>/sprout/agents_status.json` with permissions `0600`
**And** the file remains completely outside the source code tree (zero Git pollution, no leaks).

### R2: Cooldown & Rate Limit Awareness
**Given** an agent hits a rate limit or a 5-hour usage window
**When** the developer or automated monitor sets a cooldown (e.g. `sprout agent cooldown claude --for 2h`)
**Then** Sprout marks the agent unavailable until the expiration timestamp
**And** subsequent `sprout` runs with `--agent auto` bypass this agent and select the next available candidate.

### R3: Real-Time Concurrency Telemetry via Herdr
**Given** multiple agents may already be executing in background Herdr workspaces
**When** Sprout evaluates eligible candidates
**Then** it queries `herdr agent list` (or `herdr pane list`) to count live running instances per agent
**And** favors the candidate with the lowest concurrent load.

### R4: Explicit CLI Overrides
**Given** the developer passes an explicit `--agent` flag (e.g., `-a claude`)
**When** Sprout dispatches the workspace
**Then** it respects the developer's explicit selection unconditionally, regardless of cooldown or concurrency heuristics.

### R5: Management Subcommands
**Given** the developer wants to inspect or adjust agent telemetry
**When** running `sprout agent status`, `sprout agent cooldown <name>`, or `sprout agent reset <name>`
**Then** Sprout formats a human-readable table of statuses, cooldowns, and active sessions, or updates the telemetry store accordingly.

## Verification Plan

1. Verify `sprout agent status` outputs current agent pool, cooldown status, and active Herdr counts.
2. Verify `sprout agent cooldown claude --for 1h` sets `cooldown_until` correctly.
3. Verify running `sprout <type> ... --agent auto` selects `agy` or `codex` when `claude` is in cooldown.
4. Verify explicit override `sprout <type> ... -a claude` proceeds with `claude` despite cooldown.
5. Verify `sprout agent reset claude` clears the cooldown.
