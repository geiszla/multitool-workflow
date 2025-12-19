# multitool-workflow (Claude Code plugin)

This repo is a Claude Code plugin that provides:

- Slash command: `/github-workflow`
- Subagents:
  - `workflow-plan-executor`
  - `workflow-review-fix-implementer`

Currently only supports GitHub-based workflows, but will be extended to other workflows over time.

## Workflow overview

`/github-workflow` runs a strict, multi-tool workflow for a single issue:

1. **Step 0 — Input & guardrails**: Identify the issue/branch; load issue context via GitHub MCP (or `gh`). If the issue links Figma, load relevant Figma data.
2. **Step 1 — Draft plan**: Summarize the issue, ask clarifying questions, and produce a concrete implementation plan.
3. **Step 2 — Codex plan refinement**: Use Codex to stress-test and improve the plan (algorithms, edge cases, security/perf), then save the refined plan to a Markdown file.
4. **Step 3 — Implementation**: Delegate to the `workflow-plan-executor` subagent to implement the refined plan step-by-step and report progress/deviations.
5. **Step 4 — Review/fix loop**: Delegate to the `workflow-review-fix-implementer` subagent to run a Codex review + iterate fixes and validation until stable.
6. **Step 5 — CodeRabbit final review**: Run CodeRabbit CLI against uncommitted changes and apply fixes for findings.
7. **Step 6 — Final report**: Produce a user-facing summary (changes, key decisions, risks/follow-ups) plus a short workflow meta-analysis.

The workflow is designed to be rerunnable and to keep the phases separate (plan → implement → review → final review).

## Prerequisites

- Claude Code installed (`claude` CLI)
- CodeRabbit [account](https://app.coderabbit.ai/login) (free plan)

## Tool setup

These tools are required to use the plugin. **Without installing these tools, the plugin will not work properly.**

#### GitHub MCP or `gh` CLI

- MCP server (recommended): https://github.com/mcp/github/github-mcp-server
- CLI: https://cli.github.com/

It’s recommended to configure the GitHub MCP server for **both Claude Code and Codex**:

- Claude Code uses it to read issues/PRs and drive `/github-workflow`.
- Codex uses it during the plan refinement and review phases for GitHub issue context.

#### OpenAI Codex

- Install and set up Codex CLI: https://github.com/openai/codex#quickstart
- Enable “agent mode” so that Codex can work autonomously: run `codex` in a terminal, then `/approvals`, then select `Agent (current)`.

#### CodeRabbit

- Install CodeRabbit for Claude Code: https://docs.coderabbit.ai/cli/claude-code-integration

## Install

1. Add the marketplace:

   ```bash
   claude plugin marketplace add geiszla/multitool-workflow
   ```

2. Install the plugin:

   ```bash
   claude plugin install multitool-workflow@multitool-marketplace
   ```

3. Start a new Claude Code session (or restart), then run:

   ```bash
   /github-workflow
   ```

## Usage examples

This workflow works best with the **Opus** model.

The command accepts an **URL** or **issue number** followed by optional free-form comments.

```text
/github-workflow https://github.com/geiszla/multitool-workflow/issues/1
```

```text
/github-workflow 485 "Only complete points 1-3, others will be implemented in a future issue."
```

### Usage with other workflows

Although the plugin includes a full workflow, it can be modified to work in combination with other workflows (e.g., brainstorming with [Superpowers](https://github.com/obra/superpowers), then implementation with *multitool* reviews). To do so, just tell the model which steps to skip.

```text
/github-workflow 123 "Planning step is completed, the plan is at <filename>. Start from the implementation step."
```

## Tips

Here are some things to look out for to use this workflow most effectively:

1. **Check the GitHub issue to be implemented**. It should include a high-level description of everything you want to be implemented, but not more (e.g., don't include any implementation details you are not sure about; it's better to let the model figure them out, then you can correct it during the plan review phase).
   - Be precise about the important details (e.g., "client specified that they would like the image to be on the right side of the screen aligned with and having the same width as the image above it"), but let it decide on the parts that are open to interpretation (e.g., how to wrap the text around the images; you automatically get a second opinion this way).
   - Add references of documentation you already have or know about, otherwise instruct it to search for them on the web or consult with an MCP server.
   - Either remove any unnecessary information or instruct the agent to ignore it.
2. The agent will only ask you about **questions that are not safe to assume an answer for** (again, so that your approach in mind doesn't influence its second opinion), then it will create a **first draft of the plan**. At this point you should **carefully check if it made any wrong assumptions** or if you know of a better approach (feel free to discuss it with the agent instead of instructing it; e.g., "what is the downside of doing it this other way instead?"). It's **important to get the high-level plan right** at this stage — as all the next steps will depend on it — but don't go into any more details than it already gave you (there is the next step for that).
3. After the high-level plan is mostly correct, the agent will give it to Codex for refinement. Codex will **research and add the details** (like API usage examples, security best practices, performance considerations, etc.). This might take 10-20 minutes, so grab a coffee while it's running. It's also important to check the technical decisions made here before the plan is finalized, as this final plan will be considered the **ground truth** for the implementation phase (however, you can still modify it if something comes up during implementation and the modifications will be carried on to all subsequent steps).
4. This workflow is designed so that its **implementation phase is as automatic as possible**. Here you should ideally only have to approve the changes (or ask for minor differences). Don't worry about checking the code very carefully, because the agent will run the review/fix cycle after. **Just check if it's working in the right direction** and not making any obvious mistakes.
5. The **review/fix cycle** should also run automatically, so just **make sure it doesn't make unnecessary fixes**.
6. When it's done, the agent will write the review report to a file and **ask you to review the changes made**. This is the point where you should address all remaining issues with the implementation that the reviewer didn't catch, and implement some of the changes suggested by the reviewer but not deemed important enough to fix by the agent. You might also want to test the implementation at this point and **make any changes for issues that only come up during testing**. After this, a final review is done so that these changes don't introduce any new issues.
7. Finally, the agent will write the **final report** to a file. You might want to use the overall report for reference for further work on related issues. You also find a **meta-analysis** at the end of this report, which you can use to improve your use of the workflow (e.g., by focusing on certain aspects of the plan that if done right might enable more effective implementation later).

The goal of this workflow is to exploit the strengths of different models and to improve as these models used here improve. Eventually, as models get better, more and more of the above-mentioned stages should be completely automated drastically speeding up the workflow and reducing the amount of manual work required.
