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
1. **Inefficient Model Sizing (Resource Waste):** Naively launching heavyweight frontier models (e.g. Claude Opus 5.5, Gemini 4 Argon, OpenAI GPT-6 Luna) for every task exhausts session budgets, rate limits, and financial costs on trivial edits like documentation updates, style tweaks, or dependency bumps. Conversely, using lightweight models for complex concurrency bugs or architectural refactors yields poor code quality and hallucinations.
2. **Context Window Saturation (Context Rot):** As coding sessions progress past 60k–100k tokens, attention dilution and lost-in-the-middle degradation cause agents to drift from original instructions, loop repetitively, and slow down dramatically.

This initiative introduces a **zero-dependency, 100% local Control Plane** directly into Sprout:
- **Task-Type Based Model Routing:** Automatically maps the task branch type (`feat`, `fix`, `docs`, `chore`, etc.) to calibrated model tiers (`frontier`, `balanced`, `fast`) and dispatches the exact model flags for `claude`, `agy`, and `codex`.
- **Warm-Start Session Recycling (`sprout session recycle`):** Dynamically terminates a saturated agent session in Herdr, takes a stateful snapshot into `.sprout/handoff.md` (containing Git status, modified file diffs, and task resumption steps), and reboots a clean session in the same worktree with a fresh context window and zero memory amnesia.

---

## Requirements

### R1: Weighted Semantic Complexity Engine
**Given** an issue summary, description, conventional commit type, issue type, and priority
**When** Sprout resolves the task requirements
**Then** it computes a **Complexity Score ($C$)** and **Uncertainty/Risk Score ($U$)** using weighted lexical matrices:
- $C \ge 4 \implies$ `frontier` tier; $1 \le C < 4 \implies$ `balanced` tier; $C \le 0 \implies$ `fast` tier.
- $U \ge 3 \implies$ `high` reasoning effort; $0 \le U < 3 \implies$ `medium` effort; $U < 0 \implies$ `low` effort.
**And** executes in < 2ms with zero external network requests, zero models, and zero API keys.

### R2: Quota & Headroom-First Smart Agent Routing
**Given** multiple coding agents (`claude`, `agy`, `codex`)
**When** resolving the target agent via `--agent auto`
**Then** Sprout evaluates `agents_status.json`:
- Excludes agents with active cooldowns (`cooldown_until > now`).
- Excludes agents exceeding maximum concurrency in Herdr.
- Computes remaining headroom percentage based on sliding window consumption (`tokens_consumed_window / window_limit_est`).
- Selects the freshest candidate with highest available headroom.

### R3: Concrete Model Mapping & Provider Flag Propagation
**Given** the resolved model tier, reasoning effort, and selected agent
**When** spawning the agent inside Herdr
**Then** it translates into concrete provider flags:
- `claude`: `--model opus` (frontier), `sonnet` (balanced), or `haiku` (fast), with `--effort <level>`
- `agy`: `--model gemini-4-argon` (frontier/balanced) or `--model gemini-3.8-flash` (fast) with `--effort <level>` and `--new-project --project <label>`
- `codex`: `--model gpt-6-luna` (frontier), `gpt-5.6-terra` (balanced), or `gpt-reserve` (fast).

### R4: Reliable Clean Shell Process Termination
**Given** interactive agents (e.g. `agy`, `claude`) intercept single `C-c` signals
**When** recycling or terminating an agent in a Herdr pane
**Then** Sprout sends `esc` + double `C-c`, inspects `herdr pane process-info`, and if necessary applies `SIGTERM` / `SIGKILL` directly to the foreground process PID until only the base shell (`shell_pid`) remains
**And** guarantees `herdr agent start` will never fail with `agent_pane_busy`.

### R5: Autonomous Non-LLM Background Watcher Daemon (`sprout session watch`)
**Given** long pair programming sessions may cause context saturation without manual developer intervention
**When** running `sprout session watch --daemon` (or automatically spawned by `sprout <type>`)
**Then** the background process wakes periodically (every 30s) without involving any LLMs:
- Measures transcript file sizes on disk (`transcript.jsonl`) and Herdr scrollback lines.
- If threshold is exceeded (>120KB or >2000 lines), automatically snapshots state to `.sprout/handoff.md`, interrupts the bloated agent, re-spawns a fresh session, and injects the warm-start prompt.
- Prevents rapid re-triggering using a 300s anti-loop cooldown per pane.

---

## Verification Plan

1. Verify running `sprout docs --feature test-docs` selects the `fast` tier (e.g. `haiku`, `gemini-3.8-flash`, or `gpt-reserve`).
2. Verify running `sprout fix --feature test-fix` selects the `frontier` tier (e.g. `opus`, `gemini-4-argon`, or `gpt-6-luna`).
3. Verify running `sprout <type> ... --model custom-model-v1` preserves the explicit model override.
4. Verify running `sprout session status` inside a worktree displays the current session and pane metadata.
5. Verify running `sprout session recycle` captures `.sprout/handoff.md` with Git diffs and successfully relaunches the agent.
