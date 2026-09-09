# Hydrate

*Bind Fusion state onto instances somebody already built in Studio, and know what cleanup does to them.*

Fusion · Instances · 10 min

---

Not every interface starts empty. If a designer laid the screen out in Studio, you do not want to rebuild it in code — you want to attach behavior to what is there.

`Hydrate` "processes and returns an existing instance, with options for setting properties, event handlers and other attributes". Everything the property table does not mention is left exactly as the designer set it.

**Adopting an instance**

```luau
local Fusion = require(game:GetService("ReplicatedStorage").Fusion)
local scope = Fusion.scoped(Fusion)
local peek = Fusion.peek
local Children, OnEvent, OnChange, Out = Fusion.Children, Fusion.OnEvent, Fusion.OnChange, Fusion.Out
local Attribute, AttributeOut, AttributeChange = Fusion.Attribute, Fusion.AttributeOut, Fusion.AttributeChange

-- pretend this came from Studio
local existing = Instance.new("TextLabel")
existing.Name = "Score"

local score = scope:Value(0)

scope:Hydrate(existing) {
	Text = scope:Computed(function(use)
		return "Score: " .. use(score)
	end),
}

score:set(7)
print(existing.Text) --> Score: 7
```

Same property table as `New`, same special keys — the only difference is where the instance came from.

> **Hydrate takes ownership, exactly like New**
>
> This is the trap. `Hydrate` puts the target into the scope with the same `table.insert` that `New` uses, so **`doCleanup` destroys the instance you hydrated** — including one a designer built in Studio. If the target has to survive, hydrate a `:Clone()`, which is what Fusion's own documentation does, or hydrate onto a scope you never clean up.

## Hydrating a clone

The trap has one fix, and it is one word. `:Clone()` hands Fusion a copy to own, and the copy is what the scope destroys; the instance the designer placed is never given to Fusion at all. Hydrate the original only when the scope genuinely outlives the screen — one you never clean up.

**Hydrate the copy, keep the original**

```luau
local Fusion = require(game:GetService("ReplicatedStorage").Fusion)
local scope = Fusion.scoped(Fusion)
local peek = Fusion.peek
local Children, OnEvent, OnChange, Out = Fusion.Children, Fusion.OnEvent, Fusion.OnChange, Fusion.Out
local Attribute, AttributeOut, AttributeChange = Fusion.Attribute, Fusion.AttributeOut, Fusion.AttributeChange

local hud = Instance.new("Folder")

-- the designer's original, which has to survive the screen closing
local template = Instance.new("TextLabel")
template.Name = "ScoreLabel"
template.Parent = hud

local score = scope:Value(7)

scope:Hydrate(template:Clone()) {
	Parent = hud,
	Text = scope:Computed(function(use)
		return "Score: " .. use(score)
	end),
}

print(#hud:GetChildren()) --> 2: the original and the live copy
Fusion.doCleanup(scope)
print(#hud:GetChildren()) --> 1: the clone is gone, the original is not
```

The scope owns the copy and nothing else. Cleanup cannot reach the template, because the template was never handed to Fusion.

**Key points**

- `scope:Hydrate(instance) { … }` takes the same property table and special keys as `New`.
- It adopts the instance: `doCleanup` destroys it.
- Hydrate a clone when the original must survive.
- Use it to add behavior to designer-built UI instead of rebuilding it in code.
