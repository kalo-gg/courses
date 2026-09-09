# Why LuauX?

*The three things markup buys you, the two things it refuses to do, and your first conversion.*

LuauX · LuauX Foundations · 9 min

---

A Roblox interface written with `create "Frame" { … }` is a tree you have to squint at: nesting is braces, siblings are commas, and a misspelled property is discovered when the game runs — usually by a player, in a screen nobody opened during testing.

The usual fix is a runtime layer — a DSL module every screen requires — which adds a dependency, a stack frame and a set of line numbers that are not yours. LuauX takes the other route: the tree is written as a tree and compiled away before Rojo ever sees it.

|  | Hand-written `create` | LuauX |
| --- | --- | --- |
| Nesting | Braces and commas | Tags and indentation |
| Class and property names | Checked when the game runs | Checked at compile time, with a suggestion |
| Runtime cost | None | None — two helpers inlined |
| Debugging | Your lines | Still your lines: N ↔ N |
| What updates | Whatever you passed a function to | The same, plus interpolated text |

The right-hand column is what the compiler adds. Notice how little of it is about the game.

## What it is not

LuauX is not a UI library: it has no state, no components of its own and no idea what a source is. It is not a template engine either — markup is an **expression** inside ordinary Luau, so `local panel = (<Frame />)` is fine, a `.luaux` file can hold plain functions above and below it, and a component is just a function that returns one. And it is not a Studio plugin: the compiler runs on your machine, in a terminal, next to Rojo.

**A .luaux file is a Luau file**

```luaux
local vide = require(game:GetService("ReplicatedStorage").Packages.vide)
local create = vide.create

local function Badge(props: { Text: string })
	return (<TextLabel Text={props.Text} TextSize={14} />)
end

-- ordinary Luau, before and after the markup
local badge = Badge({ Text = "New" })
print(badge.ClassName) -- TextLabel, because under Vide an element is an Instance

return Badge
```

Everything outside the angle brackets is untouched by the compiler. The `create` at the top is the constructor the markup will call — the lesson "Your first file" explains why it has to be there.

**The same file, compiled**

```luau
local vide = require(game:GetService("ReplicatedStorage").Packages.vide)
local create = vide.create

local function Badge(props: { Text: string })
	return (create("TextLabel")({ Text = props.Text, TextSize = 14 }))
end

-- ordinary Luau, before and after the markup
local badge = Badge({ Text = "New" })
print(badge.ClassName) -- TextLabel, because under Vide an element is an Instance

return Badge
```

Twelve lines in, twelve lines out. Only the tag changed: everything above and below it is the file you typed, and there is no `require` of LuauX to find, because there is nothing to require.

> **How mature is it?**
>
> The compiler is covered by 274 tests across Linux, macOS and Windows, plus golden suites that **run** the generated code against real Vide and real React under Lune, and a corpus sweep over the Luau, Vide and react-lua sources confirming ordinary `.luau` is never mistaken for markup. Expect the syntax to be stable and the tooling to keep moving.

**Key points**

- Markup buys structure, compile-time name checking and debuggable output; it costs nothing at runtime.
- LuauX is not a UI library, not a template engine and not a plugin.
- Markup is an expression: it can sit anywhere a Luau expression can.
- Whatever the tree, the constructor call is the library's.
