# Explorer Worker Workflow

Read when the prompt says you are an explorer subagent spawned by the Explore skill.

## Explorer Worker Mode

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
