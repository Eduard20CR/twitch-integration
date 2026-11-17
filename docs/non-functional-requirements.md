# Non-functional requirements

Each requirement has an ID (`NFR-<AREA>-XX`) and a verifiable criterion.

---

## Security (SEC)

| ID | Requirement | How it is verified |
|---|---|---|
| NFR-SEC-01 | The Twitch client secret, the webhook secret and every other secret live only in the backend's environment (`.env` locally). They are never in the frontend bundle, the code or the repo. `.env` is in `.gitignore`; `.env.example` has the names only. | Review of every pull request; a search of the built frontend bundle for the secrets' names finds nothing. |
| NFR-SEC-02 | The Webhook Server verifies every request's signature: HMAC-SHA256 with the webhook secret over `Twitch-Eventsub-Message-Id` + `Twitch-Eventsub-Message-Timestamp` + the raw body, prefixed with `sha256=`, compared with a constant-time function (`hmac.compare_digest`). A request with a missing or wrong signature gets a 403 and nothing is published. | Tests: a valid signature passes, a wrong one and a missing one get 403. |
| NFR-SEC-03 | The Webhook Server does not publish a message whose `Twitch-Eventsub-Message-Timestamp` is more than 10 minutes old, and still answers 2xx, so Twitch does not count it as a failure and revoke the subscription. | Test with an old timestamp: not published, answered 2xx. |
| NFR-SEC-04 | The Streamer is asked only for the scopes `user:read:chat`, `user:bot`, `channel:bot` and `user:write:chat`. | Review of the OAuth request; a manual sign-in shows only those permissions. |
| NFR-SEC-05 | The Streamer's access token and refresh token are stored only encrypted, with AES-GCM and a key from the backend's environment (`TOKEN_ENCRYPTION_KEY`), using the Streamer id as associated data ([ADR-0004](adr/0004-store-streamer-tokens-encrypted.md)). | Test: the stored value is not the token, and it cannot be decrypted with another key or under another Streamer id. |
| NFR-SEC-06 | The session cookie is `__Host-session`, `HttpOnly`, `Secure`, `SameSite=Lax`, `Path=/`, and holds only a random id (`secrets.token_urlsafe(32)`). The session itself is in Redis. | Test of the login response headers. |
| NFR-SEC-07 | Every login creates a new session id; an old one is never reused. | Test: two logins give two different ids. |
| NFR-SEC-08 | The Twitch login uses a random `state` value, checked on the callback. A callback with a missing or wrong `state` is rejected. | Test of the callback. |
| NFR-SEC-09 | Every request that changes data (POST, DELETE) and every WebSocket opening checks that the `Origin` header is the Dashboard's origin; otherwise it is rejected with 403. | Tests with a foreign `Origin` and with no `Origin`. |
| NFR-SEC-10 | A session ends 7 days after login, or after 24 hours without use, whichever comes first. Logout deletes the session from Redis at once. | Tests of both expiries and of logout. |
| NFR-SEC-11 | Outside `localhost`, the app is served only over HTTPS. | Review when hosting is decided. |
| NFR-SEC-12 | The session id is never written to logs. | Review of the logging code; a test that the login log line has no session id. |
| NFR-SEC-13 | The Streamer's tokens never reach the frontend and are never written to logs. | Review of the API responses and of the logging code. |
| NFR-SEC-14 | Every stored token is validated with Twitch (`GET https://id.twitch.tv/oauth2/validate`) at least once an hour, as Twitch requires. A token that is no longer valid and cannot be refreshed means the Streamer must sign in again. A revocation asks the Streamer to sign in again only when their permission is gone (`authorization_revoked`, `user_removed`); for any other reason the subscription is just created again. | Test of the reaper with a valid and an invalid token. |

## Privacy (PRIV)

| ID | Requirement | How it is verified |
|---|---|---|
| NFR-PRIV-01 | Chat Messages are never stored: not in MongoDB, Redis (beyond Pub/Sub delivery) or `localStorage`. | Review of the data model and of the Dashboard's `localStorage` keys. |
| NFR-PRIV-02 | A Giveaway Result keeps the number of entries and the winner (Twitch id and display name), never the list of entrants. | Review of the Result shape; test of the saved Result. |
| NFR-PRIV-03 | The Streamer can delete their account. It does, in this order: delete their Twitch subscription, revoke their token at Twitch, delete their Results, their Redis keys (every one of their sessions, in every browser, and `needs_reauth`) and their Streamer record. Each step skips what no longer exists, so a failed deletion can be retried. | Test: the deletion run twice ends with no data left and no error. |

## Reliability (REL)

| ID | Requirement | How it is verified |
|---|---|---|
| NFR-REL-01 | A Chat Message that Twitch delivers more than once (same `Twitch-Eventsub-Message-Id`) is published once, even when the copies reach different Webhook Server copies. A repeated one still gets a 2xx. | Test: the same message sent twice is published once, and both get 2xx. |
| NFR-REL-02 | No server keeps anything important in its own memory: sessions, presence and the Active Dashboard are in Redis; Streamers and Results are in MongoDB. Any copy of a server can handle any request. docker-compose runs two copies of the App Server and two of the Webhook Server behind Traefik. | Review; the end-to-end tests run against the two copies. |
| NFR-REL-03 | No connection or subscription stays open forever. The App Server pings each WebSocket every 30 seconds and closes it after 60 seconds without an answer. A Twitch subscription whose Streamer has had no Dashboard for 10 minutes is deleted by the reaper, which checks Twitch's real subscription list, not our data. The reaper also deletes any subscription whose callback is not the current webhook address. | Tests of the ping timeout and of the reaper. |
| NFR-REL-04 | The Dashboard reconnects on its own when the WebSocket drops, and a Round in progress survives a page reload (`localStorage`). Chat Messages that arrive during the reconnect are lost ([ADR-0002](adr/0002-chat-delivery-with-redis-pub-sub.md)). | Manual test: reload the page in the middle of a Round. |
| NFR-REL-05 | Only one App Server copy refreshes a Streamer's token at a time: it takes `SET refresh_lock:<streamer_id> NX EX 30` first, saves the new access token and the new refresh token, and the other copies read them instead of refreshing. | Test: two concurrent refreshes call Twitch once. |

## Performance (PERF)

| ID | Requirement | How it is verified |
|---|---|---|
| NFR-PERF-01 | The Webhook Server answers Twitch in under 200 ms (p95): it does only the checks, the duplicate key and the publish before answering. | Load test with the Twitch CLI against local docker-compose. |
| NFR-PERF-02 | A Chat Message reaches the Dashboard in under 1 second (p95) after the Webhook Server receives it, with 20 Chat Messages per second in one Channel. | Load test against local docker-compose. |

## Cost (COST)

| ID | Requirement | How it is verified |
|---|---|---|
| NFR-COST-01 | Running the app costs nothing: everything runs locally in docker-compose, the load balancer is Traefik, and the tunnel is ngrok's free plan. Any paid service needs a new ADR first. | Review. |

## Maintainability (MAINT)

| ID | Requirement | How it is verified |
|---|---|---|
| NFR-MAINT-01 | Every Game implements the same interface (start, stop, receive a Chat Message, show progress). Adding a Game is adding one new file plus its registration; nothing else changes. | Review of the pull request that adds a Game. |
| NFR-MAINT-02 | Games receive Chat Messages in the app's own format (id, Viewer id, Viewer name, color, text, date) and know nothing about Twitch. | Review: no Twitch types or field names inside a Game. |
