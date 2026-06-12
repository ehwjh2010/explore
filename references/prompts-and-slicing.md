# Prompts And Slicing

Read when building explorer prompts, choosing research slices, or needing concrete slicing examples. See `examples/explorer-prompt.md` for a completed fictional prompt.

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
