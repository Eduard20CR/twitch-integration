# ADR-0002 — Chat Messages reach the Dashboard through Redis Pub/Sub, with no replay

Status: Accepted

The Webhook Server publishes each Chat Message to the Streamer's Redis Pub/Sub channel, and the App Server forwards it to the Dashboard. Pub/Sub keeps nothing, so the Chat Messages that arrive while the Dashboard is reconnecting or reloading are lost, and a few Commands (for example a `!vote`) may not count. We accept this in the first version to keep the design simple. Revisit it if Streamers report lost votes or entries: then move to Redis Streams, so a Dashboard that reconnects asks for the Chat Messages it missed.
