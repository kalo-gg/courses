# Mounting an app

*Where the code lives, how it starts, and how a whole UI tree hangs off one root.*

Vide · Shipping a Vide App · 9 min

---

UI runs on the client, so a Vide app starts from a **LocalScript**, typically in `StarterPlayerScripts`. That script requires Vide, requires your top-level component, and calls `mount` with the local player's `PlayerGui` as the target. Everything else is components in ModuleScripts under `ReplicatedStorage`.

**StarterPlayerScripts/App.client.luau**

```luau
local Players = game:GetService("Players")
local ReplicatedStorage = game:GetService("ReplicatedStorage")

local vide = require(ReplicatedStorage.Packages.vide)
local App = require(ReplicatedStorage.UI.App)

vide.strict = true -- while developing (see the next lesson)

local unmount = vide.mount(App, Players.LocalPlayer.PlayerGui)
```

`mount` returns the destructor. Calling it destroys every scope in the app, so bindings stop updating. The instances themselves stay where they are: queue them with `cleanup(gui)` inside the app, or destroy them yourself, if the screen should disappear.

## Where the files live

With Rojo, the split is three folders: the packages Wally installed and your components as ModuleScripts under ReplicatedStorage, and the one LocalScript that starts the app under StarterPlayerScripts. The project file below is all it takes, and it is the shape to reach for when your components require each other through `script.Parent` or an absolute path from `ReplicatedStorage`.

**default.project.json**

```json
{
  "name": "my-game",
  "tree": {
    "$className": "DataModel",
    "ReplicatedStorage": {
      "Packages": { "$path": "Packages" },
      "UI": { "$path": "src/ui" }
    },
    "StarterPlayer": {
      "$className": "StarterPlayer",
      "StarterPlayerScripts": {
        "$className": "StarterPlayerScripts",
        "App": { "$path": "src/App.client.luau" }
      }
    }
  }
}
```

This course's own two projects use a different shape: everything — `main.client.luau`, `state.luau`, `components` and `Packages` — sits under one `App` folder in StarterPlayerScripts, mirroring the file layout, so that their relative requires (`./state`, `../Packages/vide`) resolve in Studio too.

**ReplicatedStorage/UI/App.luau**

```luau
local vide = require(script.Parent.Parent.Packages.vide)
local create = vide.create
local Hud = require(script.Parent.Hud)

local function App()
	return create "ScreenGui" {
		Name = "App",
		ResetOnSpawn = false,

		Hud {},
	}
end

return App
```

## Independent lifetimes

Because `mount` creates the top-level stable scope, every effect in every component below it has a home. You only need extra `root` calls when something must outlive or be destroyed independently of the app, such as a notification spawned from game code.

A toast spawned from game code, a debug overlay, a confirmation that must survive a screen change: give each its own `mount` (or `root`), keep the destructor, and `cleanup(gui)` inside it so closing it removes the instances too.

**Key points**

- Start the app from a LocalScript with `mount(App, Players.LocalPlayer.PlayerGui)`.
- Keep components in ModuleScripts; the entry script only wires them up.
- One mount gives the whole tree a scope; extra roots are for independent lifetimes.
- Unmounting stops every binding but leaves instances in place; `cleanup(gui)` removes them too.
