# Reading what it generated

*One frozen table per side, and the method names tell you which side you are on.*

Blink · Blink Foundations · 10 min

---

Each generated module returns a single `table.freeze`d table whose keys are the names you declared. What hangs under a key is not the same on both sides: the sending side gets ways to send and the receiving side gets a way to listen, so the method you can reach is itself the answer to which side of the wire you are reading.

**Server.luau (generated, read-only)**

```luau
local Returns = table.freeze({
	StepReplication = StepReplication,

	DamageTaken = {
		Fire = function(Player: Player, Value1: Player, Value2: number): () ... end,
		FireAll = function(Value1: Player, Value2: number): () ... end,
		FireList = function(List: {Player}, Value1: Player, Value2: number): () ... end,
		FireExcept = function(Except: Player, Value1: Player, Value2: number): () ... end,
	},
	Move = {
		On = function(Listener: (Player: Player, Value: Vector3) -> ()): () -> () ... end
	},
})
```

Bodies elided. The server gets four ways to send a server to client event, and one way to hear a client to server one.

| Event direction | Server module | Client module |
| --- | --- | --- |
| `From: Server` | `Fire(player, …)`, `FireAll(…)`, `FireList({players}, …)`, `FireExcept(player, …)` | `On(listener)` |
| `From: Client` | `On(listener)`, whose first argument is the `Player` | `Fire(…)` |
| `function` | `On(listener)`, first argument the `Player`, and it **returns** a value | `Invoke(…)` |

## Connecting, and disconnecting

`On` hands back a function that disconnects the listener, so a system that stops caring can stop listening without reaching into the module. What the module does with your listener depends on the `Call` mode: a `Single*` event keeps exactly one, in a slot, while a `Many*` event keeps a list and appends to it.

> **Connecting twice to a Single event replaces the first listener**
>
> `SingleSync` and `SingleAsync` compile to `Events.Reliable[0] = Listener`, which overwrites. No error, no warning, and the first listener simply stops running. `ManySync` and `ManyAsync` `table.insert` instead. If two systems both care about one event, the event is a `Many*` one.

**combat.client.luau**

```luau
local Net = require(ReplicatedStorage.Network.Client)

local disconnect = Net.DamageTaken.On(function(target: Player, amount: number)
	print(target.Name, "took", amount)
end)
```

Hand-written, on the consumer side. The argument names and types came from the description.

You never edit either module. Both carry `-- This file is not meant to be edited` under a header naming the version that wrote them, and both are overwritten from scratch on the next compile. A fix belongs in the `.blink` file, which is the only input.

## Your turn

**Key points**

- A generated module is one frozen table keyed by your declared names.
- The sending side gets `Fire` and friends; the receiving side gets `On`, which returns a disconnect function.
- A `From: Client` event's server listener receives the firing `Player` as its first argument.
- Connecting twice to a `Single*` event silently replaces the first listener.
- Neither generated file is editable: they are rewritten on every compile.
