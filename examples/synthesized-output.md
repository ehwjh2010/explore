# Synthesized Output Example

This example shows how the main agent can combine explorer findings before broad local reads. The project is fictional.

## Findings

All rostered explorer subagents have reached terminal results. The auth-path explorer and tests explorer returned useful findings; the legacy-path explorer returned no active usage beyond migration tests.

The active password-login path appears to be `src/routes/login.ts -> src/auth/password.ts -> src/auth/session.ts`. An older implementation under `src/legacy/auth.ts` is imported only by migration tests and should not be changed for the requested behavior.

The main risk is preserving the existing error response shape while changing session expiration logic. The current tests cover invalid passwords, but not expiration edge cases.

## Key Files

| File | Role | Read next? | Reason | Source |
| --- | --- | --- | --- | --- |
| `src/routes/login.ts` | API entry point | Yes | Handles the active password-login request and maps auth errors to responses | auth-path explorer |
| `src/auth/password.ts` | Domain helper | Yes | Verifies credentials before session creation | auth-path explorer |
| `src/auth/session.ts` | Session logic | Yes | Owns token creation and expiration calculation | auth-path explorer |
| `tests/auth/session.test.ts` | Test | Yes | Best place to add expiration coverage | tests explorer |
| `src/legacy/auth.ts` | Legacy implementation | No | Only referenced by migration tests; not active for login route | legacy-path explorer |

## Change Map

| Change type | Primary files | Notes |
| --- | --- | --- |
| API | `src/routes/login.ts` | Preserve current response body and status codes |
| Domain logic | `src/auth/session.ts` | Change expiration behavior here, not in route code |
| Tests | `tests/auth/session.test.ts` | Add expiration and boundary coverage |

## Open Questions

- Confirm whether the expiration value should come from config or remain hard-coded.
- Check if refresh tokens use the same expiration helper before editing.
