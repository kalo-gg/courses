# Pathfinding

*Compute a route round obstacles, walk an NPC along it with `MoveTo`, and compute again when it is blocked.*

Roblox Intermediate · The world · 11 min

---

A guard hears the alarm and should run to it. `humanoid:MoveTo(alarm.Position)` sends it in a straight line, into the wall between them, where it walks in place until it gives up.

**PathfindingService** finds a way round. It computes a route as a list of waypoints, for an agent of the size you describe. Walking that route is still your job.

## Computing a path

```luau
local PathfindingService = game:GetService("PathfindingService")

local path = PathfindingService:CreatePath({
	AgentRadius = 2,     -- how wide the agent is
	AgentHeight = 5,     -- how tall
	AgentCanJump = true, -- may the route include jumps?
})

local ok = pcall(function()
	path:ComputeAsync(guard.HumanoidRootPart.Position, alarm.Position)
end)

if ok and path.Status == Enum.PathStatus.Success then
	for _, waypoint in path:GetWaypoints() do
		print(waypoint.Position, waypoint.Action)
	end
end
```

`ComputeAsync` yields and can error, so it goes in a `pcall`. Even when it returns, `Status` says whether a route exists: `Success` or `NoPath`. The enum's other items are deprecated.

Walk a route with `humanoid:MoveTo(waypoint.Position)`, one waypoint at a time. `MoveToFinished` fires with `true` on arrival, or `false` if the humanoid did not get there within **8 seconds**. The limit is per call, and waypoints are 4 studs apart by default (`WaypointSpacing`), so one call per waypoint stays well inside it.

A waypoint's `Action` says how to reach it. When it is `Jump`, set `humanoid.Jump = true` as you move towards it.

## Walking it, and computing again

```luau
local waypoints = path:GetWaypoints()
local nextIndex = 1

humanoid.MoveToFinished:Connect(function(reached)
	if reached and nextIndex < #waypoints then
		nextIndex += 1
		local waypoint = waypoints[nextIndex]
		if waypoint.Action == Enum.PathWaypointAction.Jump then
			humanoid.Jump = true
		end
		humanoid:MoveTo(waypoint.Position)
	end
end)

local blockedConnection
blockedConnection = path.Blocked:Connect(function(blockedIndex)
	if blockedIndex >= nextIndex then -- ahead of the guard, not behind it
		blockedConnection:Disconnect()
		-- compute again from where the guard stands now
	end
end)

humanoid:MoveTo(waypoints[nextIndex].Position)
```

`Blocked` passes the index of the blocked waypoint. It also fires for blockages **behind** the agent, so compute again only when the index is at or after the waypoint it is heading for.

A route goes stale when the world changes: a door shuts, a crate falls. The path fires `Blocked`, and computing again from where the agent stands gives it a fresh route.

The server sees the whole world, so it is the natural home for NPC pathfinding. Client code may fail to compute a path to something that has not streamed in.

> **A player's own character**
>
> The pathfinding guide steers a *player's* character along the waypoints from a client script with `Humanoid:Move`, which takes a direction rather than a goal. This lesson drives an NPC from the server with `MoveTo`, which walks to a point on its own.

**Key points**

- `CreatePath` describes the agent; `ComputeAsync` (in a `pcall`) finds the route; `Status` says whether it exists.
- `GetWaypoints` is empty when there is no path, so check `Status` first.
- Walk one waypoint per `MoveTo`; each call gives up after 8 seconds, and a `Jump` waypoint needs `humanoid.Jump = true`.
- `Blocked` also fires behind the agent: compute again only when the index is at or after the next waypoint.
