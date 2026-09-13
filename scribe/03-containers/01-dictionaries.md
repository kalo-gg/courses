# Dictionaries

*DictOf holds many entries of one shape under one name; writing a key creates it and every entry validates like a field.*

Scribe · Containers & Big Numbers · 10 min

---

Most player data is a collection. `Scribe.DictOf(shape, opts)` is a string-keyed map whose every entry has the same declared shape. Writing a key creates the entry, and the entry is a typed accessor like anything else. `Get()` on an unwritten key is `nil` and `Count()` excludes it; `Remove(key)` deletes the entry and hands it back.

**Emberfall's Inventory**

```luau
local RARITIES = { "Common", "Rare", "Epic", "Legendary" }

Inventory = Scribe.DictOf({
	Qty = Scribe.Int(1, { Min = 1, Max = 999 }),
	Rarity = Scribe.Enum("Common", RARITIES),
}, { MaxKeys = 200 }),

-- later
data.Inventory.Emberblade.Set({ Qty = 1, Rarity = "Legendary" })
data.Inventory.Emberblade.Qty.Increment(1) --> 2

print(data.Inventory.Emberblade.Rarity.Get()) --> Legendary
print(data.Inventory.Count()) --> 1

data.Inventory.Remove("Emberblade")
```

## Element rules

Records are closed: a field the shape does not declare is a write error naming it, so `Colour.Set("red")` fails loudly instead of persisting forever. Omitted fields fill from their defaults: `Torch.Set({ Rarity = "Common" })` stores `Qty = 1` too. Caps reject growth: `MaxKeys`, and `MaxKeyLength` in bytes, turn runaway growth into an error naming the field; a whole-container `Set` that does not grow the count is still allowed over the cap. A `Scribe.Optional` element field has no default and reads `nil` until written.

**Defaults fill in, undeclared fields do not**

```luau
data.Inventory.Torch.Set({ Rarity = "Common" })
print(data.Inventory.Torch.Qty.Get()) --> 1, from the element default

print(pcall(function()
	data.Inventory.Torch.Colour.Set("red")
end)) --> false  Scribe: "Colour" is not a declared field of the element shape at "Inventory.Torch"
```

## Maps with typed keys

`DictOf` is string-keyed. When the keys are user ids, `Scribe.MapOf("integer", shape, opts)` declares the key type: a DataStore serialises every object key to a JSON string, and only the declared type makes turning `"123"` back into `123` unambiguous. Scribe restores the keys on load. Fractional, infinite and NaN keys are refused at the write.

**Friends, keyed by user id**

```luau
Friends = Scribe.MapOf("integer", {
	Name = Scribe.String("", { MaxLength = 32 }),
	LastSeen = Scribe.Int(0, { Min = 0 }),
}),

data.Friends[ben.UserId].Name.Set("Ben") -- no tostring, no tonumber
```

**Key points**

- `Scribe.DictOf(shape, opts)`: writing a key creates a typed entry; reading an absent key is nil.
- Records are closed, omitted fields fill from defaults, and caps refuse growth with an error naming the field.
- `Count()` counts written keys; `Remove(key)` deletes one.
- `Scribe.MapOf("integer", …)` declares the key type so ids survive the DataStore's JSON round trip.
