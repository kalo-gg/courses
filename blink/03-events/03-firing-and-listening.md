# Firing and listening

*Four ways to send from the server, one from the client, and a Player argument you did not write.*

Blink · Events & Functions · 10 min

---

The server gets four ways to send a `From: Server` event and the client gets one way to hear it. Turn the arrow round and the shapes swap. The rule that catches everyone is on the listening end of a `From: Client` event: the server's listener receives the `Player` as its **first** argument, inserted by Blink, and your `Data` follows after it. Nothing in the definition says so.

**match.server.luau**

```luau
local Net = require(ReplicatedStorage.Network.Server)

-- From: Client: the Player comes first, then your Data
Net.Hit.On(function(player: Player, amount: number)
	applyDamage(player, amount)
end)

-- From: Server: four ways out
Net.KillFeed.FireAll(killer, victim)
Net.KillFeed.FireExcept(killer, killer, victim)
```

## Why FireAll is not a loop

**Server.luau (generated, read-only)**

```luau
FireAll = function(Value1: Player, Value2: number): ()
    Load()
    SerdesFunctions.WriteEVENT_DamageTaken(Value1, Value2)
    local Buffer, Size, Instances = SendBuffer, SendCursor, SendInstances
    for _, Player in Players:GetPlayers() do
        Load(PlayersMap[Player])
        local Position = Allocate(Size)
        buffer.copy(SendBuffer, Position, Buffer, 0, Size)
        ...
    end
end
```

Bodies elided. The payload is written once, then copied.

The serialisation happens once, at the top, and the loop only copies those bytes into each player's own outgoing buffer. Every player has one, and it is flushed on `Heartbeat` by `StepReplication`. So `FireAll` is one pack and N memory copies, not N packs, which is why hand-written loops over `Fire` are slower than the thing they are replacing.

## Type packs

`Data` takes one type, or a parenthesised comma-separated pack. A pack becomes positional arguments on both sides: `Data: (u8, u16)` gives you `Fire(a, b)` and `On(function(a, b) end)`, with the `Player` still in front on a server listener. The generated names are `Value1`, `Value2` and so on, which is a good reason to keep packs short and reach for a `struct` once there are more than three.

**Key points**

- A `From: Server` event gives the server `Fire`, `FireAll`, `FireList` and `FireExcept`, and the client `On`.
- A `From: Client` event's server listener gets the `Player` first, then your `Data`.
- `FireAll` serialises once and copies per player, so it beats a loop over `Fire`.
- A parenthesised type pack becomes positional arguments on both sides.
