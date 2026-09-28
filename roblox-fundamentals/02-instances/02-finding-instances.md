# Finding instances

*Dot indexing, `FindFirstChild` and its class-based relatives, searching every depth, and where `attempt to index nil` comes from.*

Roblox Fundamentals · Instances · 10 min

---

A script needs the sign in the lobby, so it writes `workspace.Lobby.Sign`. That works until a builder renames the sign, and then the script stops with an error on that line. Sometimes stopping is right, because a missing sign is a bug. When the thing may legitimately be missing, ask first with `FindFirstChild`.

## Dot indexing and FindFirstChild

```luau
local lobby = workspace.Lobby

local door = lobby.Door -- no child called Door? This line errors:
--> Door is not a valid member of Folder "Workspace.Lobby"

local sign = lobby:FindFirstChild("Sign") -- nil if there is none
if sign then
	sign.Transparency = 0
end
```

Dot indexing throws when the name is missing. `FindFirstChild` returns `nil` instead, and your `if` decides what a missing sign means.

Dot indexing looks up **properties first**, then children. A part with a child called `Name` or `Color` still gives you the property for `part.Name` and `part.Color`, and only `FindFirstChild` reaches the child. Give children names no property uses, and use `FindFirstChild` for names that come from data, such as the item a player picked.

> **attempt to index nil with …**
>
> `FindFirstChild` returns `nil` when nothing matches, so the error comes one step later: `workspace:FindFirstChild("Sign").Transparency = 0` stops with `attempt to index nil with 'Transparency'`. The line that errors uses the `nil`; the cause is the search that found nothing. Check the result with `if` before you use it.

## Searching by class, down and up

| Call | Finds | Returns |
| --- | --- | --- |
| `FindFirstChild(name)` | The first child with that name | Instance or `nil` |
| `FindFirstChild(name, true)` | The first descendant with that name, at any depth | Instance or `nil` |
| `FindFirstChildOfClass(class)` | The first child whose `ClassName` is exactly `class` | Instance or `nil` |
| `FindFirstChildWhichIsA(class)` | The first child for which `IsA(class)` is true | Instance or `nil` |
| `GetChildren()` | Every child | Array |
| `GetDescendants()` | Every descendant, at any depth | Array |
| `FindFirstAncestorOfClass(class)` | The nearest ancestor whose `ClassName` is `class` | Instance or `nil` |

Of the class searches, only `WhichIsA` also takes the `recursive` argument. The docs promise no order for either array.

Each `FindFirst` method returns one result. When siblings share a name, as the coins in a level do, dot indexing and `FindFirstChild` each return just one of them. To handle all of them, loop over `GetChildren` or `GetDescendants` and test each one by name, by class, or both.

```luau
-- every part in the bridge, however deeply nested
for _, descendant in workspace.Bridge:GetDescendants() do
	if descendant:IsA("BasePart") then
		descendant.Anchored = true
	end
end

-- from a part up to the model it belongs to
local bulb = workspace.Street.Lamp.Bulb
print(bulb:FindFirstAncestorOfClass("Model")) --> Lamp, the nearest Model above
```

Searching down finds what a container holds. Searching up answers *what is this part part of?*, which Module 4 uses to get from a touched limb to its character.

**Key points**

- Dot indexing errors when a child is missing; `FindFirstChild` returns `nil`, which you check with `if`.
- Properties win over children of the same name, so only `FindFirstChild` reaches a child called `Name`.
- `FindFirstChildOfClass` matches `ClassName` exactly; `FindFirstChildWhichIsA` uses `IsA`.
- `GetDescendants` covers every depth. Loop over it to handle every match, not just the first.
- `attempt to index nil with …` usually means an earlier search found nothing.
