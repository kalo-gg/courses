# Latency and feel

*Show the shot on the client at once, let the server decide what it hit, and give every client the same clock.*

Roblox Intermediate · Shipping multiplayer · 11 min

---

At 150 ms of ping, a blaster that waits for the server before showing anything feels broken: click, and nothing for a sixth of a second. Let the client decide the hit instead and it feels instant, and an exploiter hits anyone, through any wall.

So split the work. The client shows the shot at once: tracer, sound, flash. The server decides what the shot did.

## Show it now, decide on the server

**The blaster's client code**

```luau
local function fire()
	local mouse = UserInputService:GetMouseLocation()
	local ray = camera:ViewportPointToRay(mouse.X, mouse.Y)
	local result = workspace:Raycast(ray.Origin, ray.Direction * RANGE, params)
	local hitPosition = if result then result.Position else ray.Origin + ray.Direction * RANGE

	drawTracer(character.Head.Position, hitPosition) -- the player sees it now
	if result then
		Shoot:FireServer(result.Instance, result.Position) -- the server decides
	end
end
```

`params` excludes the player's own character. The tracer is cosmetic, so drawing it before the server answers risks nothing. The damage waits for the server.

The server could repeat the whole shot with its own raycast. But by the time the request arrives, the target has moved on from where the shooter saw it, so on a slow connection fair shots would miss. The docs' client-server guide checks the client's claimed hit instead, with a **tolerance** for that movement, and from **its own** view of the shooter.

| The server checks | Why |
| --- | --- |
| A `BasePart` and a finite `Vector3` arrived | The client can send anything, NaN included |
| The claimed point is within 4 studs of that part, on the server | The target moved since the shooter saw it |
| Its own ray, from its own copy of the shooter's head to the point, hits no map part | No shots through walls. It tests only static geometry, because characters move |
| Then it applies damage from its own constant | The client asks; the server decides the outcome |

> **Server authority, the engine feature**
>
> Roblox is building server authority into the engine: with `Workspace.AuthorityMode` set to `Server`, the server simulates physics and movement, and clients predict and roll back to hide latency. It is in beta, and the docs say it will be released soon. This course will be updated to teach it when it ships. Until then, checking on the server by hand, as here, keeps hits honest.

## One clock for every client

```luau
-- Server: the round ends 90 seconds from now, for everyone
workspace:SetAttribute("RoundEndsAt", workspace:GetServerTimeNow() + 90)

-- Client: count down to that moment
local left = workspace:GetAttribute("RoundEndsAt") - workspace:GetServerTimeNow()
timerLabel.Text = tostring(math.max(0, math.ceil(left)))
```

Send *when*, not *how long*. The attribute reaches a laggy client late, but the moment it names is the same for every player.

`workspace:GetServerTimeNow()` is each client's estimate of the server's clock in Unix seconds, so every client gets roughly the same answer. It never goes backwards.

The estimate lives on the client, so never let a client's timestamp grant anything. The docs say to time rewards on the server.

**Key points**

- Show cosmetic effects on the client at once; let the server decide what a shot did.
- The client sends what it hit; the server checks it against its own view, with a tolerance for movement.
- Test walls with the server's own ray from its copy of the shooter, against static geometry only.
- `workspace:GetServerTimeNow()` gives every client the same clock: send an end time, not a duration.
- Engine server authority (`Workspace.AuthorityMode`) is in beta; until it ships, validate by hand.
