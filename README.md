# Explore

Explore is a Codex skill for delegating complex codebase reconnaissance to explorer subagents before the main agent reads or edits many files or high-density code paths.

It is useful when a coding task touches an unfamiliar repository, spans multiple areas, likely requires reading many files, or depends on a small number of large, dense files. The skill asks subagents to return concise findings, a key-files table, and suggested next reads, so the main conversation stays focused on decisions and implementation.

Codex supports independent subagents, so this skill treats explorer delegation as the default path for broad reconnaissance. The main agent should split the question, send read-only slices to explorer subagents, and synthesize their summaries before doing detailed local reads.

## Install

Clone this repository into your Codex skills directory:

```bash
git clone https://github.com/oil-oil/codex-explore-skill.git ~/.codex/skills/explore
```

Then invoke it with:

```text
Use $explore to map this codebase before making changes.
```

## When To Use It

Use `$explore` when the work benefits from a reconnaissance pass before implementation:

- Understanding an unfamiliar repository or subsystem.
- Tracing a feature across UI, API, state, data model, tests, and configuration.
- Investigating a bug where several root-cause areas are plausible.
- Planning a refactor, migration, review, or architecture change that may require many reads.
- Inspecting a few large or high-density files where core workflow, service, controller, or engine logic is concentrated.
- Tracing several symbols, branches, call chains, permissions, state transitions, or data assembly paths inside a small file set.

Skip it for tiny targeted edits where the relevant file and change are already obvious, such as a typo, one-line constant update, or simple style fix.

## Project Map

- `SKILL.md`: the runtime instructions for the Codex skill.
- `agents/openai.yaml`: minimal OpenAI agent metadata. Keep this conservative unless the supported schema is confirmed.
- `references/`: background research and design notes.
- `examples/`: sample explorer prompts and synthesized outputs.

## What It Enforces

- Delegate complex read-only code exploration to explorer subagents first, based on reconnaissance complexity rather than file count alone.
- Keep the main agent from reading the target codebase while subagents are exploring.
- Require a key-files table before broad local reads or edits.
- Ask explorers to distinguish primary, legacy, experimental, generated, unused, or unclear code paths when possible.
- Group likely changes by concern, such as UI, API, state, data model, permissions, i18n, tests, or configuration.
- Ask explorers for concise summaries and evidence paths, not large file dumps.

## Expected Output

A completed reconnaissance pass should give the main agent a small reading map:

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

This skill is designed for Codex environments with independent explorer subagents. It is close to Claude Code's Explore Agent pattern when the runtime supports separate subagent contexts, parallel independent investigations, and concise result handoff.

Some guarantees remain runtime-dependent. Tool permission enforcement, automatic routing, parallel scheduling details, and whether only summaries return to the main context are controlled by the agent platform, not by this repository alone. The skill documents the desired behavior and fallback path when subagents are unavailable.

## License

MIT
