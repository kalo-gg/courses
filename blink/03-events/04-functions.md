# Functions: asking the server

*The client invokes, the server answers, and Yield picks how the waiting is written.*

Blink · Events & Functions · 10 min

---

A `function` declaration stands in for a `RemoteFunction`: client to server only, with a value coming back. Blink creates no `RemoteFunction` to do it. An invocation rides the same reliable remote as every event, tagged with a `u8` identifier so the reply can find the call that is waiting for it. Three fields describe one: `Yield`, `Data` and `Return`, and only the first is required.

**loadout.blink**

```blink
struct Loadout {
    Weapon: string(1..24),
    Ammo: u8(0..60)
}

function RequestLoadout {
    Yield: Coroutine,
    Data: u8,
    Return: Loadout
}
```

| `Yield` | Client call site | Needs |
| --- | --- | --- |
| `Coroutine` | `local v = Net.RequestLoadout.Invoke(1)`, which yields the caller | nothing |
| `Future` | `Net.RequestLoadout.Invoke(1):Await()` | `option FutureLibrary = "…"` |
| `Promise` | `Net.RequestLoadout.Invoke(1):await()` | `option PromiseLibrary = "…"` |

There is no `From`, no `Type` and no `Call` on a function, and writing one is an error: the direction is fixed, the transport is always reliable, and the server side is always a single handler that returns. `Data` and `Return` are both optional, so a function with neither is a legal round trip that carries nothing but the fact that it happened.

> **Future and Promise need their option**
>
> `Yield: Future` without `option FutureLibrary` is not a Blink diagnostic. It is a bare Luau error with a stack trace: `Cannot use yield type: "Future", without providing a path to the future library.` You will not get a caret and a code, so recognise it by the `[Stack Begin]` block. Declare the option in the same file, even while the path is still a placeholder.

## The server side

**Server.luau (generated, read-only)**

```luau
RequestLoadout = {
    On = function(Listener: (Player, number) -> ({ Weapon: string, Ammo: number })): ()
        ...
        local Success, Error = pcall(function ()
            ReturnValue = Listener(Player, Value)
        end)
```

Your handler runs inside a `pcall`, so an error in it becomes a failed invocation rather than a dead remote.

That `pcall` is why a mistake in the handler does not hang the client forever. The invocation comes back failed, and the client raises `There was an exception while processing "RequestLoadout".` You lose the original error text, so a handler worth debugging should do its own `pcall` and return something the caller can read.

> **256 invocations in flight**
>
> The client keeps a 256-slot table keyed by a `u8` identifier, one slot per invocation waiting for its reply. The 257th errors with `More than 256 calls are awaiting a response, this packet has been dropped.` Fire-and-forget invocation in a loop is the way to meet it.

## Your turn

**Key points**

- A `function` is client to server with a return value, and Blink creates no `RemoteFunction` for it.
- Its fields are `Yield`, `Data` and `Return`; there is no `From`, `Type` or `Call`.
- `Yield` changes only the client's call site: `Coroutine`, `Future` or `Promise`.
- `Future` and `Promise` need their library option, and failing to give it is a bare Luau error.
- At most 256 invocations may be waiting at once.
