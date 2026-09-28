# What crosses the wire

*Every argument arrives as a copy, and some arrive as nothing at all. Send plain data the other side can use.*

Roblox Intermediate · Remotes · 11 min

---

A building tool shows a see-through ghost block where the player is aiming. The client made the ghost, and on **Place** it sends it: `placeBlock:FireServer(ghost)`. The server's handler gets `nil`, and nothing errored anywhere.

The remote works fine. Arguments do not travel as themselves: each one is copied across, and some things have no copy on the other side.

## What arrives

**main.client.luau**

```luau
local ghost = Instance.new("Part") -- made on this client: the server has no copy
ghost.Parent = workspace

local Stats = {}
Stats.__index = Stats
function Stats:isAlive()
	return self.hp > 0
end
local stats = setmetatable({ hp = 100 }, Stats)

probe:FireServer(
	ghost, -- nil
	workspace.Baseplate, -- the server's Baseplate
	stats, -- a copy, { hp = 100 }, with no metatable
	function() end, -- nil
	{ [workspace.Baseplate] = true } -- the key arrives as a string
)
```

These are the argument limits in the docs' "Remote events and callbacks" guide. The same rules apply in both directions, and to RemoteFunctions.

Every table arrives as a copy. Change it on the server and the client's table does not change. Fire the same table twice and the server gets two different tables.

An Instance is different: it arrives as the receiver's own copy of that object, but only if the receiver can see it. A part in ServerStorage reaches a client as `nil`, just as the ghost reached the server as `nil`.

> **Pure arrays or pure dictionaries**
>
> The docs warn against tables that mix numeric and string keys, and against `nil` gaps at any index. They do not promise what arrives from either, so never send one. Send `{ 1, 2, 3 }` or `{ x = 1, y = 2 }`, never both in one table.

## Send data, not objects

**Reshape before you send**

```luau
-- The keys are Instances: each arrives as a string, not as the tool
equip:FireServer({ [sword] = 1, [shield] = 2 })

-- An array in slot order: the values are Instances the server can see
equip:FireServer({ sword, shield })
```

An Instance the receiver can see survives as a *value*, but never as a *key*. Here `sword` and `shield` live in ReplicatedStorage.

The fix is always the same shape. Send the plain values the other side needs, in a pure array or a pure dictionary, and let it rebuild the object. Numbers, strings, booleans, `Vector3`, `CFrame`, `Color3`, EnumItems and Instances both sides can see all make the trip.

**Key points**

- Arguments cross as copies: a received table is never the one that was sent.
- Metatables are stripped, and functions arrive as `nil`.
- An Instance arrives only if the receiver can see it; otherwise it is `nil`. Non-string keys become strings.
- Send pure arrays or pure dictionaries of plain values, and rebuild objects on the other side.
