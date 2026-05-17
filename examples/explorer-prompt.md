# Explorer Prompt Example

This example shows a bounded, read-only task for an explorer subagent. The project is fictional.

```text
You are doing read-only codebase reconnaissance. Do not edit files.

Goal: Find the active authentication request path for password login and identify the files the main agent should read before changing session expiration behavior.

Scope:
- Search terms: login, password, session, token, expires, auth middleware
- Likely areas: src/routes, src/auth, src/middleware, tests/auth

Find:
- The most relevant files and symbols.
- How the login request reaches session creation.
- Which code paths are active, primary, legacy, experimental, or unused when that can be inferred.
- Existing patterns the main agent should preserve.
- Tests, fixtures, configs, or docs that matter.
- Risks, unclear areas, and follow-up reads.

Return only a concise report with:
1. Findings
2. Key files table
3. Suggested next reads

Key files table columns:
| File | Why it matters | Relevant symbols or sections | Confidence |
```

Good explorer output should name evidence paths and symbols. It should not paste large chunks of source code.
