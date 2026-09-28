# UI in the world

*BillboardGuis float over parts and face the camera, SurfaceGuis lie flat on a face, and TweenService animates both.*

Roblox Intermediate · Interface · 12 min

---

Players want to see who is about to lose a fight: a health bar over each character's head that stays readable as the camera swings round. A ScreenGui cannot do it, because it is pinned to the screen, not to a character.

Roblox has two containers that draw UI in the 3D world. A `BillboardGui` floats at a part and always faces the camera. A `SurfaceGui` lies flat on one face of a part, like a sign.

## BillboardGui

**A name over a character's head**

```luau
local tag = Instance.new("BillboardGui")
tag.Size = UDim2.new(4, 0, 1, 0) -- 4 studs wide, 1 stud tall
tag.StudsOffset = Vector3.new(0, 2, 0) -- 2 studs above the head
tag.MaxDistance = 60 -- not drawn beyond 60 studs from the camera
tag.AlwaysOnTop = false -- walls still hide it (the default)

local label = Instance.new("TextLabel")
label.Size = UDim2.fromScale(1, 1)
label.TextScaled = true -- grows and shrinks with the billboard
label.Text = player.DisplayName
label.Parent = tag

tag.Parent = character:WaitForChild("Head")
```

In a BillboardGui's `Size`, the scale parts are studs and the offsets are pixels, so a scale size shrinks with distance as the world does. It is drawn at its parent part, or at `Adornee` when that is set.

A `SurfaceGui` draws on one face of its parent part, picked with `Face`, or of `Adornee` when that is set. The whole face is its canvas, so the docs recommend scale sizes for its children.

Clicks are the catch. Buttons in a SurfaceGui take input only when the SurfaceGui is inside the player's PlayerGui, its `Adornee` points at the part, and the part's `CanQuery` is on. Buttons in a BillboardGui also need it in PlayerGui.

**main.client.luau: a kiosk screen that takes clicks**

```luau
local Players = game:GetService("Players")

local playerGui = Players.LocalPlayer:WaitForChild("PlayerGui")
local kiosk = workspace:WaitForChild("Kiosk")

local screen = Instance.new("SurfaceGui")
screen.Face = Enum.NormalId.Front
screen.Adornee = kiosk -- drawn on the kiosk, though it lives in PlayerGui
screen.ResetOnSpawn = false

local open = Instance.new("TextButton")
open.Size = UDim2.fromScale(1, 0.25)
open.Text = "Open shop"
open.Parent = screen

screen.Parent = playerGui
```

`CanQuery` is true by default, so the kiosk takes clicks unless someone turned it off. Connect `open.Activated` as on any other button.

## Tweening UI

UI properties such as `Size`, `Position` and `BackgroundTransparency` tween with the TweenService you met in Roblox Fundamentals. Tween a health bar's fill to its new width, and a hit reads as a slide instead of a jump.

```luau
local TweenService = game:GetService("TweenService")
local SLIDE = TweenInfo.new(0.25, Enum.EasingStyle.Quad, Enum.EasingDirection.Out)

local function showHealth(fill: Frame, humanoid: Humanoid)
	local goal = { Size = UDim2.fromScale(humanoid.Health / humanoid.MaxHealth, 1) }
	TweenService:Create(fill, SLIDE, goal):Play()
end
```

A newer tween on the same property cancels the one still running, so a burst of hits never fights itself.

> **Recognising older code**
>
> `fill:TweenSize(UDim2.fromScale(0.5, 1), Enum.EasingDirection.Out, Enum.EasingStyle.Quad, 0.25, true)` and `TweenPosition` are deprecated GuiObject methods. Write a TweenService tween instead; the old callback argument becomes the tween's `Completed` event.

> **Frameworks do this bookkeeping**
>
> Every bar here needed a connection to set up and a respawn to handle, by hand. UI frameworks let you describe the interface from state and keep the Instances in sync for you: Kalo's Fusion, Vide and Fluid courses each teach one. Build a screen or two by hand first, so you know what the framework is doing, then pick one.

**Key points**

- A `BillboardGui` draws at its parent part or `Adornee` and always faces the camera; the scale parts of its `Size` are studs.
- `StudsOffset` shifts a billboard, `AlwaysOnTop` draws it over 3D objects, and `MaxDistance` stops drawing it far from the camera.
- A `SurfaceGui` draws on the `Face` of its part; its buttons need it in PlayerGui, with `Adornee` set and the part's `CanQuery` on.
- Tween UI with TweenService; `TweenSize` and `TweenPosition` are deprecated.
