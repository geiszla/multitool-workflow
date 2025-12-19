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

## Install locally (this machine)

1) Add this plugin as a local marketplace:

```bash
claude plugin marketplace add /path/to/multitool-workflow-plugin
```

2) Install the plugin:

```bash
claude plugin install multitool-workflow@multitool-workflow-marketplace
```

If you use Claude Code’s interactive UI, you can also run:

- `/plugin marketplace add /path/to/multitool-workflow-plugin`
- `/plugin install multitool-workflow@multitool-workflow-marketplace`

## Sharing

### Option A: Share as a GitHub repo (recommended)

1) Put `claude-plugins/multitool-workflow-plugin` in its own repository (or subtree).
2) Push it to GitHub (e.g. `your-org/multitool-workflow-plugin`).
3) Other users install it:

```bash
# Add the marketplace
claude plugin marketplace add your-org/multitool-workflow-plugin

# Install the plugin from that marketplace
claude plugin install multitool-workflow@multitool-workflow-marketplace
```

### Option B: Share as a folder/zip

Send the `multitool-workflow-plugin` directory to someone, then they run:

```bash
claude plugin marketplace add /absolute/path/to/multitool-workflow-plugin
claude plugin install multitool-workflow@multitool-workflow-marketplace
```
