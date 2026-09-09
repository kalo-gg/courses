# Delayed destruction

*Return a delay from a dynamic component to play an exit animation before it is destroyed.*

Vide · Dynamic Scopes · 9 min

---

Destroying a menu the instant its condition turns false leaves no room for a fade-out. Every dynamic scope function lets the component return a second value: a number of seconds to wait before the scope is destroyed. During that wait the `present` source the component received is `false`, which is the cue to start the exit animation. The examples use `spring`, covered in the next module: it is a source that moves towards its target over time, and that is all you need to know here.

```luau
local function Menu(props: { Visible: () -> boolean })
	local transparency = spring(function()
		return if props.Visible() then 0 else 1
	end)

	return create "Frame" {
		BackgroundTransparency = transparency,
	}
end

local toggled = source(true)

create "ScreenGui" {
	show(toggled, function(_, present)
		return Menu { Visible = present }, 3 -- keep the scope 3 s after present goes false
	end),
}

toggled(false)
-- present() becomes false immediately, the spring starts fading,
-- and the scope is destroyed 3 seconds later
```

## Reviving during the delay

If the condition turns truthy again within the delay, the pending destruction is canceled, `present` flips back to `true` and the same scope carries on with the state it had. That is what makes an interrupted fade-out look right: the spring simply turns around.

## Every dynamic scope

The same second return value works for `switch`, `indexes` and `values`. The timer counts Vide's stepping clock, which is driven by `RunService.Heartbeat` in Roblox and by `step()` in the Kalo sandbox.

**Rows that linger**

```luau
-- inside a root or component
local feed = source({ "Picked up Sword" })

create "Frame" {
	create "UIListLayout" {},
	indexes(feed, function(text, i)
		return create "TextLabel" { Text = text, LayoutOrder = i }, 1 -- rows stay for a second
	end),
}

feed({}) -- present() is false in every row; the labels leave one second later
```

**Key points**

- Return `instance, seconds` from a dynamic component to delay its destruction.
- `present` flips to false at the start of the delay: use it to drive exit animations.
- A condition that turns truthy again during the delay cancels the destruction.
