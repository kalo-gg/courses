# Testing with many players

*Server & Clients for bugs between players, the Network Simulator for bad connections, and a checklist for both.*

Roblox Intermediate · Shipping multiplayer · 10 min

---

In a solo Test, pinging a spot shows a marker every time. On the live server, players report that pings "sometimes do nothing". Nobody can reproduce it alone, because the bug needs a second player: the cooldown is one variable, and anyone's ping resets it for everyone.

A solo Test runs one server and one client. Bugs between players need more clients.

## Server & Clients

Pick **Server & Clients** in Studio's test modes, choose how many clients (the docs say one or two is usually enough, and up to eight are allowed), and start. Studio opens one session for the server and one per client. Act in one client and watch the other, and read the server's Output, where every client's requests land.

```luau
-- Solo: correct. With two clients, both read "You bought the Sniper"
Remotes.Notify:FireAllClients("You bought the " .. name)

-- Only the buyer
Remotes.Notify:FireClient(player, "You bought the " .. name)
```

With one client, *every client* and *this client* are the same player, so a solo Test cannot tell these apart.

## A bad connection, on purpose

Studio's local test has almost no latency, which hides a whole class of bugs. The **Network Simulator**, a Studio beta, adds latency, jitter and packet loss to the playtest connection, separately for each direction. Run the same action on the default preset and on a slow one, and compare what the player sees.

| Preset | Server to client | Client to server |
| --- | --- | --- |
| Wired Broadband | 25 ms, 3 ms jitter, 0% loss | 25 ms, 3 ms jitter, 0% loss |
| Standard Mobile (4G/LTE) | 45 ms, 20 ms jitter, 0.40% loss | 55 ms, 30 ms jitter, 0.50% loss |
| Bad Connection (3G) | 150 ms, 70 ms jitter, 0.50% loss | 180 ms, 90 ms jitter, 0.50% loss |

From the docs' "Network Simulator" page. It needs the New Device Simulator beta; open Device Simulator from the Test menu, then the Network pill above the viewport. It affects Studio playtests only.

> **A multiplayer checklist**
>
> - Two clients do the same thing at the same moment.
> - A player joins after the round has started.
> - A player leaves mid-action, and the others carry on.
> - A player respawns, then does it all again.
> - Every action that waits on the server, on a slow preset.

```luau
Ping.OnServerEvent:Connect(function(player, position)
	print(`[Ping] {player.Name} at {position}`)
	-- ...
end)
```

The server's Output shows every client's requests in one place. Name the player in each line, and a request refused right after another player's stands out.

Some bugs need none of this. A damage formula or a cooldown table is plain Luau in a ModuleScript, and the Lest course shows how to test modules like that without starting a server.

**Key points**

- A solo Test is one server and one client: bugs between players need Server & Clients.
- Use one or two clients for most bugs; Studio allows up to eight.
- The Network Simulator (a Studio beta) adds latency, jitter and loss per direction, from presets or custom values.
- State shared by every player, messages sent to everyone, and cleanup on leave are the usual two-player bugs.
- Test pure logic in ModuleScripts without a server, as the Lest course shows.
