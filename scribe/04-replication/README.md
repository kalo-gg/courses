# 4. Replication & Commands

*Who receives a field, how the client reads it, and the command that is the only way a client changes anything.*

Scribe · Medium

---

Every field answers two questions: is it saved, and who sees it. This module wraps roots as ServerOnly, Shared and Session and shows what each costs, then builds the client half: the mirror that reads template defaults until the first snapshot lands, Observe as the whole of a reactive UI, and the optimistic write the server always overrules. Commands close the loop: a handler on the server, an argument spec that proves the shape before your code runs, and the three-value failure shape that tells a refusal from a handler's own answer.

## Lessons

1. [Who sees a field](01-visibility.md) — Every root answers two questions, saved or not and seen by whom; ServerOnly, Shared and Session set the answers.
2. [The client half](02-the-client.md) — The mirror reads defaults until the snapshot lands, Observe is the whole of a reactive UI, and a client write never leaves the client.
3. [Commands](03-commands.md) — Register a handler on the server, call it by name from the client, and let an Args spec prove the shape before your code runs.
4. [Requests and their failures](04-requests-and-failures.md) — Read the third return value, retry with the same key, and know the order a request is checked in.
