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

Highest-priority role rule: if you are already an explorer subagent spawned by the Explore skill, do not invoke Explore again. Perform the assigned read-only search and file inspection directly, then return your concise report.

For the main coordinator agent, skill applicability is the authorization signal. When the task matches this skill's applicability conditions, or when the user invokes `$explore`, explorer subagent reconnaissance is mandatory. The user does not need to also say `subagents`, `delegate`, `parallel agents`, or any other authorization phrase.

There is no local reconnaissance fallback for the main coordinator agent. If explorer subagents cannot be spawned, stop the Explore workflow and report that the task is blocked by subagent unavailability. Do not replace explorer work with local `rg`, file reads, or structure scans.

The coordinator must treat spawned explorers as an all-subagents barrier. Once a set of explorers has been launched, do not read target code, synthesize a final reading map, make implementation decisions, or edit files until every launched explorer has returned a terminal result: useful report, empty/low-value report, explicit failure, or explicit unavailability. A terminal result only means the explorer has completed its report; it does not mean the spawned subagent has been closed or terminated. A subset of completed explorers is only interim progress, not authorization to proceed.

After every launched explorer reaches a terminal result, the coordinator must explicitly clean up the spawned subagents before synthesis, local target-code reads, implementation decisions, or edits. Use the current runtime's supported close, terminate, or equivalent operation for each started explorer. Record a cleanup status for each explorer: `closed`, `close failed`, or `close unavailable`. If cleanup fails, retry once when the runtime supports retrying. If cleanup remains failed, keep the returned reconnaissance result but report the close failure and the possible runtime resource risk in the synthesis. If the runtime has no close or terminate API, record and report `close unavailable` instead of implying the subagent was closed.

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

When choosing whether this skill applies, skip it for a tiny targeted edit where the relevant file and change are already known, such as a typo, one-line constant update, or simple style fix. If the task meets the Explore conditions or the user explicitly invokes `$explore`, subagent reconnaissance is required before the main coordinator agent reads target code or edits files.

## Workflow

### Coordinator Mode

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

### Explorer Worker Mode

Use this mode when the prompt says you are an explorer subagent spawned by the Explore skill.

1. Do not invoke Explore again. Perform the assigned read-only search and file inspection directly.
2. Treat local `rg`, file reads, directory inspection, and similar read-only codebase scanning as your normal assigned work, not as fallback.
3. Start broad-to-narrow inside the assigned scope: first identify relevant directories, entry points, naming patterns, configs, tests, docs, and other clues, then narrow to the key files and symbols that explain the path.
4. Combine search methods when available and useful: filename search, text search, structural or AST search, reference or LSP search, config/test/doc search, and history search. These are guidance for better judgment, not mandatory tool dependencies.
5. Match reconnaissance scope to explorer reasoning effort. With `low`, identify entry points, key files, the main path evidence, and obvious tests or configs. With `medium` or `high`, also trace references, analogous implementations, structural matches, history clues, and competing paths.
6. When evidence is sufficient, include the active path or control/data flow in `Findings` as a line or short list, such as `route -> service -> repository -> test`. If it cannot be confirmed, write `unclear` and name the missing evidence.
7. Look for nearby analogous implementations and local patterns in naming, error handling, permissions, configuration, and tests. Report the pattern the main agent should preserve, or write `No analogous implementation found in assigned scope` when none is found.
8. Stay within the assigned goal and scope unless a small neighboring read is necessary to avoid a misleading report.
9. Return concise evidence-backed findings, a key-files table, and suggested next reads. Do not edit files.

Default responsibility split:

- Main coordinator agent: split the task, spawn explorers, wait for summaries, synthesize findings, and perform only the smallest necessary local verification before editing.
- Explorer worker subagents: perform direct read-only search and file inspection within their assigned slice, identify active and inactive paths when possible, and return concise evidence-backed summaries. This direct local exploration is their delegated job, not a fallback.
- No coordinator fallback local pass: if explorer subagents cannot run, stop and report the blocker.

## Explorer Prompt Template

Use this structure for each explorer subagent:

