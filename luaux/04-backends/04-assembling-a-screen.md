# Assembling a screen

*One component per file, a style module, an App that assembles them and a LocalScript that mounts: the real layout.*

LuauX · Backends · 10 min

---

A screen is several files, and the layout that stays readable is the obvious one: one component per `.luaux` file, a plain `.luau` module of shared prop tables, an `App.luaux` that assembles the pieces, and a LocalScript that mounts `App` and does nothing else.

Each `.luaux` file opens with its own requires — `vide`, the `create` that `[factory]` names, and the components it uses — because the compiler never adds one for you. Files without markup are not compiled; `luaux build` **copies** them into the output, so `require(script.Parent.Styles)` resolves beside the compiled modules exactly as it did beside the sources.

**src and what it becomes**

```text
src/client/                 build/client/            (Rojo syncs this one)
  Styles.luau         ->      Styles.luau            copied
  Toggle.luaux        ->      Toggle.luau            compiled
  Row.luaux           ->      Row.luau               compiled
  SettingsPanel.luaux ->      SettingsPanel.luau     compiled
  App.luaux           ->      App.luau               compiled
  main.client.luau    ->      main.client.luau       copied: mounts App
```

**src/client/App.luaux**

```luaux
local vide = require(game:GetService("ReplicatedStorage").Packages.vide)
local create = vide.create
local source = vide.source
local Styles = require(script.Parent.Styles)
local Toggle = require(script.Parent.Toggle)
local SettingsPanel = require(script.Parent.SettingsPanel)

local music = source(true)
local version = "1.4.0"

local function App()
	return (
		<ScreenGui ResetOnSpawn={false}>
			<SettingsPanel Title="Settings">
				<Toggle Label="Music" Value={music} />
			</SettingsPanel>
			<TextLabel {Styles.Heading} TextSize={12}>v{version}</TextLabel>
		</ScreenGui>
	)
end

return App
```

State is created here and handed down as sources; components never create the state they display. The footer spreads a style table and overrides one key after it.

## The Rojo side

**default.project.json**

```json
{
  "name": "settings",
  "tree": {
    "$className": "DataModel",
    "ReplicatedStorage": {
      "Packages": { "$path": "Packages" }
    },
    "StarterPlayer": {
      "StarterPlayerScripts": { "$path": "build/client" }
    }
  }
}
```

`build/client`, never `src/client`. `main.client.luau` becomes a LocalScript, and the modules beside it are what it requires.

Build the screen bottom-up and it never fails on something you have not written yet: the style module first, then leaf components that take only values and sources, then the containers that forward children, then `App`, then the entry that mounts it. Keep state in `App` (or a store module it requires) and pass sources down; a component that creates its own source is one you cannot reset from outside.

**Key points**

- One component per file; shared prop tables in a plain `.luau` module the build copies.
- Every `.luaux` file requires what it uses — the compiler adds nothing.
- Rojo's tree points at `build/client`; the LocalScript beside the modules mounts `App`.
- Build bottom-up; keep state in `App` and pass sources down.
