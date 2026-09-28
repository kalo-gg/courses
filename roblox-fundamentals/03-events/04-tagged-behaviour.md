# One script, every tagged part

*Drive every part with a tag from one script: the parts tagged now, the parts tagged later, and none after the tag goes.*

Roblox Fundamentals · Events · 12 min

---

An obby has forty blinking platforms. Put a copy of a Script in each and one fix means forty edits, and a platform created at runtime gets no Script at all. The docs' Script locations guide says the opposite: don't attach scripts to parts in Workspace. **Tag** the parts instead, and handle every one of them from one script through `CollectionService`.

## Now, later, and never again

```luau
local CollectionService = game:GetService("CollectionService")

local function setUp(part: BasePart)
	print("now handling", part:GetFullName())
end

local function cleanUp(part: BasePart)
	print("no longer handling", part:GetFullName())
end

for _, part in CollectionService:GetTagged("Blink") do
	setUp(part)
end
CollectionService:GetInstanceAddedSignal("Blink"):Connect(setUp)
CollectionService:GetInstanceRemovedSignal("Blink"):Connect(cleanUp)
```

`GetTagged` returns what is tagged now, and the added signal never fires for those parts, so the loop handles them. The signal covers everything tagged later. This is the pattern in the docs' Script locations guide.

The removed signal fires in two cases: the tag is removed, or a tagged part leaves the game, for example because it was destroyed. Either way, whatever `setUp` started has to stop. The docs are direct about it: connections and table entries made for a tag should be cleaned up when it goes, or they leak. A loop is no different.

## A loop per part

Flashing is a loop: switch, wait, switch. `task.spawn` runs a function on a new thread straight away and returns that thread, so each part gets a loop that runs alongside everything else. Keep each thread in a table keyed by the part, and `task.cancel` it when the tag goes. Read attributes inside the loop, so an edit takes effect on the next pass.

```luau
local loops: { [BasePart]: thread } = {}

local function setUp(part: BasePart)
	loops[part] = task.spawn(function()
		local lit = false
		while true do
			lit = not lit
			part.Material = if lit then Enum.Material.Neon else Enum.Material.Plastic
			task.wait(part:GetAttribute("FlashTime") or 0.5)
		end
	end)
end

local function cleanUp(part: BasePart)
	local thread = loops[part]
	if thread then
		task.cancel(thread)
		loops[part] = nil
	end
end
```

An `Alarm` light's loop. Clearing `loops[part]` matters as much as the cancel: otherwise the table keeps every removed part alive.

> **Recognising older code**
>
> Free models often ship a Script inside every part, starting `local part = script.Parent`. It still runs, but the Script locations guide recommends tagging the parts and handling them from one place, as this lesson does.

**Key points**

- Tag parts and drive them from one script; the docs advise against a Script inside every part.
- Loop over `GetTagged` for the parts tagged now, and connect `GetInstanceAddedSignal` for the ones tagged later.
- `GetInstanceRemovedSignal` fires when the tag is removed or the part leaves the game: undo `setUp` there.
- Run each part's loop with `task.spawn`, keep the thread in a table, and `task.cancel` it and clear the entry on removal.
