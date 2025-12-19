# multitool-workflow (Claude Code plugin)

Packages your existing `/github-workflow` command plus the two supporting agents:

- `workflow-plan-executor`
- `workflow-review-fix-implementer`

## What you get

- Slash command: `/github-workflow`
- Subagents:
  - `workflow-plan-executor`
  - `workflow-review-fix-implementer`

Note: The plugin is named `multitool-workflow`, but it currently bundles your existing
`/github-workflow` command unchanged. You can add other non-GitHub workflow commands later
without changing this plugin’s identity.

## Prerequisites

This workflow assumes you have (or can use alternatives):

- GitHub MCP server configured (preferred) or `gh` CLI available
- Optional: Figma MCP server (only if issues link Figma)
- Optional: `codex` CLI (for the Codex review phase)
- Optional: CodeRabbit (for the final review phase)

## Install

### Install from GitHub (recommended)

1) Add the marketplace:

```bash
claude plugin marketplace add geiszla/multitool-workflow
```

2) Install the plugin:

```bash
claude plugin install multitool-workflow@multitool-workflow-marketplace
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
claude plugin install multitool-workflow@multitool-workflow-marketplace
```

## How other users get it (step-by-step)

1) Ensure they have Claude Code installed (the `claude` CLI).
2) Run:

```bash
claude plugin marketplace add geiszla/multitool-workflow
claude plugin install multitool-workflow@multitool-workflow-marketplace
```

3) Start a new Claude Code session and use `/github-workflow`.

## Sharing / contributing

- Repo: https://github.com/geiszla/multitool-workflow
- PRs welcome: add new commands under `commands/` and new agents under `agents/`.
