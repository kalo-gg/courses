# ScreenGui and PlayerGui

*On-screen UI lives in each player's PlayerGui: StarterGui fills it on spawn, and ResetOnSpawn decides what a respawn wipes.*

Roblox Intermediate · Interface · 10 min

---

A client script builds a coin counter at the top of the screen. It works until the player falls off the map. After the respawn the counter is gone, and the Output shows no error.

Nothing was broken: the respawn removed it. This lesson covers where on-screen UI lives, how it gets there, and which of it a respawn resets.

## From StarterGui to PlayerGui

A `ScreenGui` draws its children on the screen only while it sits in a player's `PlayerGui`, a container inside their Player. The docs put it plainly: on-screen UI objects and code are stored and changed on the client.

There are two ways to get one there. Put it in `StarterGui` in Studio, and the engine copies it into each player's PlayerGui when their character first spawns. Or build it in client code and parent it to the PlayerGui yourself.

**main.client.luau: a HUD built in code**

```luau
local Players = game:GetService("Players")

local playerGui = Players.LocalPlayer:WaitForChild("PlayerGui")

local hud = Instance.new("ScreenGui")
hud.Name = "Hud"
hud.ResetOnSpawn = false

local coins = Instance.new("TextLabel")
coins.Name = "Coins"
coins.Text = "Coins: 0"
coins.Parent = hud

hud.Parent = playerGui
```

`ResetOnSpawn` defaults to `true`, which makes a ScreenGui delete itself when the character respawns. A copy from StarterGui is then copied in again. One you built in code has nothing to replace it, so turn the property off.

> **Where the code that builds UI runs**
>
> Many places put a LocalScript inside each ScreenGui in StarterGui. It is copied with the ScreenGui, so on every reset it starts again with a fresh copy. The client entry point this course uses, a `Client` Script in ReplicatedStorage, runs once, which is why UI it builds needs `ResetOnSpawn = false`.

## Layers and safe areas

Most games have several ScreenGuis: a HUD, a shop, a settings menu. Keeping them separate lets you show or hide each one with its `Enabled` property, and two more properties decide how they share the screen.

| ScreenGui property | Default | What it decides |
| --- | --- | --- |
| `Enabled` | `true` | When false, nothing in it renders, takes input or updates. |
| `DisplayOrder` | `0` | Which ScreenGui draws on top: a higher value draws over a lower one. |
| `ScreenInsets` | `CoreUISafeInsets` | The area its contents fit in. The default keeps them clear of the Roblox top bar and of cut-outs such as a phone's camera notch. |
| `ResetOnSpawn` | `true` | Whether it deletes itself when the character respawns. |

**A shop over the HUD, and a full-screen backdrop**

```luau
local shop = Instance.new("ScreenGui")
shop.Name = "Shop"
shop.DisplayOrder = 10 -- draws over the HUD, which is at 0
shop.Enabled = false -- hidden until the player opens it
shop.ResetOnSpawn = false
shop.Parent = playerGui

local backdrop = Instance.new("ScreenGui")
backdrop.Name = "Backdrop"
backdrop.ScreenInsets = Enum.ScreenInsets.None -- the whole screen, under the top bar too
backdrop.ResetOnSpawn = false
backdrop.Parent = playerGui
```

The docs keep `ScreenInsets.None` for non-interactive content such as background images, because anything under the top bar or a notch can end up hidden. Buttons and status text belong in the default safe area.

**Key points**

- A ScreenGui draws only inside a player's PlayerGui, and on-screen UI is built and changed on the client.
- StarterGui's ScreenGuis are copied into each PlayerGui when the character first spawns.
- `ResetOnSpawn` (default true) deletes a ScreenGui on every respawn: StarterGui copies come back fresh, code-built ones do not come back.
- `DisplayOrder` layers ScreenGuis, and the default `ScreenInsets` keeps content clear of the top bar and notches.
