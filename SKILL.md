---
name: explore
description: Use when a coding task requires broad codebase reconnaissance before implementation, especially unfamiliar repositories, architecture questions, feature tracing, bug investigations, refactors, migrations, reviews, or changes that may require reading roughly 10+ files or searching multiple independent areas. Delegates read-only exploration to explorer subagents first, keeps the main conversation lean, prevents the main agent from reading code while subagents are exploring, and requires a key-files table before the main agent proceeds with detailed code reading or edits.
---

# Explore

## Core Rule

Before reading many files in the main conversation, delegate codebase reconnaissance to one or more explorer subagents. Keep the main context focused on coordination, decisions, implementation, and verification; let explorer subagents absorb verbose search results and file reads.

Use this skill when any of these signals appear:

- The task is in an unfamiliar repository or subsystem.
- The task likely needs reading about 10 or more files.
- The task spans multiple independent areas, such as API, database, UI, tests, docs, or build tooling.
- The user asks for architecture understanding, feature tracing, refactoring, migration, bug root-cause analysis, or broad review.
- You need a fresh, read-only pass before editing.

Skip delegation when the task is a tiny targeted edit and the relevant file is already known.

## Workflow

1. State briefly that you are using explorer subagents for the reconnaissance pass.
2. Identify independent research slices. Prefer 2-4 slices when the codebase is broad.
3. Spawn explorer subagents when the environment supports subagents and policy allows it. Codex supports independent subagents, so this is the default path for broad reconnaissance. Give each subagent a narrow, read-only task. Default explorer reasoning effort to `low`; raise to `medium` or `high` only when the task is architecturally complex, ambiguous, high-risk, or has multiple similar code paths that are easy to confuse.
4. While subagents run, do not read code from the target codebase in the main conversation. Only coordinate, wait, or work on unrelated non-codebase tasks.
5. Collect the explorer outputs and synthesize the result before reading large files yourself.
6. Produce a key-files table, then use it as the reading map for the next step.

Default responsibility split:

- Main agent: decide whether `$explore` applies, split the task, spawn explorers, wait for summaries, synthesize findings, and perform only the smallest necessary local verification before editing.
- Explorer subagents: perform read-only search and file inspection within their assigned slice, identify active and inactive paths when possible, and return concise evidence-backed summaries.
- Fallback local pass: only used when subagents are unavailable or blocked by policy.

## Explorer Prompt Template

Use this structure for each explorer subagent:

```text
You are doing read-only codebase reconnaissance. Do not edit files.

Goal: <specific research question>
Scope: <directories, modules, packages, feature names, or search terms>

Find:
- The most relevant files and symbols.
- How the pieces fit together.
- Which code paths are active, primary, legacy, experimental, or unused when that can be inferred.
- Existing patterns the main agent should preserve.
- Tests, fixtures, configs, or docs that matter.
- How likely changes should be grouped by concern, such as UI, API, state, data model, background jobs, permissions, i18n, styling, tests, or configuration.
- Risks, unclear areas, and follow-up reads.

Return only a concise report with:
1. Findings
2. Key files table
3. Suggested next reads

Key files table columns:
| File | Why it matters | Relevant symbols or sections | Confidence |
```

## Slicing Guidance

Split by natural boundaries:

- Vertical feature path: frontend, API route, service layer, persistence, tests.
- Layered architecture: UI, domain logic, data model, integration, build/runtime config.
- Problem hypotheses: likely root cause areas, reproduction path, neighboring implementations.
- Independent packages in a monorepo.

Avoid vague prompts such as "explore everything." A good explorer task has a scope, a question, and a requested output shape.

Concrete slicing examples:

- Authentication bug: split into `request entry points and middleware`, `session/token domain logic`, and `tests plus fixtures`. Ask each explorer to find active code paths, relevant symbols, and existing error handling conventions.
- Data persistence refactor: split into `models and schema`, `repository/query layer`, `background jobs or integrations`, and `tests/migrations/config`. Ask each explorer to identify ownership boundaries and compatibility risks.
- Frontend feature change: split into `route/page entry points`, `state/data fetching`, `component library patterns`, and `tests/storybook/docs` when those areas are independent.

## Required Synthesis

After subagents finish, summarize the useful findings and include this table before making broad reads or edits:

| File | Role | Read next? | Reason | Source |
| --- | --- | --- | --- | --- |
| `path/to/file` | Entry point / model / test / config / helper | Yes / Maybe / No | Why this file affects the task | Explorer name or local verification |

Use the table to decide the next local reads. Prefer reading only files marked `Yes` first.

Also include a short "Change Map" when the user asks where to make a feature change:

| Change type | Primary files | Notes |
| --- | --- | --- |
| UI / API / data model / tests / config | `path/to/file` | What to change here and what to avoid |

When multiple similar implementations exist, explicitly label each as `primary`, `legacy`, `experimental`, `generated`, or `unclear`. State what evidence supports the label, such as route usage, API client usage, tests, docs, naming, or recent surrounding patterns.

## Guardrails

- Treat explorer work as reconnaissance, not implementation.
- Ask explorers for summaries and file references, not full file dumps.
- Keep explorer prompts read-only unless the user explicitly asks for delegated edits and the environment allows them.
- Use parallel explorers only when the slices are independent.
- While explorer subagents are reading the target codebase, the main agent must not run local code searches, file reads, or structure scans against that same target.
- Do not bake one investigation's project-specific findings into this skill. Capture reusable rules and output shapes only.

## Fallback And Error Handling

If subagents are unavailable:

- Say that the runtime cannot spawn explorer subagents for this pass.
- Perform a local read-only reconnaissance pass using focused commands such as `rg --files`, `rg '<symbol-or-term>'`, and targeted file reads.
- Keep local notes concise and still produce the required key-files table before broad implementation reads.
- Do not present the fallback as equivalent to isolated subagent exploration; it does not provide separate context windows.

If an explorer returns empty or low-value results:

- Check whether the prompt was too narrow, used the wrong terminology, or scoped the wrong directory.
- Retry once with a broader but still bounded scope, adding concrete search terms, likely file patterns, or neighboring modules.
- If the second pass is still empty, mark the area as `unclear` in the synthesis and verify the smallest plausible local path before editing.

If explorer findings conflict:

- Preserve both claims in the synthesis with their sources.
- Verify the smallest relevant set of files locally before deciding which path is primary.
- Label the outcome as `primary`, `legacy`, `experimental`, `generated`, `unused`, or `unclear` only when there is evidence such as route usage, imports, tests, docs, naming, or recent surrounding patterns.

If an explorer returns large file dumps:

- Ignore the bulk dump for synthesis unless a specific excerpt is necessary.
- Ask for a concise retry when time permits.
- Prefer evidence paths, symbol names, and line-oriented references over copied code.

## Design Notes

This workflow adapts Claude Code's Explore Agent pattern to Codex: isolate high-volume code search in separate contexts, return only relevant summaries, and make the handoff concrete through a key-files table. For source notes, read `references/claude-explore-agent-research.md` when updating the skill or explaining its design.
