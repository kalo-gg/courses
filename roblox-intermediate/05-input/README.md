# 5. Input

*Input on every device, tools in the backpack, and the camera the player looks through.*

Roblox Intermediate · Medium

---

Keys, clicks, taps and gamepad buttons happen on the player's device, so input code is client code. This module starts with UserInputService, which reports every input along with whether the game already used it. It then binds one named action to a key, a gamepad button and an on-screen button with the Input Action System, and splits a tool into the client half that notices the click and the server half that checks it. It ends with the camera: taking it over, and turning the mouse position into a ray in the world.

## Lessons

1. [UserInputService](01-user-input.md) — Every key, click, touch and gamepad button reaches the client as an InputObject, with a flag that says whether the game already used it.
2. [Actions on every device](02-actions.md) — An InputContext holds InputActions, and each action has one InputBinding per device, so one handler serves keyboard, gamepad and touch.
3. [Tools and the backpack](03-tools-and-backpack.md) — A Tool moves from the Backpack into the character when equipped; its clicks happen on the client, and the server checks every request that follows.
4. [The camera](04-the-camera.md) — Each client has its own camera: take it over with a Scriptable CameraType, and turn a point on the screen into a ray in the world.
