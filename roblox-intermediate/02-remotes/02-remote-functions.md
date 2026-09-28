# RemoteFunction

*A call that returns: the client asks, waits, and gets the server's answer. And why it never runs the other way.*

Roblox Intermediate · Remotes · 10 min

---

The shop screen opens and needs the player's rank, which only the server knows. With a RemoteEvent you would fire a request, listen for the reply on a second event, and match the two up.

A `RemoteFunction` is a call that returns. The client calls it, waits, and gets the server's answer as the return value.

## Ask and wait

**main.server.luau**

```luau
local ReplicatedStorage = game:GetService("ReplicatedStorage")

local getCoins = Instance.new("RemoteFunction")
getCoins.Name = "GetCoins"
getCoins.Parent = ReplicatedStorage

local coins = {} -- User.Id -> coins, kept on the server

getCoins.OnServerInvoke = function(player: Player)
	return coins[player.User.Id] or 0
end
```

`OnServerInvoke` is a callback you **assign**, not an event you connect. Like `OnServerEvent`, it gets the calling player first.

**main.client.luau**

```luau
local ReplicatedStorage = game:GetService("ReplicatedStorage")

local getCoins = ReplicatedStorage:WaitForChild("GetCoins")
local amount = getCoins:InvokeServer() -- this thread waits here
print("You have " .. amount .. " coins")
```

`InvokeServer` yields for the whole round trip: the request, the callback's run, and the reply. What the callback returns comes back as `InvokeServer`'s results.

Because the caller waits, use a RemoteFunction only when you need the answer. The docs recommend a RemoteEvent whenever the result is not needed, since firing never waits. Invoke where a pause is fine, such as opening a menu, and never once a frame.

|  | RemoteEvent | RemoteFunction |
| --- | --- | --- |
| Client asks the server | `FireServer` raises `OnServerEvent` | `InvokeServer` runs `OnServerInvoke` |
| The caller waits | No | Yes, for the whole round trip |
| An answer comes back | No | The callback's return values |
| Handlers | Any number of connections | One callback; the last assignment wins |
| Server to client | `FireClient`, `FireAllClients` | `InvokeClient`: avoid it |

## Never invoke a client

A RemoteFunction also has `InvokeClient`, which makes the server wait on a client's `OnClientInvoke`. The docs list what can go wrong. If the client errors, the server throws the error too. If the client leaves mid-call, `InvokeClient` throws. If the client never returns, the server waits forever.

The client controls its own callback, so an exploiter can pick the worst of those on purpose. Send to a client with `FireClient`. If you need a reply, the client can fire one back, and the server treats it like any other request.

**Key points**

- `InvokeServer(...)` yields until the server's `OnServerInvoke(player, ...)` returns, and hands back what it returned.
- `OnServerInvoke` is one assigned callback: assign it twice and only the last one runs.
- Use a RemoteEvent when you do not need an answer, so nothing waits.
- Never call `InvokeClient`: a client can make the server error or wait forever. Use `FireClient`.
