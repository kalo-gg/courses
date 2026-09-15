# Designing a network layer

*Decide direction, reliability and frequency for every message before you write a line.*

Blink · Best Practices & Capstone · 9 min

---

Three questions, in this order, for every message you are about to add. Who is the authority for it? Does a missed one matter? How often does it go? The first answer is `From`, the second is `Type`, and the third decides `Call` and, more usefully, whether the message deserves an event of its own or belongs inside a snapshot that was going out anyway.

| Message | Authority | Miss matters? | Frequency | Verdict |
| --- | --- | --- | --- | --- |
| Player movement | client (input) | no, the next supersedes it | every frame | `From: Client`, `Unreliable`, `ManySync` |
| Damage dealt | server | yes | on hit | `From: Server`, `Reliable`, `SingleAsync` |
| Phase change | server | yes | four times a match | `From: Server`, `Reliable`, `ManyAsync` |
| Loadout request | client asks, server decides | yes | on spawn | a `function`, `Yield: Coroutine` |

Crosswire's four kinds of message. Everything else in the game is a variation on one of these rows.

**net/init.blink**

```blink
option ServerOutput = "../src/shared/Network/Server.luau"
option ClientOutput = "../src/shared/Network/Client.luau"
option TypesOutput = "../src/shared/Network/Types.luau"

import "./match"
import "./combat"
import "./entities"
```

The layout the table implies: one file per subsystem, one `init.blink` that sets the options and imports the rest, and a shared types file the subsystems import for themselves.

One compile, one pair of modules. Splitting a description across files does not split the remotes and does not add a byte: it splits the file you have to read. The event ids are still handed out in one sequence across the whole import graph.

## The limit nobody enforces

> **`Unreliable` has a 1,000-byte ceiling and Blink will not stop you**
>
> The events page states the limit. A definition with `Data: string(2000..3000)` on an `Unreliable` event compiles without a word of complaint. The failure happens at run time, inside Roblox, on a machine you are not sitting at.

So budget unreliable events yourself, with the same arithmetic as the last lesson and a margin. Count the block, count the worst-case length of every variable field, and keep the total well under a thousand. Bound the array rather than hoping: an unbounded list of positions is a packet that is fine in a play-test and gone in a full server.

**net/combat.blink**

```blink
struct Slot {
    id: u8,
    pos: vector<i16>
}

-- 1 id + 1 length + 8 slots of 7 bytes = 58 bytes at the cap.
event Positions {
    From: Server,
    Type: Unreliable,
    Call: ManySync,
    Data: Slot[..8]
}
```

The sandbox reports `block: Positions = 2 bytes` for this, because an array's contents are allocated after the block: the block is the floor and the bound is what makes the ceiling finite. Write the sum in a comment beside it; the next person to add a field will read it.

**Key points**

- Authority, then reliability, then frequency, then whether it should exist at all.
- Splitting a description into files splits the reading, not the remotes.
- `Unreliable` caps at 1,000 bytes and Blink does not check it: you do.
- Bound every array on an unreliable event, and write the arithmetic beside it.
