# Explore

Explore is a Codex skill for delegating broad codebase reconnaissance to explorer subagents before the main agent reads or edits many files.

It is useful when a coding task touches an unfamiliar repository, spans multiple areas, or likely requires reading many files. The skill asks subagents to return concise findings, a key-files table, and suggested next reads, so the main conversation stays focused on decisions and implementation.

## Install

Clone this repository into your Codex skills directory:

```bash
git clone https://github.com/oil-oil/codex-explore-skill.git ~/.codex/skills/explore
```

Then invoke it with:

```text
Use $explore to map this codebase before making changes.
```

## What It Enforces

- Delegate large read-only code exploration to explorer subagents first.
- Keep the main agent from reading the target codebase while subagents are exploring.
- Require a key-files table before broad local reads or edits.
- Ask explorers to distinguish primary, legacy, experimental, generated, unused, or unclear code paths when possible.
- Group likely changes by concern, such as UI, API, state, data model, permissions, i18n, tests, or configuration.

## License

MIT
