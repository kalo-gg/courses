# Watching for change

*React when a property, an attribute or a child changes, using the signal that names exactly what you care about.*

Roblox Fundamentals · Events · 10 min

---

A door has an `Open` attribute. A lever flips it, a timer flips it, an admin command flips it. The door should not care who did it: it should swing whenever `Open` changes. Checking it every frame works, and spends every frame on a value that changes twice a minute. Roblox fires a signal for each property, attribute and child change instead. The skill is picking the narrowest one.

## One property, or all of them

```luau
local door = workspace.Door

-- one property: no arguments, so read the value yourself
door:GetPropertyChangedSignal("Transparency"):Connect(function()
	print("Transparency is now", door.Transparency)
end)

-- any property: the handler gets the property's name
door.Changed:Connect(function(property)
	print(property, "changed")
end)
```

The docs prefer `GetPropertyChangedSignal` to a `Changed` handler that filters on the name. Asking twice for the same property returns the same signal.

Two exceptions. A value object such as `IntValue` has its own `Changed`: it fires only when `Value` changes and passes the **new value**, not a name. And physics is silent: when gravity or a collision moves a part, `Position` and `CFrame` change without firing either signal. The Motion module watches those every frame instead.

| To hear | Connect to | The handler receives |
| --- | --- | --- |
| One property | `inst:GetPropertyChangedSignal("Color")` | nothing |
| Any property | `inst.Changed` | the property's name |
| A value object's `Value` | `intValue.Changed` | the new value |
| One attribute | `inst:GetAttributeChangedSignal("Open")` | nothing |
| Any attribute | `inst.AttributeChanged` | the attribute's name |
| A direct child | `inst.ChildAdded`, `inst.ChildRemoved` | the child |
| Anything below | `inst.DescendantAdded` | each new descendant |

## Attributes and children

Attributes get the same pair: `GetAttributeChangedSignal(name)` for one, `AttributeChanged` for any. The tree has `ChildAdded` and `ChildRemoved` for direct children and `DescendantAdded` for any depth. Parenting a model that holds ten parts fires `DescendantAdded` eleven times: once for the model and once for each part.

```luau
local door = workspace.Door

local function showDoor()
	local open = door:GetAttribute("Open")
	door.Transparency = if open then 0.8 else 0
	door.CanCollide = not open
end

door:GetAttributeChangedSignal("Open"):Connect(showDoor)
showDoor() -- the signal reports changes only, so apply the current value once

workspace.Pickups.ChildAdded:Connect(function(child)
	print("new pickup:", child.Name)
end)
```

A change signal never fires for the value the door already has. Call the handler once yourself and the door is right from the start.

**Key points**

- `GetPropertyChangedSignal(name)` fires for one property and passes nothing; `Changed` fires for any and passes the name.
- A value object's `Changed` fires only for `Value` and passes the new value.
- Physics moving a part fires no property signal.
- Attributes have `GetAttributeChangedSignal` and `AttributeChanged`; the tree has `ChildAdded`, `ChildRemoved` and `DescendantAdded`.
- A change signal reports changes only, so apply the current value once yourself.
