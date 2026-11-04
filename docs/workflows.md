# Workflows and Methodologies

Sprout is designed to decouple issue tracking from development methodologies and agent runtimes. This document describes how workflows operate and how to use or extend them.

## Supported Agents

Any agent supported by Herdr can be passed via `--agent` (or `-a`):

| Agent | CLI Flag | Canonical Command | Description |
| :--- | :--- | :--- | :--- |
| **Claude Code** | `-a claude` | `claude` | Anthropic's interactive agentic CLI |
| **Antigravity** | `-a agy` | `agy` | Google Deepmind's agentic CLI |
| **Codex** | `-a codex` | `codex` | OpenAI / custom Codex CLI |
| **Gemini** | `-a gemini` | `gemini` | Gemini CLI integration |

If omitted, the default is configured in `<git-common-dir>/sprout/config.toml` or falls back to `claude`.

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
Spec-Compiled Product Engineering focuses on executable specs, vertical slices, and rules/scenarios.

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
agent = "claude"     # Default agent: claude, agy, or codex
method = "bmad"      # Default workflow: bmad, scpe, or none
```
