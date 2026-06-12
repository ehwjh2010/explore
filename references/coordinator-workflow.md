# Coordinator Workflow

Read when you are the main coordinator agent deciding whether and how to run Explore.

## Coordinator Mode

Use this mode when you are the main agent deciding whether and how to run Explore.

1. State briefly that you are using the Explore workflow and explorer subagents for the reconnaissance pass.
2. Identify independent research slices. Start with business dimensions, risk hypotheses, state paths, permission boundaries, and side effects; use technical layers as supporting boundaries. Let the number of explorer subagents follow the useful, non-overlapping slices; do not impose a fixed count.
3. Build an explicit explorer roster before dispatch: name each slice, its scope, and the question it must answer. This roster defines the completion barrier for the reconnaissance pass.
4. Spawn explorer subagents when this skill applies; the matching task is enough authorization and subagent reconnaissance is required. Give each subagent a clear, complementary, read-only task. Default explorer reasoning effort to `low`; raise to `medium` or `high` only when the task is architecturally complex, ambiguous, high-risk, or has multiple similar code paths that are easy to confuse.
5. While any explorer in the roster is still running, pending, or has an unknown status, do not read code from the target codebase, synthesize final findings, choose implementation files, or edit files in the main conversation. Only coordinate, wait, or work on unrelated non-codebase tasks.
6. If some explorers finish earlier than others, record their outputs as interim notes only. Do not act on partial results unless every other rostered explorer has also reached a terminal result or has been explicitly marked unavailable/failed in the synthesis.
7. After every rostered explorer reaches a terminal result, collect each explorer's output and terminal status: `useful`, `empty/low-value`, `failure`, or `unavailable`.
8. For each started explorer, run the active runtime's supported close, terminate, or equivalent cleanup operation. Record each cleanup status as `closed`, `close failed`, or `close unavailable`. Retry a failed close once when the runtime supports retrying.
9. Only after the terminal-result barrier and cleanup recording are complete, synthesize the result before reading large files yourself.
10. Produce a key-files table, then use it as the reading map for the next step.

## Responsibility Split

- Main coordinator agent: split the task, spawn explorers, wait for summaries, synthesize findings, and perform only the smallest necessary local verification before editing.
- Explorer worker subagents: perform direct read-only search and file inspection within their assigned slice, identify active and inactive paths when possible, and return concise evidence-backed summaries. This direct local exploration is their delegated job, not a fallback.
- No coordinator fallback local pass: if explorer subagents cannot run, stop and report the blocker.
