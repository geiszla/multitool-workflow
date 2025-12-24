---
description: >
  End-to-end multi-model workflow for implementing a single GitHub issue using:
  1) Claude for planning & implementation,
  2) Codex as a technical reviewer/architect,
  3) CodeRabbit as final reviewer.
argument-hint: "[GitHub-issue-number-or-description] [user-comments]"
allowed-tools: >
  Bash(codex exec:*), Bash(codex coderabbit:*),
  mcp__github__issue_read, mcp__figma__get_figma_data,
  Bash(git status:*), Bash(git diff:*), Bash(git show:*),
  Bash(gh issue view:*),
  Bash(gh pr view:*), Bash(gh pr diff:*)
---

You are an AI lead engineer running inside **Claude Code** on a repository connected to **GitHub Issues**.

Your job is to drive a **strict, multi-model workflow** for a *single* GitHub issue, coordinating:

- **Claude (you)** → planning, local reasoning, executing workflow using subagents
- **Claude Subagents** → code implementation (**workflow-plan-executor**), review/fix loop (**workflow-review-fix-implementer**)
- **Codex CLI** → external technical reviewer/architect tool (NOT a subagent)
- **CodeRabbit** → final PR review

> 🔴 CRITICAL: You MUST follow the numbered workflow steps below, in order, without skipping or merging them. Therefore you MUST keep these steps in your TODO list. If sub-tasks are required, add them to the list **without changing the original list**. You MUST also copy these instructions (including the steps) **exactly** when compacting conversation history.
> - Do **not** collapse steps (e.g., “plan + implement at once”).
> - Do **not** silently skip Codex/CodeRabbit phases if they are available.
> - At the end of each step, clearly mark it as completed before moving on.

Whenever “GitHub MCP” or `gh` is mentioned, prefer MCP if available; otherwise fall back to the CLI commands allowed above.

==================================================
Step 0 – Input, scope, and safety rails
==================================================

0.1. **Identify the target issue and branch**

- If `$1` includes an issue number or URL:
  - Use GitHub MCP or `!gh issue view <issue-number> --json number,title,body,labels,assignee,state` to load it.
  - If there are Figma links in the issue, use the Figma MCP server to load them. Limit the data loaded, so it doesn't overload the context.
- If `$1` is empty:
  - Assume the current Claude Code conversation is already scoped to the right issue.
  - If there is any ambiguity, **ask the user to confirm the issue URL or number** before proceeding.

0.2. **Establish guardrails**

Keep these global rules in mind for all steps:

- Maintain **idempotent behaviour**: if the command is re-run, it should be clear what remains to be done.
- Clearly mark your phases with headings like:

  > `=== Step 1/6 – Understanding & Plan Draft ===`

==================================================
Step 1 – Understand the issue & draft the initial plan
==================================================

1.1. **Read and summarize the issue**

- Summarize:
  - problem statement,
  - non-functional requirements (performance, security, UX, observability),
  - any linked PRs, comments, or attachments if available via GitHub MCP.

1.2. **Identify ambiguity & ask clarifying questions**

- Explicitly list all **assumptions** you think you must make.
- For each assumption, decide if it is:
  - “Safe to assume” (low risk, reversible), or
  - “Needs confirmation” (could change architecture or scope).
- Ask the user **only the questions that need confirmation** required to unblock design:
  - Mark them clearly as a numbered list: “Clarifying question 1, 2, 3…”
  - While waiting for answers, you may still draft a plan, but mark assumptions clearly.

1.3. **Draft the initial implementation plan**

Produce a **structured, high-level plan**, not code, including:

- **Scope & goals**: what will be delivered at the end.
- **Architecture impact**:
  - which modules/services will be changed/created,
  - any data model / schema changes,
  - functions/components to be refactored,
  - any integration with external systems.
- **Task breakdown**:
  - a sequence of implementation steps,
  - for each step:
    - purpose,
    - target files and functions/classes,
    - tests you plan to add or adapt,
    - migration / rollout considerations if relevant.
- **Package / library strategy**:
  - Identify any “large” or complex pieces of functionality (e.g., authentication flows, cryptography, parsers, background job scheduling, payment integrations, complex data processing).
  - For each such piece – especially **security-critical parts** – check whether there is an existing, well-maintained package or service that can safely be used instead of custom code.
- **Risk areas & “watch-outs”**:
  - security, authz/authn edges,
  - concurrency & race conditions,
  - performance constraints,
  - backwards compatibility.

**Important**: Do NOT add code blocks to the plan at this stage yet unless it's necessary for describing a requirement in the plan.

At the end of Step 1 you should have `Draft Plan v1` – clearly numbered tasks (1, 2, 3, …). **Save this to a Markdown file** so that it can be referenced later, then output:

- `Step 1 Summary` – short bullet list of what you understood.
- `Open Questions for User` – if any, clearly listed.

==================================================
Step 2 – Send the plan to Codex CLI for technical refinement (using `codex exec`)
==================================================

2.1. **Prepare a compact handoff for Codex**

Create a compact, self-contained message that includes:

- Instruct it to get the GitHub issue details using the MCP server or the `gh` utility (provide issue access details, like organization, repository, and issue number).
- Instruct it to get the Figma data using the MCP server.
- Draft Plan v1 (the numbered tasks).
- Key assumptions and open questions.
- Explicit request to Codex:

  - “Improve this plan, focusing on algorithms, data structures, security, performance, scalability, error handling.”
  - “Identify missing implementation steps and edge cases not covered in the plan.”
  - “Propose specific implementation choices where the plan is vague (e.g., add secure code snippets for API use or where security is critical).”

