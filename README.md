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
- GitHub MCP configured (recommended) or `gh` CLI available
- Codex CLI (OpenAI)
- CodeRabbit CLI

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

## Usage examples

The command accepts an issue number or URL (or a short description) followed by optional
free-form comments.

```text
/github-workflow 123
```

```text
/github-workflow 123 "Please preserve API compatibility; add tests; keep changes minimal."
```

```text
/github-workflow https://github.com/geiszla/multitool-workflow/issues/1 "Use Opus; run Codex review; finish with CodeRabbit."
```

```text
/github-workflow "Implement dark mode" "No issue number yet; create a plan first and ask clarifying questions."
```

## Tool setup links

### GitHub MCP (recommended)

It’s recommended to configure GitHub MCP for **both Claude Code and Codex**:

- Claude Code uses it to read issues/PRs and drive `/github-workflow`.
- Codex can use it during the Codex review phase for extra GitHub context.

Setup links:

- Claude Code MCP + GitHub example: https://code.claude.com/docs/en/mcp
- Codex MCP configuration (`mcp_servers`): https://github.com/openai/codex/blob/main/docs/config.md#mcp_servers

### Codex (OpenAI)

- Install Codex CLI: https://github.com/openai/codex#quickstart
- Enable “agent mode” (interactive TUI): run `codex` (docs: https://github.com/openai/codex/blob/main/docs/getting-started.md#cli-usage)

### CodeRabbit

- Install CodeRabbit CLI: https://docs.coderabbit.ai/cli/overview#install-cli
- Claude Code integration guide: https://docs.coderabbit.ai/cli/claude-code-integration
