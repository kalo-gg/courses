# Buttons and text

*Buttons that answer every device, text boxes that submit only on Enter, and text sized to stay readable.*

Roblox Intermediate · Interface · 11 min

---

A shop's Buy button is wired up in a server Script, and clicking it does nothing. A name box saves whatever is in it when the player clicks away, half-typed names included.

Both bugs come from how UI events fire. They fire on the client, and each one says more than "something happened": which input caused it, or whether Enter was pressed. This lesson uses both, then sizes text so it stays readable.

## Buttons fire Activated

**main.client.luau**

```luau
local Players = game:GetService("Players")
local ReplicatedStorage = game:GetService("ReplicatedStorage")

local buy = ReplicatedStorage:WaitForChild("Buy")
local shop = Players.LocalPlayer:WaitForChild("PlayerGui"):WaitForChild("Shop")
local button = shop:WaitForChild("Panel"):WaitForChild("BuySword")

button.Activated:Connect(function()
	buy:FireServer("Sword")
end)
```

The client hears the click and asks. The server looks up the price, checks the player's coins, and decides, as Module 3 taught.

`GuiButton.Activated` fires for a left click on a desktop, a tap on a touch screen, and A (cross) on a gamepad in UI navigation, so one handler serves every device. The docs say it does not fire on the server.

`MouseButton1Click` is documented only as a full left click of the mouse, released while still over the button. It is not deprecated, but for a button any player might press, connect `Activated`.

## Text boxes and text

A `TextBox` fires `FocusLost` whenever it stops being typed in: the player pressed Enter, clicked or tapped somewhere else, or a script called `ReleaseFocus`. Its first argument, `enterPressed`, says which. Submit only when it is true, and read the box's `Text` in the handler.

**Text that stays readable**

```luau
local price = Instance.new("TextLabel")
price.TextSize = 24 -- the height of one line, in pixels
price.AutomaticSize = Enum.AutomaticSize.X -- widen to fit the text
price.Size = UDim2.fromOffset(0, 32) -- now the minimum size
price.RichText = true
price.Text = 'Sword: <b>50</b> <font color="#FFD700">gold</font>'
```

`AutomaticSize` grows the label to fit its text, so `Size` becomes the smallest it can be. `RichText` reads markup tags in `Text` for bold, italics, colours and more; write `&lt;` to show a literal `<`.

> **Recognising older code**
>
> `label.TextScaled = true` on on-screen UI is common in older code. It ignores `TextSize` and stretches the text to fill the label, which can shrink it until it is unreadable. The docs recommend `TextSize` with `AutomaticSize` instead. `TextScaled` still suits a label in a BillboardGui sized in studs, as the next lesson shows.

> **Filter what players write**
>
> Text a player typed must pass through Roblox's text filtering before any other player sees it, on a name tag, a sign or anywhere else. The server filters it; Roblox Advanced's *Text, chat and safety* lesson teaches how. Until then, keep what players type to themselves, as the exercise's server does.

**Key points**

- `Activated` fires for a click, a tap or a gamepad press, and never on the server: a button that changes the game fires a remote.
- `TextBox.FocusLost(enterPressed)` fires however focus is lost; submit only when `enterPressed` is true.
- Size text with `TextSize` and let `AutomaticSize` fit the object to it; the docs recommend avoiding `TextScaled` for on-screen UI.
- `RichText` turns on markup such as `<b>` and `<font color>` inside `Text`.
- Text a player typed is filtered on the server before anyone else sees it.
