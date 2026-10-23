# Typed remotes

*Declare a remote once, fire and handle it from both sides, and let attributes limit, validate and time out what crosses.*

Alloy · Alloy on Roblox · 11 min

---

A Luau remote is an Instance you make by hand, look up by a string name, and hand untyped arguments. Nothing stops a client from firing a server-only event, and a typo in the name is found at run time.

An Alloy `remote` is one typed declaration that both sides import. If remotes are new to you, Roblox Intermediate's remotes lesson covers the engine side.

## Declare once, use on both sides

**src/shared/net.aly**

```alloy
--- A client asks the server to damage a player.
export remote Damage(target: Player, @u16 amount: number) from client

--- A client asks for its coin count; the server answers.
export remote function GetCoins() -> number from client

--- The server tells every client about a round.
export remote Announce(text: string) from server
```

`from client`, `from server` or `from client or server` names who fires. A `remote function` returns an answer.

**src/server/main.server.aly**

```alloy
import { Damage, GetCoins, Announce } from '@shared/net'

Damage.on(function(sender, target, amount)
  print(`{sender.Name} hits {target.Name} for {amount}`)
end)

GetCoins.on(function(sender) -> number
  return sender:GetAttribute('Coins') ?? 0
end)

Announce.fire_all('Round 2 starts')
```

`'@shared/net'` reaches `src/shared/net.aly` through the `shared` mount that `alloy init` writes (Module 7). A server handler gets the sender first, typed `Player`, then the parameters as declared. On the client, `Damage.fire(target, 20)` sends, and `await GetCoins.call()` waits for the answer.

The declaration emits one `__alloy.remote({ … })` call on its own line. The runtime makes the RemoteEvent the first time the remote is used. In a `.server.aly` or `.client.aly` file the compiler also checks direction: a server file that fires a `from client` remote is an error, because it knows which side it is on. A shared file runs on both sides, so it gets no report.

| Member | Who calls it | What it does |
| --- | --- | --- |
| `fire(…)` | Either side | From a client, sends to the server. From the server, sends to the player passed first. |
| `fire_all(…)`, `fire_except(p, …)` | Server | Sends to every client, or to all but one |
| `on(h)`, `once(h)` | The receiving side | Handles each event, or the next one, and returns the connection |
| `call(…)` | The firing side of a `remote function` | Yields a `Future` of the answer |
| `wait()` | The receiving side | A `Future` of the next event |
| `on_ratelimited(h)` | Server | Hears a sender that `@ratelimit` refused |
| `spec`, `instance` | Either side | The declaration, and the Roblox object behind it |

```alloy
@ratelimit(5, 1)
@validate(function(sender, amount) return amount > 0 end)
export remote Heal(@u8 amount: number = 10) from client

@unreliable
export remote Aim(@f32 yaw: number) from client

@timeout(5)
export remote function Trade(offer: string) -> boolean from client
```

`@ratelimit(5, 1)` allows each player 5 events a second. `@validate` drops an event before the handler sees it. A default makes `amount` optional. `@unreliable` uses an UnreliableRemoteEvent, and `@timeout(5)` rejects a `call` that gets no answer in 5 seconds.

Remote parameters carry data only. A function, a `thread`, a `Future` or a `Signal` cannot cross the network, so it is a compile error, `error(4.3): WireType`. Send an id or a name, and keep the callback on the side that owns it.

> **Recognising older code**
>
> Luau code makes the instance itself: `local damage = Instance.new('RemoteEvent')`, then a `Name`, a `Parent`, and `OnServerEvent` with untyped arguments. The style guide asks for a `remote` declaration instead.

**Key points**

- `remote Name(params) from client` (or `from server`, or `from client or server`) declares a typed channel both sides import.
- A server handler gets the sender first. `call` on a `remote function` yields a `Future`.
- `@ratelimit`, `@validate`, `@timeout`, `@unreliable` and `@immediate` configure a remote where it is declared.
- A function, thread, `Future` or `Signal` parameter is `WireType`: a remote carries only data.