```text
Do not invoke Explore again. Perform the assigned read-only search and file inspection directly.

You are doing read-only codebase reconnaissance. Do not edit files.

Goal: <specific research question>
Scope: <directories, modules, packages, feature names, or search terms>

Find:
- The most relevant files and symbols.
- The broad-to-narrow search path used inside the assigned scope.
- How the pieces fit together.
- The active control/data path, such as `route -> service -> repository -> test`, or `unclear` with the missing evidence.
- Which code paths are active, primary, legacy, experimental, or unused when that can be inferred.
- Nearby analogous implementations and local patterns the main agent should preserve.
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

After every rostered subagent has reached a terminal result and the coordinator has recorded cleanup status for each started explorer, summarize the useful findings and include this table before making broad reads or edits. The synthesis must mention any explorer that failed, returned low-value findings, was unavailable, had cleanup unavailable, or could not be closed, so the user can see that the coordinator did not proceed from only a partial subset or hide subagent lifecycle risk.

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
- Keep explorer prompts read-only and start them with the explorer subagent identity and no-recursion instructions from the template. Do not add restrictions beyond the read-only reconnaissance scope unless the task itself requires a narrower boundary.
- Use parallel explorers only when the slices are independent.
- While any rostered explorer subagent is still running, pending, or has unknown status, the main agent must not run local code searches, file reads, or structure scans against that same target.
- Do not treat the first useful explorer report, a majority of explorer reports, or the highest-confidence explorer report as sufficient to continue. The barrier is every rostered explorer reaching a terminal result.
- Do not treat a terminal result as lifecycle cleanup. Before synthesis, target-code reads, implementation decisions, or edits, the main agent must close or terminate each started explorer through the active runtime's available mechanism, or record and report `close failed` or `close unavailable`.
- If an explorer reached a terminal result but cleanup failed after the supported retry, the main agent may synthesize from that explorer's returned result, but must explicitly report the close failure and possible runtime resource risk.
- If an explorer has not reached a terminal result, cleanup failure or cleanup unavailability cannot bypass the all-subagents barrier.
- Explorer worker subagents may run local code searches, file reads, and structure scans inside their assigned read-only scope.
- Explorer worker search-method guidance does not authorize coordinator-local reconnaissance or fallback. It applies only to spawned explorer workers inside their assigned read-only scope.
- Do not bake one investigation's project-specific findings into this skill. Capture reusable rules and output shapes only.

## Blocking And Error Handling

If you are the main coordinator agent and subagents are unavailable or the spawn tool rejects the request at runtime:

- Say that Explore is blocked because explorer subagents are mandatory for this skill and cannot be spawned in the active runtime.
- If the rejection appears to require narrower explicit user wording, explain that runtime limitation without changing the skill rule that applicability is authorization.
- Do not perform local read-only reconnaissance with `rg --files`, `rg '<symbol-or-term>'`, targeted file reads, or equivalent local codebase scanning.
- Do not produce the required key-files table from local fallback notes.

If you are an explorer worker subagent, lack of a spawn tool is not a blocker. Do not try to delegate; complete the assigned read-only reconnaissance directly and report any inaccessible paths or missing context as findings.

If an explorer returns empty or low-value results:

- Check whether the prompt was too narrow, used the wrong terminology, or scoped the wrong directory.
- Retry once with a broader but still bounded scope, adding concrete search terms, likely file patterns, or neighboring modules.
- If the second pass is still empty, mark the area as `unclear` in the synthesis and verify the smallest plausible local path before editing.

If one or more explorers are still pending while others have finished:

- Wait for the pending explorers before synthesizing final findings or reading target code locally.
- If the runtime reports a pending explorer as failed, cancelled, unavailable, or unreachable, include that terminal status in the synthesis and mark its area as `unclear`.
- Do not replace a pending explorer with coordinator-local reconnaissance. If the area is still necessary, spawn a bounded replacement explorer when the runtime allows it; otherwise report that Explore is blocked or partially blocked for that slice.

If one or more explorers reached a terminal result but cleanup fails:

- Retry the close, terminate, or equivalent cleanup operation once when the runtime supports retrying.
- If the retry succeeds, record the explorer cleanup status as `closed`.
- If the retry fails, record `close failed`, keep the explorer's returned findings, continue synthesis only after all rostered explorers have terminal results, and report the close failure plus possible runtime resource risk.
- If the runtime has no close or terminate API, record `close unavailable` and report cleanup unavailability rather than claiming the explorer was closed.
- Do not use close failure, close unavailability, or an attempted termination as a substitute for a missing terminal result from a running, pending, or unknown-status explorer.

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
