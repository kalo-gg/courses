# Validating arguments

*Check each argument's type, then its value, then which instance it is and whose, before the handler uses it.*

Roblox Intermediate · Trust · 12 min

---

`Paint:FireServer(tile, 0.5)` asks the server to shade one of the player's tiles grey. The handler checks `shade < 0 or shade > 1` and trusts the rest. Then three requests arrive: a table dressed up as a Part, which makes the handler error; another player's tile; and a shade of `0/0`, which passes the range check.

Each needs its own kind of check. This lesson builds them in order: type, value, then instance.

## Types: typeof, not type

```luau
print(type(Vector3.one))              --> userdata
print(typeof(Vector3.one))            --> Vector3
print(typeof(workspace.Baseplate))    --> Instance
print(typeof({ ClassName = "Part" })) --> table
print(typeof(0 / 0))                  --> number
```

`type` knows only Luau's own types, so every Roblox value is `userdata`. `typeof` names them. A table can carry a `ClassName` field, the trick the docs' shop example shows, but `typeof` still says `table`.

## Numbers and strings

`typeof(0/0)` is `"number"`, and NaN fails every comparison, so `shade < 0 or shade > 1` is false and waves it through. `math.huge` passes a type check too, and both spread into every sum they touch. The docs' client-server guide rejects them with `math.isfinite`, which is false for NaN and ±infinity.

For strings, cap the length. `#` counts bytes, so count characters with `utf8.len`. It returns `nil` for text that is not valid UTF-8, which a DataStore would refuse to save.

```luau
local function validShade(shade: unknown): boolean
	return typeof(shade) == "number" and math.isfinite(shade) and shade >= 0 and shade <= 1
end

local function validLabel(label: unknown): boolean
	if typeof(label) ~= "string" then
		return false
	end
	local length = utf8.len(label) -- nil for invalid UTF-8
	return length ~= nil and length <= 20
end
```

Type first, then value: comparing the string `"1"` with `0` errors, and `nil <= 20` errors too. `#"é"` is 2, but `utf8.len("é")` is 1.

## Instances: which one, and whose

An Instance argument can be any instance the client can see. Check that it is an Instance at all (`typeof`), that it is the right class (`IsA`), and that it sits where it should and belongs to this player (`IsDescendantOf`). Put `typeof` first: calling `IsA` on a table errors.

**The docs' shop, checking which item it was sent**

```luau
BuyItem.OnServerEvent:Connect(function(player, item: unknown)
	if typeof(item) ~= "Instance" or not item:IsA("Folder") then
		return
	end
	if not item:IsDescendantOf(ReplicatedStorage.ItemData) then
		return
	end
	-- item is one of ours: only now read its price
end)
```

A failed check returns quietly. An error would also stop the request, but it fills the log, and one thrown halfway through a handler can leave the work half done.

**Key points**

- `typeof` names Roblox types, where `type` says `userdata`. A table can pose as an Instance, but not to `typeof`.
- Type annotations check nothing at runtime; every argument is whatever the client sent.
- `typeof(0/0)` is `"number"` and NaN fails every comparison: reject it and ±infinity with `math.isfinite`.
- Cap strings with `utf8.len`, which counts characters and returns `nil` for invalid UTF-8.
- For an Instance: `typeof`, then `IsA`, then `IsDescendantOf` the place it must be, including the player's own part of the world.
