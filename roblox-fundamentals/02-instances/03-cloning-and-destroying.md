# Cloning and destroying

*Copy templates kept in ServerStorage with `Clone`, and remove instances for good with `Destroy`.*

Roblox Fundamentals · Instances · 10 min

---

A level needs fifty coins with the same size, colour and attributes. Building each with `Instance.new` repeats a dozen lines fifty times, and changing the coin later means finding all fifty. Build the coin once, keep it as a **template**, and copy it.

## Templates and Clone

```luau
local ServerStorage = game:GetService("ServerStorage")
local template = ServerStorage.Templates.Coin

local coin = template:Clone()
coin.Position = Vector3.new(0, 5, 0)
coin.Parent = workspace
```

`Clone` copies the instance and all its descendants. The copy starts with no parent, so set it up and parent it last, as with `Instance.new`.

Keep templates in **ServerStorage**. Its contents are not part of the world and are never sent to players, and scripts inside it do not run, so a template waits there untouched.

A clone keeps the template's attributes as well as its properties. Tags are stored with the instance too, though no doc page promises that a clone keeps them. `Clone` skips any descendant whose `Archivable` is `false`, and cloning an instance that is itself not `Archivable` returns `nil`.

## Destroy, or Parent = nil

A collected coin should be gone for good. The docs name `Destroy` as the way to dispose of an instance: it sets `Parent` to `nil`, **locks** `Parent`, disconnects the instance's connections and destroys every descendant. Setting `Parent = nil` only takes the instance out of the place, ready to be put back.

```luau
coin:Destroy()
print(coin.Parent) --> nil

coin.Parent = workspace
--> The Parent property of Coin is locked, current parent: NULL, new parent Workspace
```

Your variable still refers to the destroyed coin, and you can read its properties. What you cannot do is put it back.

> **The Parent property of … is locked**
>
> This error means code is reusing an instance that was destroyed: often a template destroyed by mistake, or a coin a second handler already collected. Clone a fresh copy instead, or set `Parent = nil` where you meant to hide the object and bring it back.

> **Recognising older code**
>
> `coin:Remove()` and `coin:remove()` are deprecated: write `coin:Destroy()`, or `coin.Parent = nil` to keep the object for later. Old code also writes `template:clone()`, the deprecated camelCase spelling of `Clone`.

**Key points**

- `Clone` copies an instance, its descendants and their attributes; the copy has no parent, so parent it last.
- Keep templates in ServerStorage and clone them. Moving the template itself leaves nothing to clone next time.
- `Archivable = false` keeps an instance out of `Clone`.
- `Destroy` removes an instance for good: it locks `Parent`, disconnects connections and destroys descendants.
- Set `Parent = nil` instead when the instance is coming back.
