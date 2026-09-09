# Inner scopes

*The scope every callback is handed, the inner scopes you make with innerScope, and the warning when lifetimes cross.*

Fusion · State Objects · 11 min

---

A menu lives as long as the player is in the shop; the shop lives as long as the session. Real interfaces are nested lifetimes, and Fusion has two ways to express them: the scope it hands every callback, and the one you make yourself with `innerScope`.

**A computed that builds an instance**

```luau
local Fusion = require(game:GetService("ReplicatedStorage").Fusion)
local scope = Fusion.scoped(Fusion)
local peek = Fusion.peek
local Children = Fusion.Children

local name = scope:Value("Ada")

local holder = scope:New "Frame" {
	[Children] = scope:Computed(function(use, scope)
		-- this scope belongs to this run of the computed
		return scope:New "TextLabel" { Text = use(name) }
	end),
}

print(#holder:GetChildren()) --> 1
name:set("Grace")
print(#holder:GetChildren()) --> 1: the old label was destroyed, a new one built
```

The inner `scope` shadows the outer one on purpose. Build with it and the label dies when the computed reruns; build with the outer one and every rerun leaks a label.

## Scopes inside scopes

`innerScope` handles that: it makes a new scope and puts it inside an existing one. Cleaning the inner one destroys everything built with it and leaves the outer scope standing, which is exactly what closing a menu inside a session should do.

In the source's words, it is "destroyed exactly once, whether by the user or by the scope that it's inside of" — close it yourself when the menu closes, or let the session's scope take it when the player leaves.

**A lifetime inside a lifetime**

```luau
local session = Fusion.scoped(Fusion)

local function openMenu()
	local menu = Fusion.innerScope(session)

	menu:New "Frame" { Parent = playerGui }

	return function()
		Fusion.doCleanup(menu) -- closing early is fine
	end
end

-- and when the session ends:
Fusion.doCleanup(session) -- any menu still open goes with it
```

> **deriveScope when you only want the methods**
>
> `deriveScope` gives you a fresh scope with the same constructors but no parent relationship. Reach for `innerScope` when the new scope should die with the old one, and `deriveScope` when it genuinely has its own lifetime.

> **When lifetimes cross: possiblyOutlives**
>
> Fusion checks lifetimes on the things you bind — properties, `Out`, `Observer`, and a spring's goal, speed and damping. Get two scopes the wrong way round and you get `possiblyOutlives`: *"X will be destroyed before Y"*, pointing you at the order things were created in and which scopes they belong to. It is a warning rather than an error, and it is the diagnostic that tells you a scope decision was wrong before the bug does.
>
> Try it: create a `Value` in an inner scope and read it from a `Computed` in the outer one. The sandbox prints `[Fusion] The use()-d Value will be destroyed before the Computed; the latter is in a different scope that gets destroyed too quickly. To fix this, review the order they're created in, and what scopes they belong to. …`, then an indented `ID: possiblyOutlives` line and a stack trace.

**Key points**

- Every callback (`Computed`, `For` processors) receives a scope Fusion cleans up on the next run; build per-run objects with it.
- `innerScope(parent)` is destroyed exactly once: by you, or by the parent, whichever comes first.
- `deriveScope` copies the constructors, not the lifetime.
- `possiblyOutlives` means something will be destroyed before what depends on it; fix the scope, not the symptom.
