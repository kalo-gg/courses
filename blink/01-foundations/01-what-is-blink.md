# What is Blink?

*An IDL compiler: you describe your network traffic, it writes the buffer code for you.*

Blink · Blink Foundations · 9 min

---

A `RemoteEvent` accepts whatever table you hand it. Nothing in its API says what a given remote carries, so the only way to find out is to read every `FireServer` call in the repository and hope you found them all. Roblox then serialises that table for you, and it is generous with bytes: a boolean does not cost a bit, and a field name you send sixty times a second is sent sixty times a second.

**crosswire.blink**

```blink
option ClientOutput = "./Client.luau"
option ServerOutput = "./Server.luau"

event Damage {
    From: Server,
    Type: Reliable,
    Call: SingleAsync,
    Data: u8
}
```

The whole description: where to write the two modules, and one event with four fields.

Blink reads that file and writes two Luau modules, one for each side. They already know the shape, so they pack the fields into a buffer instead of a table; they already know the bounds, so they reject what arrives malformed; and they already batch, so a hundred calls in one frame leave as one send.

**Server.luau (generated, read-only)**

```luau
local Returns = table.freeze({
	StepReplication = StepReplication,

	Damage = {
		Fire = function(Player: Player, Value: number): ()
			Load(PlayersMap[Player])
			SerdesFunctions.WriteEVENT_Damage(Value)
			PlayersMap[Player] = Save()
		end,
		FireAll = function(Value: number): () ... end,
		FireList = function(List: {Player}, Value: number): () ... end,
		FireExcept = function(Except: Player, Value: number): () ... end,
	},
})
```

Three of the four bodies elided. You did not write any of this, and you never edit it.

## What Blink is not

|  | What it is | Relation to Blink |
| --- | --- | --- |
| `RemoteEvent` | Roblox's own remote, sends Luau values | Blink creates **one `RemoteEvent` and one `UnreliableRemoteEvent`** for the whole game and sends buffers through them. It creates no `RemoteFunction` at all, even for a `function` declaration. |
| Zap | A different networking IDL with the same idea | A real alternative, compared honestly in the lesson "Blink or Zap". |
| ByteNet | A runtime-defined buffer networking library | No compiler and no file: you write packet definitions in Luau. |

An IDL rather than a library, because the description is then one file a reviewer can read end to end, and the cost of a field is decided once, in writing, instead of at each call site. The project's own benchmark (2025-04-30, against blink v0.17.1) puts its Entities bench at 41.81 Kbps against 559,364 Kbps for plain remotes. Zap and ByteNet land within a percent of Blink there, at 41.71 and 41.64, because all three are doing the same thing.

> **Blink is Luau, not Rust**
>
> The compiler is written in Luau and shipped as a standalone binary built by Lune. That is why the Studio plugin can run the *same* compiler inside Studio, and why the exercises on this page compile for real in your browser. (The two projects are the exception: they are graded on the text of the files, and nothing runs.)

**Key points**

- A `.blink` file is an interface description: each event names its direction, reliability, listener style and payload.
- The compiler emits one Luau module per side; they pack, validate and batch, and you never edit them.
- One RemoteEvent and one UnreliableRemoteEvent carry the whole game, and no RemoteFunction is ever created.
- The saving is in the bytes, not in magic: Zap and ByteNet reach the same figures by doing the same work.