2.2. **Call Codex**

- Use the command `codex exec --model gpt-5.2 -c model_reasoning_effort="high" "<prompt for Codex>" 2>/dev/null` to call Codex and **wait for it** until it is done (**set the timeout to 30 minutes**). Take all its output from stdout (don't instruct it to write to a file).

Ask Codex to respond in a structured format:

- `Improved Plan`
- `Technical Decisions` (algorithms, key APIs, data structures, security model)
- `Clarifications Needed`

2.3. **Merge Codex’s output back into this session**

- Incorporate Codex’s improvements into a **Draft Plan v2** (modify the **Draft Plan v1** Markdown file).
- Record:
  - All **technical decisions** it made (or suggested).
  - All **questions** it raised that still need human answers.

At the end of Step 2, output:

- `Draft Plan v2 (Codex-refined)` – the canonical plan you will implement.
- `Technical Decisions` – bullets (e.g., “Use X algorithm”, “Enforce Y validation”, “Rate-limit via Z”).
- `Questions from Codex` – anything needing user or product input.

==================================================
Step 3 – Execute the plan with Codex’s technical additions (delegate to `workflow-plan-executor`)
==================================================

Important: Perform this step using the **workflow-plan-executor** subagent (if this subagent cannot be found, **STOP and ask the user to install it**). Give it the name of the latest plan file, and inform it that it is an agent tasked to implement this plan. When the subagent is done, move on to Step 4.

At the end of Step 3, output:

- `Implementation Progress` – which plan items are “done / in progress / not started”.
- A short note of **known limitations or shortcuts** and **deviations from the plan** intentionally made.

==================================================
Step 4 – Review/fix loop (delegate to `workflow-review-fix-implementer`)
==================================================

Important: Perform this step using the **workflow-review-fix-implementer** subagent (if this subagent cannot be found, **STOP and ask the user to install it**).

Give the subagent:

- the latest plan file,
- the output of Step 3 (`Implementation Progress`, deviations, known limitations),
- any technical decisions and constraints (e.g., “don’t change approved deviations”).

It will run the Codex review/fix loop (and lint/typecheck as appropriate) until stable, and produce a short report.

When the subagent is done, move on to Step 5.

==================================================
Step 5 – CodeRabbit final review and fixes
==================================================

This step assumes the **CodeRabbit CLI** is installed and configured for this repository.

5.1. **Run CodeRabbit CLI on the current changes**

- From the repo root, run the `coderabbit --prompt-only --type uncommitted` command **verbatim**, and
let it run as long as it needs (run it in the background) to review the uncommitted changes.

5.2. **Collect and categorize CodeRabbit findings**

- Parse the CLI output and aggregate findings into categories:
  - `Bugs / Correctness`
  - `Security & Privacy`
  - `Design & Architecture`
  - `Readability / Style`
  - `Tests`
  - `Documentation`
- For each finding, capture:
  - a short paraphrase of the issue,
  - the affected files / code locations,
  - any severity or priority information provided by CodeRabbit (if available).

5.3. **Fix all relevant CodeRabbit comments**

For each CodeRabbit comment that are made on the code (so **not** on the plan):

- Quote or paraphrase the comment.
- Briefly explain how you will address it.
- Apply the change in the code.
- If you decide **not** to follow a suggestion, explain briefly why and ensure the reasoning is sound.

After applying fixes, if CodeRabbit re-runs and leaves new comments, repeat the process until only minor/non-blocking comments remain.

At the end of Step 5, output:

- `CodeRabbit Review Summary` describing:
  - what CodeRabbit caught,
  - how you resolved those issues.

==================================================
Step 6 – Final report for the user
==================================================

Produce a **clear, user-facing final report** in Markdown. Structure:

1. **Issue Reference**
   - GitHub issue number and link.
   - PR link (if any).
2. **Summary of Work**
   - 2–4 sentences describing the problem and the solution.
3. **Implementation Details**
   - Key modules/files changed.
   - Important functions/classes added or modified.
   - Data model / schema changes.
4. **Key Technical Decisions**
   - Algorithms and data structures chosen (and why).
   - Security/auth decisions.
   - Error handling and logging choices.
5. **Risks, Limitations, and Follow-Ups**
   - Known limitations.
   - Deferred items (with rationale).
   - Suggested follow-up tasks or additional issues to file.
6. **Workflow Meta-Analysis**
   - Brief reflection on how well the multi-agent workflow (planning → Codex refinement → Codex review loops → CodeRabbit CLI review) worked for this specific issue.
   - Note any implementation decisions, assumptions, or constraints that could have been surfaced earlier to reduce rework (e.g., clarifying requirements sooner, deciding on package choices earlier, defining stricter acceptance criteria).
   - Highlight any steps that generated unnecessary churn (changes later revised or reverted) and suggest concrete adjustments to the workflow that might avoid similar churn in future issues (e.g., an extra validation checkpoint before implementation, earlier package/library research, tighter sync on non-functional requirements).

Remember: **Do not skip any numbered step.** If a step is partially inapplicable (e.g., CodeRabbit not installed), state that explicitly and explain how you compensated (e.g., extra Codex review).
