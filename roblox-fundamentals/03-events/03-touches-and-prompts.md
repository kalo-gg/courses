# Touches and prompts

*Make a pad react once per touch with a cooldown, and give players a prompt they press on purpose.*

Roblox Fundamentals · Events · 11 min

---

A pad gives a coin to whoever steps on it, and the first player to test it walks off with a pile of coins. Nothing is broken. `Touched` fires once for each part of the character that touches the pad, and it keeps firing in quick succession as the contact settles. A handler that should act once per touch needs a **debounce**: a flag that ignores touches until a cooldown ends.

## A cooldown on the pad

```luau
local pad = workspace.CoinPad
local COOLDOWN = 1

pad.Touched:Connect(function(otherPart)
	if pad:GetAttribute("CoolingDown") then
		return
	end
	pad:SetAttribute("CoolingDown", true)
	print(otherPart.Name .. " touched the pad")
	task.wait(COOLDOWN)
	pad:SetAttribute("CoolingDown", false)
end)
```

The first touch sets the flag, and every touch after it returns at once. `task.wait` pauses only this handler's thread; each firing runs on its own thread, so the game carries on. The flag lives on the pad, so each pad has its own cooldown.

Touch events come only from physical movement, and one of the two parts must be unanchored. An anchored pad is fine, because characters are not anchored. `CanTouch` switches touch events off: both parts need it on, and connecting `Touched` on a part with it off errors. `TouchEnded` fires as each part stops touching, which suits a plate that stays down while someone stands on it.

> **Touches in the sandbox**
>
> No physics runs here, so `sim.touch(pad, player)` stands in for a contact: every part of the character touches the pad, over three frames, and a handler with no debounce runs many times. It honours `CanTouch`.

## Prompts: an action the player chooses

A touch happens whether the player meant it or not. A `ProximityPrompt` asks first. Parent one to a part and a key hint appears when a character comes near; `Triggered` fires when the player presses it, and passes that player.

```luau
local chest = workspace.Chest

local prompt = Instance.new("ProximityPrompt")
prompt.ActionText = "Open"
prompt.ObjectText = "Chest"
prompt.HoldDuration = 0.5 -- hold the key for half a second
prompt.Parent = chest

prompt.Triggered:Connect(function(player)
	print(player.Name .. " opened the chest")
	prompt.Enabled = false -- stop showing the prompt
end)
```

`Triggered` hands you the player. A `Touched` handler gets a body part and has to work out the player from it; the Players module shows how.

| The player… | Listen to | The handler receives |
| --- | --- | --- |
| bumps into a part | `BasePart.Touched`, `TouchEnded` | the other part |
| presses a key or button near an object | `ProximityPrompt.Triggered` | the player |
| clicks or taps an object | `ClickDetector.MouseClick` | the player |

> **Recognising older code**
>
> Older debounces end with `wait(1)`, and older loops start with `spawn(fn)`. Those globals are deprecated and throttle. Write the `task` versions, which do not: `task.wait(1)`, `task.spawn(fn)` and `task.delay(1, fn)`.

**Key points**

- `Touched` fires for each touching part and repeats as a contact settles, so a once-per-touch handler needs a debounce.
- Keep the cooldown on the pad (an attribute) and time it with `task.wait`, never the deprecated `wait`.
- `CanTouch` must be on for both parts; touches come only from physical movement.
- `ProximityPrompt.Triggered` and `ClickDetector.MouseClick` pass the player; `Touched` passes a part.
