---
name: explore
description: >-
  Use when a coding task requires broad read-only codebase reconnaissance before
  implementation, including unfamiliar repositories, architecture questions,
  feature tracing, bug investigations, refactors, migrations, reviews,
  multi-area changes, large or dense files, deep call chains, or high-risk
  domain workflows. When this skill applies or the user invokes $explore, the
  coordinator must spawn read-only explorer subagents before reading target code
  or editing files, wait for every explorer terminal result, explicitly clean up
  started subagents when supported, and report blockage if subagents cannot be
  spawned. Explorer subagents must not invoke Explore again or spawn subagents.
---

# Explore

## Core Rule

Highest-priority role rule: if you are already an explorer subagent spawned by the Explore skill, do not invoke Explore again. Perform the assigned read-only search and file inspection directly, then return your concise report. Read `references/explorer-worker-workflow.md` for worker instructions.

For the main coordinator agent, skill applicability is the authorization signal. When the task matches this skill's applicability conditions, or when the user invokes `$explore`, explorer subagent reconnaissance is mandatory. The user does not need to also say `subagents`, `delegate`, `parallel agents`, or any other authorization phrase.

There is no local reconnaissance fallback for the main coordinator agent. If explorer subagents cannot be spawned, stop the Explore workflow and report that the task is blocked by subagent unavailability. Do not replace explorer work with local `rg`, file reads, or structure scans.

The coordinator must treat spawned explorers as an all-subagents barrier. Once a set of explorers has been launched, do not read target code, synthesize a final reading map, make implementation decisions, or edit files until every launched explorer has returned a terminal result: useful report, empty/low-value report, explicit failure, or explicit unavailability.

After every launched explorer reaches a terminal result, the coordinator must explicitly clean up the spawned subagents before synthesis, local target-code reads, implementation decisions, or edits. Record a cleanup status for each explorer: `closed`, `close failed`, or `close unavailable`.

Keep the main context focused on coordination, decisions, implementation, and verification; let explorer subagents absorb verbose search results and file reads.

## Applicability

Use this skill when any of these signals appear:

- The task is in an unfamiliar repository or subsystem.
- The task likely needs reading about 10 or more files.
- The task only touches a few files, but those files are large, dense, or central to core business behavior.
- The task spans multiple independent areas, such as API, database, UI, tests, docs, or build tooling.
- The user asks for architecture understanding, feature tracing, refactoring, migration, bug root-cause analysis, or broad review.
- You need a fresh, read-only pass before editing.

Complexity indicators that should push toward `$explore` even with a small file count:

- A file is around 500+ lines, especially a service, controller, engine, workflow, or orchestration module.
- One file mixes entry points, orchestration, permissions, data assembly, state transitions, or side effects.
- The task requires tracing several symbols, branches, call chains, or side-effect paths inside a small set of files.
- Misidentifying primary, legacy, experimental, generated, or unused code paths would create meaningful implementation risk.

When choosing whether this skill applies, skip it for a tiny targeted edit where the relevant file and change are already known, such as a typo, one-line constant update, or simple style fix. If the task meets the Explore conditions or the user explicitly invokes `$explore`, subagent reconnaissance is required before the main coordinator agent reads target code or edits files.

## Progressive References

Read only the files needed for the current role and step:

- `references/coordinator-workflow.md`: read when you are the main coordinator deciding how to run Explore, spawn explorers, wait at the barrier, clean up subagents, and synthesize the initial reading map.
- `references/explorer-worker-workflow.md`: read when the prompt says you are an explorer subagent spawned by Explore.
- `references/prompts-and-slicing.md`: read when building explorer prompts, choosing research slices, or needing concrete slicing examples. See also `examples/explorer-prompt.md`.
- `references/synthesis-and-error-handling.md`: read before producing the required synthesis, handling failed or low-value explorers, resolving conflicts, or reporting blocked states. See also `examples/synthesized-output.md`.
- `references/claude-explore-agent-research.md`: read only when updating this skill or explaining its design.

## Design Notes

This workflow adapts Claude Code's Explore Agent pattern to Codex: isolate high-volume code search in separate contexts, return only relevant summaries, and make the handoff concrete through a key-files table.
