# Synthesis And Error Handling

Read before producing the required synthesis, handling failed or low-value explorers, resolving conflicts, or reporting blocked states. See `examples/synthesized-output.md` for a completed fictional synthesis.

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
