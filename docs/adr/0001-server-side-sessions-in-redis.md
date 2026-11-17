# ADR-0001 — Server-side sessions in Redis, not Cognito or JWT

Status: Accepted

The Streamer signs in with Twitch, and the App Server keeps the session as a random id in an `HttpOnly`, `Secure`, `SameSite=Lax` cookie (`__Host-session`), with the session itself stored in Redis. We chose it because any App Server copy can read it, logout takes effect at once, and it runs the same in docker-compose and on AWS. The session rules are in `docs/non-functional-requirements.md`.

## Considered options

- **Amazon Cognito**: Twitch is not one of its built-in providers, so it would need a generic OIDC setup asking for the chat scopes, it adds a piece between us and Twitch instead of removing one, and it has no free local version for docker-compose.
- **JWT**: no server state, but it cannot be revoked before it expires, and storing it where JavaScript can read it exposes it to scripts on the page.
