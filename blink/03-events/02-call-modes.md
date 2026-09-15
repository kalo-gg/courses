# Sync, async, many and polling

*Five call modes, one of which will corrupt a frame if your listener yields.*

Blink · Events & Functions · 10 min

---

`Call` decides the API the receiving side gets, and it is the field people set by habit and regret later. Four of the five values are one two-by-two choice: one listener or many, and may the listener yield or not. The fifth, `Polling`, hands you a queue instead of a callback.

| `Call` | Listeners | May yield | API |
| --- | --- | --- | --- |
| `SingleSync` | one (last wins) | **no** | `On(fn)` |
| `SingleAsync` | one (last wins) | yes | `On(fn)` |
| `ManySync` | many | **no** | `On(fn)` |
| `ManyAsync` | many | yes | `On(fn)` |
| `Polling` | none | — | `Iter()` |

**net.blink**

```blink
event Chat {
    From: Client,
    Type: Reliable,
    Call: ManyAsync,
    Data: string(1..140)
}

event Snapshot {
    From: Server,
    Type: Unreliable,
    Call: Polling,
    Data: vector
}
```

`Chat` compiles to `server: Chat.On = function(Listener: (Player: Player, Value: string) -> ()): () -> ()`; `Snapshot` to `client: Snapshot.Iter = function(): () -> (number, Vector3)`.

Read "sync" as a promise, not as a performance setting. A sync listener runs on the thread that is deserialising the packet, so declaring `Sync` is you telling Blink that this listener returns without yielding. It is not faster code; it is code that skips one `task.spawn` because you said it was safe to.

> **What a sync yield actually costs**
>
> If a sync listener yields, the handler for the **next** packet notices the parked thread, warns, and closes it, so the rest of that first packet is never deserialised. The warning is `[Blink]: Yielded in a Sync call. Event "<name>" uses a Sync call, consider using an Async call instead.` Blink's own docs put it bluntly: "Yielding or erroring in sync event can cause undefined and sometimes game-breaking behaviour." Default to `Async` and reach for `Sync` when a profile tells you to.

## Polling

`Call: Polling` gives the receiving side no callback at all. Incoming payloads queue, and you drain the queue with `Iter()` whenever you like. That suits a system that already runs on a schedule: the work happens where you decided it should, inside your own update, instead of wherever the packet happened to land. Each iteration yields the queue index and then your `Data`, in order.

**client.luau**

```luau
local Net = require(ReplicatedStorage.Network.Client)

RunService.Heartbeat:Connect(function()
	for index, position in Net.Snapshot.Iter() do
		applySnapshot(position)
	end
end)
```

`Iter()` returns an iterator, so it goes straight into a generic `for`. The queue is empty again once the loop ends.

## Your turn

**Key points**

- `Single*` keeps one listener and a second connection silently replaces the first; `Many*` keeps a list.
- `Sync` is a promise that your listener will not yield, not a speed setting.
- A sync listener that yields is warned about and killed, and the rest of that packet is lost.
- `Polling` replaces `On` with `Iter()`, so the work happens inside your own loop.
- When in doubt, `ManyAsync`.
