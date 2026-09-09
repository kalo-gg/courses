# Your first file

*Write a .luaux file, read the .luau it becomes, and mount it from a LocalScript next to it.*

LuauX · LuauX Foundations · 10 min

---

A `.luaux` file is a Luau file with markup expressions in it, so it starts the way a Luau module starts: with its requires. One of them matters to the compiler.

Your `luaux.toml` says `create = "create"`, and LuauX never emits a `require` of its own, so **each file that uses markup has to bring `create` into scope itself** — `local create = vide.create` at the top. The compiler checks that the name is there rather than letting Studio discover it is not.

**src/client/Hello.luaux**

```luaux
local vide = require(game:GetService("ReplicatedStorage").Packages.vide)
local create = vide.create

local function Hello()
	return (
		<ScreenGui ResetOnSpawn={false}>
			<TextLabel Size={UDim2.fromOffset(200, 40)} Text="Hello from LuauX" />
		</ScreenGui>
	)
end

return Hello
```

**build/client/Hello.luau — what comes out**

```luau
local vide = require(game:GetService("ReplicatedStorage").Packages.vide)
local create = vide.create

local function Hello()
	return (
		create("ScreenGui")({ ResetOnSpawn = false,
			create("TextLabel")({ Size = UDim2.fromOffset(200, 40), Text = "Hello from LuauX" }),
		})
	)
end

return Hello
```

Twelve lines in, twelve lines out: every construct lands on the line it came from. The spacing inside a line is the compiler's; the shape — `create("Class")({ props, children })` — is Vide's own idiom, which is why the file reads as if you had typed it.

## Mounting it

**src/client/main.client.luau**

```luau
local Players = game:GetService("Players")
local vide = require(game:GetService("ReplicatedStorage").Packages.vide)
local Hello = require(script.Parent.Hello)

vide.mount(Hello, Players.LocalPlayer:WaitForChild("PlayerGui"))
```

This file has no markup, so `luaux build` copies it into `build/client` unchanged, beside the compiled `Hello.luau`. Rojo turns the `.client.luau` suffix into a LocalScript, and `script.Parent.Hello` finds the module next to it.

That is the whole loop. Edit `src/`, let `luaux watch` write `build/`, and let `rojo serve` sync `build/` into Studio. The `.luau` files are an artifact: never edit them, keep them out of version control, and read them when something goes wrong — they are the truth about what runs.

**Key points**

- A `.luaux` file is Luau with markup expressions; its requires come first.
- Whatever `[factory]` names has to be in scope where it is used — LuauX emits no `require`.
- The output keeps your line count; read it when Studio complains.
- A LocalScript beside the compiled module mounts it; Rojo syncs the build directory.
