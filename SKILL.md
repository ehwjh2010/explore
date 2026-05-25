---
name: explore
description: Use when a coding task requires codebase reconnaissance before implementation, especially unfamiliar repositories, architecture questions, feature tracing, bug investigations, refactors, migrations, reviews, changes that may require reading roughly 10+ files, searching multiple independent areas, inspecting large or high-density files, tracing deep intra-file call chains, or reasoning about high-risk domain workflows. Skill applicability is the authorization signal: when a task matches these conditions, or when the user invokes $explore, subagent reconnaissance is mandatory. Spawn read-only explorer subagents before the main agent reads target code or edits files; if subagents cannot be spawned, stop and report that Explore is blocked instead of doing local fallback reconnaissance.
---

# Explore

## Core Rule

Skill applicability is the authorization signal. When the task matches this skill's applicability conditions, or when the user invokes `$explore`, explorer subagent reconnaissance is mandatory. The user does not need to also say `subagents`, `delegate`, `parallel agents`, or any other authorization phrase.

There is no local reconnaissance fallback for Explore. If explorer subagents cannot be spawned, stop the Explore workflow and report that the task is blocked by subagent unavailability. Do not replace explorer work with local `rg`, file reads, or structure scans.

Keep the main context focused on coordination, decisions, implementation, and verification; let explorer subagents absorb verbose search results and file reads.

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

When choosing whether this skill applies, skip it for a tiny targeted edit where the relevant file and change are already known, such as a typo, one-line constant update, or simple style fix. If the task meets the Explore conditions or the user explicitly invokes `$explore`, subagent reconnaissance is required before the main agent reads target code or edits files.

## Workflow

1. State briefly that you are using the Explore workflow and explorer subagents for the reconnaissance pass.
2. Identify independent research slices. Start with business dimensions, risk hypotheses, state paths, permission boundaries, and side effects; use technical layers as supporting boundaries. Let the number of explorer subagents follow the useful, non-overlapping slices; do not impose a fixed count.
3. Spawn explorer subagents when this skill applies; the matching task is enough authorization and subagent reconnaissance is required. Give each subagent a clear, complementary, read-only task. Default explorer reasoning effort to `medium`; raise to `high` only when the task is architecturally complex, ambiguous, high-risk, or has multiple similar code paths that are easy to confuse.
4. While subagents run, do not read code from the target codebase in the main conversation. Only coordinate, wait, or work on unrelated non-codebase tasks.
5. Collect the explorer outputs and synthesize the result before reading large files yourself.
6. Produce a key-files table, then use it as the reading map for the next step.

Default responsibility split:

- Main agent: split the task, spawn explorers, wait for summaries, synthesize findings, and perform only the smallest necessary local verification before editing.
- Explorer subagents: perform read-only search and file inspection within their assigned slice, identify active and inactive paths when possible, and return concise evidence-backed summaries.
- No fallback local pass: if explorer subagents cannot run, stop and report the blocker.

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

Prefer business-first slicing. Technical layers such as frontend, backend, service layer, or tests are useful starting points, but they are not always natural explorer boundaries. Do not treat `backend`, `service layer`, or `business logic` as a single explorer task when that area contains multiple business rules, state paths, side effects, or risk hypotheses.

Split by natural boundaries:

- Vertical feature path: frontend, API route, service layer, persistence, tests, then split any dense layer further by business concern.
- Layered architecture: UI, domain logic, data model, integration, build/runtime config.
- Problem hypotheses: likely root cause areas, reproduction path, neighboring implementations.
- Independent packages in a monorepo.
- Dense single-path investigations: entry point, call chain, data flow, tests, historical implementation, and risk assumptions.

For backend-heavy or workflow-heavy tasks, consider separate explorer slices for:

- Request entry points, routing, and parameter validation.
- Permissions, tenancy, ownership, visibility, or role constraints.
- Business state machines, workflow transitions, lifecycle rules, or approval paths.
- Domain services, orchestration, command handlers, or policy objects.
- Data models, persistence, query behavior, migrations, and compatibility.
- External integrations, webhooks, third-party APIs, and sync paths.
- Background jobs, queues, schedulers, notifications, audit logs, and other side effects.
- Tests, fixtures, factories, and regression coverage.

For complex tasks, prefer multiple explorer subagents split by independent questions, layers, risk hypotheses, or code regions. For a single path or file that is large and dense, multiple explorers can still be useful when each checks a different perspective such as entry points, call graph, data flow, tests, historical implementations, or risk points.

Avoid vague prompts such as "explore everything" or "inspect all backend logic." A good explorer task has a scope, a question, and a requested output shape. Do not send multiple explorers to answer the same question over the same scope.

Concrete slicing examples:

- Authentication bug: split into `request entry points and middleware`, `session/token domain logic`, and `tests plus fixtures`. Ask each explorer to find active code paths, relevant symbols, and existing error handling conventions.
- Data persistence refactor: split into `models and schema`, `repository/query layer`, `background jobs or integrations`, and `tests/migrations/config`. Ask each explorer to identify ownership boundaries and compatibility risks.
- Frontend feature change: split into `route/page entry points`, `state/data fetching`, `component library patterns`, and `tests/storybook/docs` when those areas are independent.
- Workflow or work-order change: do not send one `backend` explorer for everything. Split into `request path and controller behavior`, `workflow state transition rules`, `permissions and visibility constraints`, `persistence and query behavior`, `side effects such as jobs, notifications, audit, or integrations`, and `tests and fixtures` when those concerns are present and independent.

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
- Keep explorer prompts read-only. Do not add restrictions beyond the read-only reconnaissance scope unless the task itself requires a narrower boundary.
- Use parallel explorers only when the slices are independent.
- While explorer subagents are reading the target codebase, the main agent must not run local code searches, file reads, or structure scans against that same target.
- Do not bake one investigation's project-specific findings into this skill. Capture reusable rules and output shapes only.

## Blocking And Error Handling

If subagents are unavailable or the spawn tool rejects the request at runtime:

- Say that Explore is blocked because explorer subagents are mandatory for this skill and cannot be spawned in the active runtime.
- If the rejection appears to require narrower explicit user wording, explain that runtime limitation without changing the skill rule that applicability is authorization.
- Do not perform local read-only reconnaissance with `rg --files`, `rg '<symbol-or-term>'`, targeted file reads, or equivalent local codebase scanning.
- Do not produce the required key-files table from local fallback notes.

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
