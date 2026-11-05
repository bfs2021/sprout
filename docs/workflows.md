# Workflows and Methodologies

Sprout is designed to decouple issue tracking from development methodologies and agent runtimes. This document describes how workflows operate and how to use or extend them.

## Supported Agents

Any agent supported by Herdr can be passed via `--agent` (or `-a`):

| Agent | CLI Flag | Canonical Command | Description |
| :--- | :--- | :--- | :--- |
| **Auto (Smart Router)** | `-a auto` | *Dynamic* | Balances load across available agents based on cooldowns & Herdr sessions |
| **Claude Code** | `-a claude` | `claude` | Anthropic's interactive agentic CLI |
| **Antigravity** | `-a agy` | `agy` | Google Deepmind's agentic CLI |
| **Codex** | `-a codex` | `codex` | OpenAI / custom Codex CLI |
| **Gemini** | `-a gemini` | `gemini` | Gemini CLI integration |

If omitted, the default is configured in `<git-common-dir>/sprout/config.toml` or defaults to `auto`.

### Smart Agent Router & Telemetry

Sprout includes a telemetry store at `<git-common-dir>/sprout/agents_status.json` that tracks cooldowns, reasons, and priorities:

```bash
# View live telemetry
sprout agent status

# Place an agent in cooldown (e.g., when reaching a 5-hour window or token limit)
sprout agent cooldown claude --for 2h --reason "5h token limit reached"

# Reset cooldown
sprout agent reset claude
```

**Selection Algorithm:**
1. **Cooldown check:** Excludes agents whose `cooldown_until` is in the future. Expired cooldowns automatically self-heal.
2. **Concurrency balancing:** Queries Herdr's live agent sessions via socket and picks the candidate with the fewest active sessions.
3. **Priority tie-breaking:** Resolves ties using the agent priority hierarchy (e.g., Claude > AGY > Codex).
4. **Graceful degradation:** If all agents are in cooldown, falls back to the highest priority candidate with a clear warning.


---

## Supported Methodologies

The `--method` (or `-m`) flag defines what initial prompt is injected into the agent when the Herdr workspace is created.

### 1. BMad Method (`-m bmad`) - Default
The BMad method uses an orchestrator skill (`bmad`) that guides analysis, product briefs, and implementations.

**Prompt injected:**
> *"BMad, analise a feature/issue <KEY> (<SUMMARY>). A descrição e os detalhes do Jira estão salvos em .sprout/issue.json e .sprout/issue.md. Inicie a análise inicial para elaborarmos o briefing da implementação."*

**What happens next:**
- The agent activates the `bmad` skill.
- Reads `.sprout/issue.json` to extract requirements, acceptance criteria, and edge cases.
- Begins drafting the initiative/briefing document (under `_bmad-output/`) and waits for your confirmation to continue.

### 2. SCPE Method (`-m scpe`)
[Spec-Compiled Product Engineering (SCPE)](https://github.com/m4rc0s/scpe) focuses on executable specs, vertical slices, and rules/scenarios.

**Prompt injected:**
> *"SCPE: analise a issue <KEY> (<SUMMARY>) a partir de .sprout/issue.json. Inicie a especificação da feature de acordo com o método SCPE."*

**What happens next:**
- The agent reads `.sprout/issue.json`.
- Formulates rules (`R#`) and Given/When/Then scenarios (`S#`).
- Outlines vertical slices for implementation in the workspace.

### 3. None / Interactive (`-m none`)
No initial prompt is sent. The agent opens cleanly in the worktree directory ready for you to type whatever you want.

---

## Configuration Defaults

You can configure project-wide defaults by adding a `[defaults]` table in `<git-common-dir>/sprout/config.toml`:

```toml
[project]
name = "Finance Tool"
github_repo = "org/finance-tool"

[defaults]
agent = "auto"       # Default agent: auto (Smart Router), claude, agy, or codex
method = "bmad"      # Default workflow: bmad, scpe, or none

[models.tier_map]
feat = "frontier"
fix = "frontier"
docs = "fast"
chore = "fast"

[models.agents.claude]
frontier = "claude-3-7-sonnet-latest"
fast = "claude-3-5-haiku-latest"
```

---

## Session Recycling & Context Compaction

### The Context Window Problem
During prolonged agent sessions, the cumulative transcript can easily exceed 80,000 tokens. This introduces:
1. **Instruction Amnesia / Drift:** The agent begins to ignore rules defined in the initial briefing.
2. **Execution Latency:** Every turn requires ingesting the bloated context window.
3. **Escalating Costs:** Input token costs compound on every turn.

### The Solution: Warm-Start Handoff
Run inside the worktree at any time:

```bash
# Check current session context and Herdr pane status
sprout session status

# Snapshot state to .sprout/handoff.md and reboot agent with clean context
sprout session recycle
```

When invoked:
1. Sprout snapshots Git diff statistics, uncommitted files, recent commit history, and task requirements to `.sprout/handoff.md`.
2. It sends an interrupt (`C-c`) to the active Herdr pane.
3. It restarts the agent process cleanly with a pristine context window (~3k tokens).
4. It submits an automatic warm-start prompt instructing the agent to resume from `.sprout/handoff.md`.

