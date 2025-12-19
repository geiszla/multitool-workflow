# multitool-workflow (Claude Code plugin)

This repo is a Claude Code plugin that provides:

- Slash command: `/github-workflow`
- Subagents:
  - `workflow-plan-executor`
  - `workflow-review-fix-implementer`

The plugin is named `multitool-workflow` so it can grow beyond GitHub-based workflows over time,
while keeping backwards compatibility.

## Recommended model

This workflow works best with the **Opus** model (higher reliability for multi-step planning +
review/fix loops).

## Prerequisites

- Claude Code installed (`claude` CLI)
- For GitHub issue workflows: GitHub MCP configured (preferred) or `gh` CLI available
- Optional: Codex CLI (OpenAI) for the Codex review phase
- Optional: CodeRabbit CLI for final review

## Install

### Install from GitHub (recommended)

1) Add the marketplace:

```bash
claude plugin marketplace add geiszla/multitool-workflow
```

2) Install the plugin:

```bash
claude plugin install multitool-workflow@multitool-marketplace
```

3) Start a new Claude Code session (or restart), then run:

- `/github-workflow`

### Install from a local clone (for development)

1) Clone:

```bash
git clone https://github.com/geiszla/multitool-workflow.git
cd multitool-workflow
```

2) Add your clone as a marketplace:

```bash
claude plugin marketplace add "$(pwd)"
```

3) Install the plugin:

```bash
claude plugin install multitool-workflow@multitool-marketplace
```

## Optional tool setup links

### Codex (OpenAI)

- Install Codex CLI: https://github.com/openai/codex#quickstart
- Enable “agent mode” (interactive TUI): run `codex` (docs: https://github.com/openai/codex/blob/main/docs/getting-started.md#cli-usage)

### CodeRabbit

- Install CodeRabbit CLI: https://docs.coderabbit.ai/cli/overview#install-cli
- Claude Code integration guide: https://docs.coderabbit.ai/cli/claude-code-integration
