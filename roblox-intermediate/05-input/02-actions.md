# Actions on every device

*An InputContext holds InputActions, and each action has one InputBinding per device, so one handler serves keyboard, gamepad and touch.*

Roblox Intermediate · Input · 11 min

---

An Interact prompt should answer E on a keyboard, X on a controller and a button on a phone. With UserInputService that is two `KeyCode` tests, an `Activated` handler for the button, and nothing to stop a menu's E from also opening a door.

The **Input Action System** turns this around: name the action once, bind each device's input to it, and listen to the action. It reached full release in June 2026, and Roblox's input guides now teach it.

## Context, action, binding

Three classes nest inside each other:

- an `InputContext`, a group of actions you switch on and off together, like gameplay or a menu;
- an `InputAction`, one thing the player does, such as Sprint or Interact;
- an `InputBinding` per device, naming a `KeyCode`, or an on-screen button in `UIButton` for touch.

**Client script: the guide's sprint, built in code**

```luau
local Players = game:GetService("Players")
local ReplicatedStorage = game:GetService("ReplicatedStorage")

local playerGui = Players.LocalPlayer:WaitForChild("PlayerGui")
local sprintButton = playerGui:WaitForChild("Controls"):WaitForChild("SprintButton")

local inputs = Instance.new("Folder") -- the guide keeps its contexts in ReplicatedStorage.Inputs
inputs.Name = "Inputs"
inputs.Parent = ReplicatedStorage

local context = Instance.new("InputContext")
context.Name = "PlayContext"
context.Parent = inputs

local sprint = Instance.new("InputAction") -- Type is Bool by default
sprint.Name = "Sprint"
sprint.Parent = context

local keyboard = Instance.new("InputBinding")
keyboard.KeyCode = Enum.KeyCode.LeftShift
keyboard.Parent = sprint

local gamepad = Instance.new("InputBinding")
gamepad.KeyCode = Enum.KeyCode.ButtonY
gamepad.Parent = sprint

local touch = Instance.new("InputBinding")
touch.UIButton = sprintButton
touch.Parent = sprint
```

The guide builds the same tree in Studio's Explorer: an `Inputs` folder in ReplicatedStorage, a context in it, an action inside the context, and a binding per device inside the action. Built by a client script, as here, it exists only on that client.

```luau
-- humanoid: the local character's Humanoid
sprint.Pressed:Connect(function()
	humanoid.WalkSpeed = 32
end)

sprint.Released:Connect(function()
	humanoid.WalkSpeed = 16
end)

print(sprint:GetState()) --> false until an input presses it
```

`Pressed` and `Released` fire on a `Bool` action when its state flips, whichever binding flipped it. `StateChanged(value)` fires for every action type, and `GetState()` reads the state at any time.

Contexts decide who hears an input. Each has `Enabled`, `Priority` (higher runs first) and `Sink`. When a context with `Sink` on binds a key, contexts of lower priority never see that key; contexts of the **same** priority still do.

A menu takes E from the game that way: a higher priority, `Sink` on, and enabled only while the menu is open. An action outside any context joins a default one, and a context nested inside another has no effect.

|  | ContextActionService (older) | Input Action System |
| --- | --- | --- |
| Define an action | `BindAction("Sprint", handler, true, Enum.KeyCode.LeftShift, Enum.KeyCode.ButtonY)` | An `InputAction` with an `InputBinding` per device |
| Touch button | Drawn for you when the third argument is `true` | Your own GuiButton, named in `UIButton` |
| Handler | One callback gets `(actionName, inputState, inputObject)` for every phase | `Pressed`, `Released` and `StateChanged` |
| Who gets a shared key | The most recently bound action, unless `BindActionAtPriority`; return `Pass` to share it | Context `Priority` and `Sink` |
| Where it lives | Only in code | Instances, built in Studio or in code |

Recognising older code: `ContextActionService` is not deprecated, but Roblox's input guides now teach the Input Action System, so this course uses it. The left column is there to help you read older scripts.

> **Roblox's own controls**
>
> Setting `Workspace.PlayerScriptsUseInputActionSystem` to `Enabled` in Studio moves Roblox's default player scripts onto the Input Action System too; scripts cannot set it. It is opt-in for now, and the guide recommends turning it on. The guide gives its own context a `Priority` of 2000 with `Sink` on, high enough to take its keys before those default contexts see them.

**Key points**

- An `InputContext` holds `InputAction`s, and each action holds one `InputBinding` per device.
- A binding names a `KeyCode`, or a GuiButton in `UIButton` for touch.
- `Pressed` and `Released` fire on a `Bool` action, `StateChanged` on every type, and `GetState()` reads the state.
- A higher `Priority` runs first, and `Sink` keeps the keys a context binds from lower-priority contexts.
- `ContextActionService` is the older shape; the input guides now teach the Input Action System.
