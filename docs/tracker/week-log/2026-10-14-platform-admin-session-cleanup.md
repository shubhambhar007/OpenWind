# Platform-admin session cleanup (#876)

## Acceptance criteria

- Ignore a session response that settles after its dashboard effect has been cleaned up.
- Preserve the identity banner when the active session request succeeds.
- Keep session-fetch failures non-fatal and silent.
- Cover the StrictMode stale-response regression and pass admin UI test, typecheck, and lint checks.

## Design

Use an effect-local cancellation flag, set it during cleanup, and guard the session state update with it. This matches the existing platform-admin MFA guard pattern without changing `fetchWithAuth` or its public API.

The tradeoff is that the request still completes in the background; the component only suppresses its stale result. An `AbortController` would cancel network work but would require threading a signal through the shared fetch helper for a small, isolated lifecycle fix.

Residual risk is limited to the separate sandbox refresh request, which is outside this issue's session-fetch scope.

This is not persistent or distributed state: the cancellation flag exists only for one React effect lifecycle.
