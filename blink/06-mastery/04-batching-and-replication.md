# Batching and manual replication

*Blink batches to the end of the frame, and ManualReplication hands you the flush.*

Blink · Best Practices & Capstone · 10 min

---

Nothing is sent when you call `Fire`. The payload is written into a per-player buffer and left there, and a single `Heartbeat` connection flushes every buffer once per frame. That is why `FireAll` serialises once and copies the result, and why a hundred `Fire` calls in one frame become one `FireClient` carrying one buffer.

**Server.luau (generated)**

```luau
local function StepReplication()
    for Player, Send in PlayersMap do
        if Send.Cursor <= 0 then
            continue
        end

        local Buffer = buffer.create(Send.Cursor)
        buffer.copy(Buffer, 0, Send.Buffer, 0, Send.Cursor)
        Reliable:FireClient(Player, Buffer, Send.Instances)

        Send.Size = 64
        Send.Cursor = 0
        Send.Buffer = buffer.create(64)
        table.clear(Send.Instances)
    end
end
RunService.Heartbeat:Connect(StepReplication)
```

One connection, one pass per player per frame. The last line is the whole of Blink's scheduling.

**The same tail with `option ManualReplication = true`**

```luau
local function StepReplication()
    for Player, Send in PlayersMap do
        -- ... identical ...
    end
end
Reliable.OnServerEvent:Connect(function(Player: Player, Buffer: buffer, Instances: {Instance})
```

The function is still there and still exported. The `Heartbeat` line is not, and that is the entire difference the option makes.

> **`StepReplication` is always exported**
>
> The documentation says `ManualReplication` exposes `StepReplication` instead of replicating automatically. The returned table carries it either way; the option only removes the `Heartbeat` connection. Verified by diffing the generated module with and without it.

So when should you take the flush? Two cases. A fixed-step simulation that wants its sends aligned to its own tick rather than to Heartbeat, and a server that runs several systems and wants all of their output in one flush at the end of its own frame. Outside those, the automatic connection is already doing the thing you would have written.

## A name you cannot use

Because `StepReplication` is in the returned table of every generated module, it is a reserved identifier in the language. Name an event or a type that and the compile stops. This is worth knowing because the error arrives at a name you probably chose on purpose, and the fix is to rename yours rather than to argue with it.

> **[E3005] Error: Reserved identifier**
>
> `"StepReplication" is reserved and cannot be used as an identifier.` The diagnostic points at your declaration, and it fires whether or not `ManualReplication` is set, because the function is exported either way.

**Key points**

- `Fire` writes into a buffer; a single `Heartbeat` connection sends them all once a frame.
- `FireAll` serialises once and copies, which is why broadcasting is cheap.
- `ManualReplication` removes the connection and nothing else; `StepReplication` is exported either way.
- Take the flush for a fixed-step clock or an end-of-frame batch, and for nothing else.
- `StepReplication` is a reserved identifier: `[E3005]` if you name a declaration that.
