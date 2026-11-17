# Twitch Integration

A web app where a Streamer runs live games with the Viewers of their Twitch Channel, who take part only by writing in the Channel's chat.

## Language

### People

**Streamer**:
The person who owns a Channel and is the only one who signs in to the app.
_Avoid_: User, broadcaster, host

**Viewer**:
A person who watches a Channel and takes part in games only by writing in its chat. A Viewer never uses the app.
_Avoid_: User, player, participant, chatter

### The app

**Dashboard**:
The page where the Streamer sees the live chat and plays the Games, and that they show on their stream.
_Avoid_: Panel, overlay, page, app

**Active Dashboard**:
The one Dashboard of a Streamer that receives the chat right now; when the Streamer opens another, the newest one takes over.
_Avoid_: Main tab, current session

### Twitch

**Channel**:
The Twitch channel of one Streamer, where the Viewers write in the chat. One Channel has exactly one Streamer.
_Avoid_: Stream, room

**Chat Message**:
One message a Viewer writes in a Channel's chat.
_Avoid_: Event, notification, comment

**Command**:
A Chat Message that starts with `!` and that a Game understands, such as `!vote A` or `!join`.
_Avoid_: Instruction, keyword

### Games

**Game**:
A kind of game the Streamer can play with the Viewers, such as Voting or Giveaway.
_Avoid_: Mini-game, activity, tab

**Round**:
One time a Game is played, from when the Streamer starts it until it ends. A Streamer has at most one Round going at a time.
_Avoid_: Match, session, game

**Result**:
What is kept when a Round ends, such as the winning option or the winner.
_Avoid_: Score, outcome, history entry

**History**:
The list of a Streamer's past Results, newest first, shown in its own tab of the Dashboard.
_Avoid_: Log, past games, records

**Voting**:
A Game where Viewers vote for one of several options with `!vote`.
_Avoid_: Poll

**Giveaway**:
A Game where Viewers enter with `!join` and one of them is picked at random as the winner.
_Avoid_: Raffle, draw, lottery
