# Explore

Explore is a Codex skill for delegating codebase reconnaissance to explorer subagents before the main agent reads or edits many files or high-density code paths.

It is useful when a coding task touches an unfamiliar repository, spans multiple areas, likely requires reading many files, or depends on a small number of large, dense files. The skill asks subagents to return concise findings, a key-files table, and suggested next reads, so the main conversation stays focused on decisions and implementation.

Skill applicability is the authorization signal for the main coordinator agent: when a task meets the Explore conditions, or when the user invokes `$explore`, the skill treats that as an explicit request to delegate read-only reconnaissance to explorer subagents. Subagent reconnaissance is mandatory before the main agent does detailed local reads or edits. The main agent should split the question, send complementary read-only slices to explorer subagents, wait for every spawned explorer to reach a terminal result, explicitly close or terminate each spawned explorer through the active runtime's available mechanism, record cleanup status, and only then synthesize their summaries before doing detailed local reads.

There is no local fallback reconnaissance mode for the main coordinator agent. If explorer subagents cannot be spawned, Explore stops and reports the blocker instead of scanning the codebase locally. Explorer subagents spawned by Explore are different: they must not invoke Explore again and should search and read files directly.

## Install

Clone this repository into your Codex skills directory:

```bash
git clone https://github.com/oil-oil/codex-explore-skill.git ~/.codex/skills/explore
```

Then use it by giving a task that meets the Explore conditions, or invoke it explicitly with:

```text
Use $explore to delegate explorer subagents to map this codebase before making changes.
```

## When To Use It

Use Explore when the work benefits from isolating high-noise code reading in explorer subagents before implementation:

- Understanding an unfamiliar repository or subsystem.
- Tracing a feature across UI, API, state, data model, tests, and configuration.
- Investigating a bug where several root-cause areas are plausible.
- Planning a refactor, migration, review, or architecture change that may require many reads.
- Inspecting a few large or high-density files where core workflow, service, controller, or engine logic is concentrated.
- Tracing several symbols, branches, call chains, permissions, state transitions, or data assembly paths inside a small file set.

When choosing proactively, skip it for tiny targeted edits where the relevant file and change are already obvious, such as a typo, one-line constant update, or simple style fix. If the task meets the Explore conditions or the user explicitly invokes `$explore`, run subagent reconnaissance before the main coordinator agent reads target code or edits files.

When splitting explorer work, prefer business dimensions and risk hypotheses before broad technical buckets. For example, a backend-heavy workflow may need separate explorer slices for request routing, state transitions, permissions, persistence, side effects, and tests rather than one generic `backend` explorer.

## Project Map

- `SKILL.md`: the runtime instructions for the Codex skill.
- `agents/openai.yaml`: minimal OpenAI agent metadata. Keep this conservative unless the supported schema is confirmed.
- `references/`: progressive workflow instructions, background research, and design notes.
- `examples/`: sample explorer prompts and synthesized outputs.

## What It Enforces

- Delegate read-only code exploration to explorer subagents when the task meets the Explore conditions or the user invokes `$explore`, based on reconnaissance complexity rather than file count alone.
- Keep the main agent from reading the target codebase while any rostered subagent is still running, pending, exploring, or has unknown status.
- Require an all-subagents barrier: partial results from one explorer, a subset of explorers, a majority of explorers, or the most confident explorer are interim notes only and are not enough for the coordinator to start implementation.
- Treat a terminal result as report completion, not subagent lifecycle cleanup.
- Require explicit cleanup of each spawned explorer before synthesis, broad local reads, or edits; record each cleanup status as `closed`, `close failed`, or `close unavailable`.
- Allow synthesis after a terminal-result explorer cannot be closed only when the close failure or unavailable close mechanism is explicitly reported as a runtime resource risk.
- Stop instead of using local fallback if explorer subagents cannot be spawned by the main coordinator agent.
- Tell explorer subagents that they are already delegated workers, should not invoke Explore again, and should perform their assigned read-only local search and file inspection directly.
- Require a key-files table before broad local reads or edits.
- Ask explorers to distinguish primary, legacy, experimental, generated, unused, or unclear code paths when possible.
- Group likely changes by concern, such as UI, API, state, data model, permissions, i18n, tests, or configuration.
- Ask explorers for concise summaries and evidence paths, not large file dumps.
- Split explorer work into clear, complementary slices rather than repeating the same question.
- Refine coarse layers like `backend`, `service layer`, or `business logic` into smaller business or risk slices when they contain multiple rules, workflows, side effects, or state paths.

## Expected Output

A completed reconnaissance pass means every rostered explorer reached a terminal result: useful report, empty or low-value report, explicit failure, or explicit unavailability. It also means the coordinator has recorded cleanup status for every spawned explorer: `closed`, `close failed`, or `close unavailable`. Only after both the terminal-result barrier and cleanup recording should the main agent create a small reading map:

| File | Role | Read next? | Reason | Source |
| --- | --- | --- | --- | --- |
| `src/routes/login.ts` | API entry point | Yes | Handles the request path for the reported issue | auth explorer |
| `src/auth/session.ts` | Domain helper | Yes | Owns token validation and expiration behavior | auth explorer |
| `tests/auth/session.test.ts` | Test | Maybe | Covers nearby behavior but not the failing case | auth explorer |

For feature work, include a short change map:

| Change type | Primary files | Notes |
| --- | --- | --- |
| API | `src/routes/login.ts` | Preserve existing error response shape |
| Tests | `tests/auth/session.test.ts` | Add expiration and invalid-token coverage |

See `examples/` for full prompt and synthesis examples.

## Runtime Notes

This skill requires explorer subagent spawning when it applies to the main coordinator agent; no extra user phrase such as `subagents`, `delegate`, or `parallel agents` is required by the skill. Whether subagents can actually be spawned and explicitly closed is determined by the active Codex runtime and available tools.

Some guarantees remain runtime-dependent. Tool availability, permission enforcement, automatic routing, parallel scheduling details, close or terminate APIs, and whether only summaries return to the main context are controlled by the agent platform, not by this repository alone. When the coordinator runtime cannot spawn explorer subagents or the tool rejects the request, the skill stops and says Explore is blocked. After spawned explorers return terminal results, the coordinator must use any runtime-supported close, terminate, or equivalent mechanism and record whether each explorer was closed, failed to close, or could not be closed because cleanup is unavailable. If cleanup fails, the coordinator retries once when supported and reports any remaining failure instead of implying automatic lifecycle handling. If the runtime requires narrower explicit wording for spawn authorization, the agent should identify that as a runtime limitation and ask the user to re-authorize with the wording required by that runtime. An explorer subagent should not treat inability to spawn another agent as blocked; it should complete its own assigned read-only reconnaissance directly.

## License

MIT
