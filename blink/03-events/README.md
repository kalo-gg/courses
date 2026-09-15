# 3. Events & Functions

*Four fields describe an event, five call modes decide its API, and functions get you an answer back.*

Blink · Medium

---

An event is a handful of fields: who fires it, how reliable it is, what kind of listener the other side gets, and what rides along. Getting the third one wrong produces the strangest bugs, so this module spends a lesson on it. Functions are the other half — the client asks, the server answers — and they come with a choice of how the waiting is written. The module ends with the first piece of Crosswire's network layer.

## Lessons

1. [The fields of an event](01-event-fields.md) — Three required fields describe an event; Data and the undocumented Poll are the optional two.
2. [Sync, async, many and polling](02-call-modes.md) — Five call modes, one of which will corrupt a frame if your listener yields.
3. [Firing and listening](03-firing-and-listening.md) — Four ways to send from the server, one from the client, and a Player argument you did not write.
4. [Functions: asking the server](04-functions.md) — The client invokes, the server answers, and Yield picks how the waiting is written.
