# UserInputService

*Every key, click, touch and gamepad button reaches the client as an InputObject, with a flag that says whether the game already used it.*

Roblox Intermediate · Input · 10 min

---

A sprint script speeds the character up while Shift is held. It works in testing, until a player types a capital letter into a text box and their character sets off running. Players on phones have no Shift key at all.

Input happens on the player's device, so input code runs on the client. This lesson covers **UserInputService**: its two events, what an `InputObject` tells you, and how to tell one device from another.

## InputBegan and InputEnded

**Client script**

```luau
local UserInputService = game:GetService("UserInputService")

UserInputService.InputBegan:Connect(function(input: InputObject, gameProcessedEvent: boolean)
	if gameProcessedEvent then
		return -- the game already used this input
	end
	if input.KeyCode == Enum.KeyCode.F then
		print("F went down")
	end
end)

UserInputService.InputEnded:Connect(function(input: InputObject)
	if input.KeyCode == Enum.KeyCode.F then
		print("F came up")
	end
end)
```

`gameProcessedEvent` is true when the engine already acted on the input, generally in UI: a button was clicked or touched with it. Return early on it in `InputBegan`, or a key meant for the interface also starts your action. Stopping is always safe, so `InputEnded` can skip the check.

An `InputObject` describes one input. `UserInputType` is its kind: `Keyboard`, `MouseButton1`, `Touch`, `Gamepad1` and so on. `KeyCode` names the key or gamepad button. A mouse click has no key, so its `KeyCode` carries nothing useful: test clicks with `UserInputType`.

Touch and gamepad input arrive through the same two events. A tap is a `Touch` input whose `Position` says where on the screen it landed, and a controller's A button is `KeyCode.ButtonA`.

## Polling, and which device

```luau
local UserInputService = game:GetService("UserInputService")

-- Is a key held right now?
if UserInputService:IsKeyDown(Enum.KeyCode.LeftShift) then
	print("Shift is held")
end

-- Which input is the player most likely using?
local function updatePrompts()
	print("Show prompts for", UserInputService.PreferredInput.Name)
	-- KeyboardAndMouse, Gamepad, Touch or MicroGamepad
end
UserInputService:GetPropertyChangedSignal("PreferredInput"):Connect(updatePrompts)
updatePrompts()
```

`PreferredInput` changes when the player picks up a controller or plugs in a keyboard, so watch it rather than reading it once.

To choose which prompts and on-screen buttons to show, the docs recommend `PreferredInput`. `TouchEnabled` is true on a touch-screen laptop too, so it is no test for a phone.

None of this exists on the server: it has no keyboard. A choice the server must trust, such as buying or attacking, travels as a remote request.

**Key points**

- `InputBegan` and `InputEnded` fire on the client for every key, click, touch and gamepad button; the server gets no input.
- Return early when `gameProcessedEvent` is true: the engine already used that input, generally in UI.
- `KeyCode` names a key or gamepad button; test mouse clicks and touches with `UserInputType`.
- `IsKeyDown` polls a key, and `PreferredInput` says which device to show prompts for as the player switches.
