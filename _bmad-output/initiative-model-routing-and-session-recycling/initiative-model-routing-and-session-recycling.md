---
tracker_id: ""
type: initiative
title: "Task-Type Model Routing & Warm-Start Session Recycling"
sources: ["bin/sprout", "docs/architecture.md", "docs/workflows.md"]
companions: []
after: ["initiative-smart-agent-router"]
risk: low
---

# Task-Type Model Routing & Warm-Start Session Recycling

## Description

Working productively with AI coding agents introduces two critical challenges:
1. **Inefficient Model Sizing (Resource Waste):** Naively launching heavyweight frontier models (e.g. Claude 3.7 Sonnet, Gemini Pro, OpenAI o3-mini) for every task exhausts session budgets, rate limits, and financial costs on trivial edits like documentation updates, style tweaks, or dependency bumps. Conversely, using lightweight models for complex concurrency bugs or architectural refactors yields poor code quality and hallucinations.
2. **Context Window Saturation (Context Rot):** As coding sessions progress past 60k–100k tokens, attention dilution and lost-in-the-middle degradation cause agents to drift from original instructions, loop repetitively, and slow down dramatically.

This initiative introduces a **zero-dependency, 100% local Control Plane** directly into Sprout:
- **Task-Type Based Model Routing:** Automatically maps the task branch type (`feat`, `fix`, `docs`, `chore`, etc.) to calibrated model tiers (`frontier`, `balanced`, `fast`) and dispatches the exact model flags for `claude`, `agy`, and `codex`.
- **Warm-Start Session Recycling (`sprout session recycle`):** Dynamically terminates a saturated agent session in Herdr, takes a stateful snapshot into `.sprout/handoff.md` (containing Git status, modified file diffs, and task resumption steps), and reboots a clean session in the same worktree with a fresh context window and zero memory amnesia.

---

## Requirements

### R1: Deterministic Task-Based Model Tiering
**Given** a task branch type (e.g. `feat`, `fix`, `docs`, `chore`)
**When** Sprout resolves model parameters
**Then** it classifies the task into a model tier:
- `feat`, `fix`, `refactor`, `perf`, `spike` -> `frontier` (deep reasoning)
- `test`, `build` -> `balanced` (standard implementation)
- `doc`, `docs`, `chore`, `style`, `ci`, `revert` -> `fast` (lightweight / cost-effective)
**And** executes locally with zero external network requests or API keys.

### R2: Concrete Model Mapping Matrix & Custom Configuration
**Given** the resolved model tier and selected agent (`claude`, `agy`, `codex`)
**When** preparing launch arguments
**Then** Sprout maps to the concrete provider model:
- `claude`: `frontier`/`balanced` -> `claude-3-7-sonnet-latest`, `fast` -> `claude-3-5-haiku-latest`
- `agy`: `frontier` -> `pro`, `balanced`/`fast` -> `flash`
- `codex`: `frontier` -> `o3-mini`, `balanced` -> `gpt-4o`, `fast` -> `gpt-4o-mini`
**And** allows customization via `<git-common-dir>/sprout/config.toml` under `[models]`.
**And** honors explicit CLI overrides via `--model <custom-model>`.

### R3: Herdr Model Propagation
**Given** Sprout initializes a Herdr workspace and pane
**When** executing `herdr agent start`
**Then** it injects the resolved model parameter into the agent CLI invocation:
- `claude`: `-- --model <model>`
- `agy`: `-- --new-project --project <label> --model <model>`
- `codex`: `-- --model <model>`.

### R4: Warm-Start Session Recycling (`sprout session recycle`)
**Given** an active agent session is saturated or transitioning between major implementation phases
**When** the developer runs `sprout session recycle` inside the worktree (or with `--pane <pane_id>`)
**Then** Sprout:
1. Generates an immutable snapshot at `.sprout/handoff.md` with current Git diff stats, modified files, Jira context, and resumption tasks.
2. Gracefully interrupts the bloated agent process in Herdr (`C-c`).
3. Re-spawns a fresh agent session in the same pane with a clean context window.
4. Automatically injects the warm-start prompt directing the agent to read `.sprout/handoff.md` and resume without context bloat.

### R5: Session Status & Telemetry Inspection (`sprout session status`)
**Given** a developer wants to inspect the state of active sessions and handoff status
**When** running `sprout session status`
**Then** Sprout prints the active agent, pane, working directory, resolved model tier, and last handoff timestamp.

---

## Verification Plan

1. Verify running `sprout docs --feature test-docs` selects the `fast` tier (e.g. `claude-3-5-haiku-latest` or `flash`).
2. Verify running `sprout fix --feature test-fix` selects the `frontier` tier (e.g. `claude-3-7-sonnet-latest` or `pro`).
3. Verify running `sprout <type> ... --model custom-model-v1` preserves the explicit model override.
4. Verify running `sprout session status` inside a worktree displays the current session and pane metadata.
5. Verify running `sprout session recycle` captures `.sprout/handoff.md` with Git diffs and successfully relaunches the agent.
