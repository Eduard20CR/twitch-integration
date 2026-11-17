# ADR-0004 — Store the Streamer's Twitch tokens, encrypted, to announce Results in the chat

Status: Accepted

At first we did not store the Streamer's tokens, because the EventSub subscriptions only need the app access token. Now the app writes each Round's Result in the Channel's chat, as the Streamer, and that needs the Streamer's user access token (scope `user:write:chat`) at any time, so we store the access token and the refresh token in MongoDB, encrypted with AES-GCM and a key that lives only in the backend's environment. The App Server refreshes the access token when it expires, with a Redis lock so only one copy refreshes at a time, and the reaper validates the tokens every hour, as Twitch requires. Learning this full token lifecycle is also a goal of the project.

## Considered options

- **Keep not storing tokens**: nothing secret to protect, but the app could not write in the chat.
- **Store the tokens in plain text**: simpler, but anyone who reads the database could write in every Streamer's chat.

## Consequences

- If the encryption key is lost, every Streamer has to sign in again; if it leaks, the tokens are exposed, so it is treated like the client secret.
- When the refresh token stops working (the Streamer changed their password or disconnected the app), the Streamer is asked to sign in again, the same as after a revocation.
- Deleting the account also revokes the token at Twitch.
