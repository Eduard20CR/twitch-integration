# Twitch Integration

## Vision

### Problem

A Streamer wants to play live games with the Viewers of their Channel, such as a vote or a giveaway, but the Viewers are only in the Twitch chat. Counting `!vote A` messages or picking a random entrant by hand, live, is slow and error-prone.

### Who it is for

Twitch Streamers. Any Streamer can sign in with Twitch and use it, and many Streamers can use it at the same time. Viewers never use the app: they only write in the Channel's chat.

### What it does

- The Streamer signs in with Twitch and opens the Dashboard, which shows the Channel's live chat.
- The Dashboard has one tab per Game. The first version has two Games: Voting (`!vote A`) and Giveaway (`!join`). Trivia and Guess the Word come later.
- The Streamer shows the Dashboard on their stream (an OBS window capture), and the Game reacts live to the Commands Viewers write in the chat.
- When a Round ends, the app writes its Result in the Channel's chat as the Streamer (for example, the Giveaway winner), the Result is kept, and the Streamer sees past Results in the History tab.

The UI is in Spanish; the Commands are in English.

### Success

This is a test project. It works when, with the Streamer's real Channel and the app running locally:

- A Chat Message shows up on the Dashboard in under 1 second (p95), with up to 20 Chat Messages per second in one Channel. This load is measured locally with fake events: at that rate a real Channel would use up ngrok's free monthly requests in about 17 minutes. The real Channel only confirms that everything works.
- A Chat Message that Twitch delivers twice counts once.
- A whole Voting Round and a whole Giveaway Round can be played, and their Results show up in History.
- It costs nothing to run.

## How it is built

All in docker-compose, behind one address: the Dashboard (React), the App Server and the Webhook Server (FastAPI, two copies each behind a Traefik load balancer), Redis and MongoDB, plus a reaper job that deletes unused Twitch subscriptions and validates the stored tokens. It is also a project to learn two things: OAuth done the way large apps do it (including storing and refreshing tokens), and distributed systems. See [Architecture](docs/architecture.md).

## How to run it locally

1. Create a Twitch app in the [Twitch developer console](https://dev.twitch.tv/console), with `http://localhost:8080/auth/callback` as its redirect URL.
2. Copy `.env.example` to `.env` and fill in the Twitch client id, the client secret, a random webhook secret and a random token encryption key. `.env` is never committed.
3. Start a tunnel that gives the Webhook Server a public HTTPS address: `ngrok http 8080` with the account's free dev domain, and put that address in `.env` as the webhook callback URL.
4. Run `docker compose up`.
5. Open the Dashboard in the browser, sign in with Twitch and write in the Channel's chat.

To test the Webhook Server without Twitch or the tunnel, send fake events with the [Twitch CLI](https://dev.twitch.tv/docs/cli/): `twitch event trigger channel.chat.message --forward-address http://localhost:8080/eventsub --secret <webhook secret>`.

To look inside MongoDB and Redis while debugging (with MongoDB Compass or Redis Insight), start the stack with the debug file: `docker compose -f docker-compose.yml -f docker-compose.debug.yml up`. It publishes MongoDB on `mongodb://localhost:27017` and Redis on `localhost:6379`, only on localhost. Without it, only port 8080 is published, and they are reached with `docker compose exec mongo mongosh` and `docker compose exec redis redis-cli`.

Use Chrome or Firefox locally: the session cookie is `Secure`, and Safari may not accept it on `http://localhost` (not verified).

The exact ports, paths and commands are added here when the code exists.

## How it is deployed

It is not deployed: for now it only runs locally. GitHub Actions runs the checks (lint, typecheck, tests) on every pull request. Hosting on AWS is decided later, when there is budget (see [Architecture](docs/architecture.md#hosting-later)).

## Documentation

- [Glossary](GLOSSARY.md)
- [Non-functional requirements](docs/non-functional-requirements.md)
- [Architecture](docs/architecture.md)
- [Data model](docs/data-model.md)
- [Decisions (ADR)](docs/adr/)
