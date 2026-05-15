# Claude Code Explore Agent Research

Sources checked on 2026-05-15:

- Anthropic Claude Code subagents docs: https://code.claude.com/docs/en/sub-agents
- Anthropic Claude Code common workflows: https://code.claude.com/docs/en/tutorials
- Anthropic blog, "How and when to use subagents in Claude Code": https://claude.com/blog/subagents-in-claude-code

Useful design points:

- Subagents run in separate context windows and return only relevant results to the main conversation.
- Claude Code positions Explore agents as fast, read-only code search agents for file discovery, code search, and codebase exploration.
- Anthropic recommends subagents for research-heavy work, high-volume tool output, independent parallel investigations, fresh reviews, and phased workflows.
- A strong signal for delegation is a task that needs about 10 or more files, or about three or more independent pieces of work.
- Effective prompts define scope, request parallel execution only for independent tasks, and specify the desired output format.
- Custom agent routing depends heavily on the description/trigger field, so skill metadata should state the exact situations that should activate the workflow.
- The common workflow docs specifically recommend delegating large codebase exploration so only findings return to the main context.

Implications for this skill:

- The main agent should not perform broad file reads first.
- Explorer prompts should be narrow, read-only, and output-shaped.
- The main handoff artifact should be a concise key-files table with reasons and confidence.
- The main agent should verify only the smallest necessary set of files before editing.
