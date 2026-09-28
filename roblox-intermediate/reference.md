# Roblox Intermediate reference

*Roblox Intermediate 0.740.19.7400931, released September 22, 2026. Checked on September 27, 2026.*

---

## RemoteEvent, RemoteFunction and UnreliableRemoteEvent

Remotes are the only way code on one side calls code on the other. Every argument crosses the network as a copy, and every argument from a client is untrusted.

### `Creating a remote`

```luau
local remote = Instance.new("RemoteEvent")
remote.Name = "Greet"
remote.Parent = ReplicatedStorage
```

A remote must sit where both sides can see it, which usually means ReplicatedStorage; the docs also allow Workspace or inside a Tool. Create remotes on the server, and have the client find them with `WaitForChild`.

### `FireServer, OnServerEvent`

```luau
RemoteEvent:FireServer(...: any)  -- client only
RemoteEvent.OnServerEvent(player: Player, ...: any)  -- event, server
```

A client fires the event, and the server's handler runs without the client waiting for it. The handler's first parameter is always the Player whose client fired it; the engine adds it, and the client's own arguments follow.

> Do not pass the local player yourself: `remote:FireServer(player, 5)` arrives as `(player, player, 5)`. Client-to-server calls are throttled at about 500 a second per client, shared by every remote of the same class.

### `FireClient, FireAllClients, OnClientEvent`

```luau
RemoteEvent:FireClient(player: Player, ...: any)  -- server only
RemoteEvent:FireAllClients(...: any)  -- server only
RemoteEvent.OnClientEvent(...: any)  -- event, client
```

The server fires to one client, or to every connected client. `OnClientEvent` receives only the arguments; there is no player parameter on the client side.

> Clients cannot fire to each other. To reach another player, fire the server and let it decide whether to pass the message on.

### `InvokeServer, OnServerInvoke`

```luau
RemoteFunction:InvokeServer(...: any) -> ...any  -- client only, yields
RemoteFunction.OnServerInvoke = function(player: Player, ...: any): ...any
```

The client yields until the server's callback returns, and the return values are sent back to that client. Use a `RemoteEvent` when no answer is needed, so nothing waits.

> `OnServerInvoke` is a callback you assign, not an event you connect: assign it more than once and only the last function runs.

### `InvokeClient`

```luau
RemoteFunction:InvokeClient(player: Player, ...: any) -> ...any  -- server only, yields
```

Calls the client's `OnClientInvoke` and waits for its answer. The docs list three risks: an error on the client is thrown on the server, a client that disconnects mid-call makes it throw, and a client that never returns leaves the server waiting forever.

> A client controls its own callback, so it can use all three against the server. Send server-to-client messages with `FireClient` instead.

### `What crosses the wire`

```luau
-- tables arrive as copies
-- metatables are stripped
-- functions arrive as nil
-- an Instance the receiver cannot see arrives as nil
-- non-string table keys become strings
```

Arguments are copied across, so a received table is never the table that was sent. An Instance survives only if it is replicated to the receiver: a part in ServerStorage reaches a client as `nil`, and a part the client created reaches the server as `nil`.

> The docs also warn against tables that mix numeric and string keys, and against `nil` gaps in either kind, without saying what arrives. Send pure arrays or pure dictionaries.

### `UnreliableRemoteEvent`

```luau
Instance.new("UnreliableRemoteEvent")  -- the same methods and events as RemoteEvent
```

Messages can be dropped or arrive out of order, and a lost one is never resent. The docs recommend it for short-lived effects and continuously changing data, and a `RemoteEvent` whenever order and delivery matter.

> A payload over 1,000 bytes is dropped; in Studio the Output says by how many bytes it went over. It has its own budget of about 500 client-to-server calls a second, shared by every UnreliableRemoteEvent.

### `Checking arguments`

```luau
typeof(value: any) -> string
math.isfinite(x: number) -> boolean
```

`typeof` names Roblox types (`"Vector3"`, `"Instance"`) where `type` says only `"userdata"`, so check every argument with it. The docs' client-server boundary guide rejects numbers with `math.isfinite`, which is false for NaN and for `±math.huge`.

> `typeof(0/0)` is `"number"`, and NaN fails every comparison, so a type check plus a range check still lets it through. An Instance argument can be any object the client can see: check it with `IsA` and `IsDescendantOf` before using it.

### `GetServerTimeNow`

```luau
workspace:GetServerTimeNow() -> number
```

The client's best estimate of the server's Unix time in seconds, so every client gets roughly the same answer. It never goes backwards and runs within 0.6% of the local clock's rate.

> It errors on a client that is not connected. The docs say it is not secure for things like timed rewards: time those on the server.

## Replicated containers and RunContext

The server holds the real game. Each client gets a copy of what replicates, in no guaranteed order, and may not get distant parts at all.

### `RunContext = Client`

```luau
Script.RunContext = Enum.RunContext.Client  -- set in Studio
```

A Script with a `Client` run context runs on each player's device. The docs' "Script locations" guide recommends one in ReplicatedStorage as the single client entry point, requiring ModuleScripts, and LocalScripts only sparingly, in StarterPlayerScripts, StarterCharacterScripts, StarterGui or StarterPack.

> A `Client` Script placed in StarterPlayerScripts or StarterCharacterScripts runs twice: the original and each player's copy both run. Scripts cannot set `RunContext`; with Rojo, a `.client.luau` file under `"emitLegacyScripts": false` becomes one.

### `Players.LocalPlayer`

```luau
Players.LocalPlayer: Player  -- nil on the server
```

The Player whose device is running the code. It is how client code finds its own character, Backpack and PlayerGui.

### `Replication order`

```luau
-- 1. ReplicatedFirst loads, and its client scripts run
-- 2. the rest of the game loads; game.Loaded fires
-- 3. PlayerScripts copies and client Scripts in ReplicatedStorage run
-- 4. the character spawns
-- 5. StarterCharacterScripts copies run
```

The start-up order the docs call predictable. At step 3 a script can read what was in ReplicatedStorage from the start without `WaitForChild`; Workspace objects, and anything the server creates later, still need it.

> The engine does not guarantee the order of later changes. A property the server sets just before `FireAllClients` may reach the client before or after `OnClientEvent` fires, so wait for the change itself rather than assuming it has arrived.

### `IsLoaded, Loaded`

```luau
game:IsLoaded() -> boolean
DataModel.Loaded()  -- event, client
```

True, and fired once, when every initial instance has replicated to the client. LocalScripts outside ReplicatedFirst do not run until then, so only ReplicatedFirst code needs `if not game:IsLoaded() then game.Loaded:Wait() end`.

### `StreamingEnabled`

```luau
Workspace.StreamingEnabled: boolean  -- not scriptable
```

Instance streaming is on by default for new places created in Studio. The server then sends a client only the parts of Workspace near its character, so a distant part may not exist on that client yet.

> Look up Workspace objects on the client with `WaitForChild` and a timeout, and handle `nil`. The same goes for an instance a RemoteFunction's callback creates: it is not guaranteed to exist on the client when the call returns.

### `RejectCharacterDeletions`

```luau
Workspace.RejectCharacterDeletions: Enum.RejectCharacterDeletions  -- Default | Disabled | Enabled
```

When enabled, the server ignores a client's attempt to delete a player character from Workspace. `Default` currently means `Enabled`. It is not scriptable; set it in Studio.

## GuiObjects, UDim2, BillboardGui and SurfaceGui

Interfaces are Instances built on the client. A ScreenGui draws on the screen, and a BillboardGui or SurfaceGui draws in the world.

### `UDim2`

```luau
UDim2.new(xScale: number, xOffset: number, yScale: number, yOffset: number) -> UDim2
UDim2.fromScale(xScale: number, yScale: number) -> UDim2
UDim2.fromOffset(xOffset: number, yOffset: number) -> UDim2
```

Each axis is a scale, a fraction of the parent's size, plus an offset in pixels added after scaling. `UDim2.new(0.5, 20, 1, 0)` is half the parent's width plus 20 pixels, by the parent's full height.

> The four arguments go scale, offset, scale, offset, X then Y. `UDim2.new(0.4, 0.4)` is 40% wide and 0.4 pixels tall, not 40% by 40%.

### `Position, Size, AnchorPoint`

```luau
GuiObject.Position: UDim2
GuiObject.Size: UDim2
GuiObject.AnchorPoint: Vector2  -- (0, 0), the top-left corner
```

`AnchorPoint` is the point of the object, as a fraction of its own size, that `Position` places, and the point it grows from. An `AnchorPoint` of `(0.5, 0.5)` with a `Position` of `UDim2.fromScale(0.5, 0.5)` centres the object in its parent.

### `AbsolutePosition, AbsoluteSize`

```luau
GuiBase2d.AbsolutePosition: Vector2  -- read-only
GuiBase2d.AbsoluteSize: Vector2  -- read-only
```

Where the object actually ended up, and how big it is, in pixels, after the UDim2 values are resolved against its parent.

### `ScreenGui, ResetOnSpawn`

```luau
ScreenGui.ResetOnSpawn: boolean  -- default true
player:WaitForChild("PlayerGui")
```

A ScreenGui draws its GuiObjects only while it is inside a player's PlayerGui. StarterGui's contents are copied into PlayerGui when the character spawns; with `ResetOnSpawn` true the copy deletes itself and is copied again on every respawn.

> A ScreenGui you build in code is deleted at the next respawn unless you set `ResetOnSpawn = false`. A `false` in StarterGui only works on a direct child of StarterGui.

### `DisplayOrder, ScreenInsets`

```luau
ScreenGui.DisplayOrder: number  -- default 0
ScreenGui.ScreenInsets: Enum.ScreenInsets  -- default CoreUISafeInsets
```

A ScreenGui with a higher `DisplayOrder` draws on top of one with a lower value. The default `ScreenInsets` keeps the contents clear of the Roblox top bar and of cut-outs such as a phone's camera notch.

> The docs reserve `ScreenInsets.None` for non-interactive content such as backgrounds, since anything else can end up hidden under the core UI.

### `UIListLayout, UIPadding`

```luau
UIListLayout.FillDirection: Enum.FillDirection
UIListLayout.Padding: UDim
UIListLayout.SortOrder: Enum.SortOrder  -- LayoutOrder | Name
UIPadding.PaddingTop, PaddingBottom, PaddingLeft, PaddingRight: UDim
```

A UIListLayout arranges its sibling GuiObjects in a row or column and ignores their `Position`; each keeps its `Size`. A UIPadding insets its parent's contents from each edge.

> With `SortOrder` set to `LayoutOrder`, items that share a `LayoutOrder` go in the order they were added. Number every item's `LayoutOrder` to fix the order.

### `Activated, MouseButton1Click`

```luau
GuiButton.Activated(inputObject: InputObject, clickCount: number)  -- event
GuiButton.MouseButton1Click()  -- event
```

`Activated` fires for a left click, a touch release, or A (cross) in gamepad UI navigation, so one handler serves every device. `MouseButton1Click` fires only for a full left click with the mouse still over the button.

> `Activated` does not fire on the server. A button that changes the game must fire a remote, and the server decides.

### `TextBox.FocusLost`

```luau
TextBox.FocusLost(enterPressed: boolean, inputThatCausedFocusLoss: InputObject)  -- event
TextBox.Text: string
```

Fires when the box loses focus: the player pressed Enter, clicked or tapped elsewhere, or a script called `ReleaseFocus`. Read `Text` in the handler.

> Check `enterPressed`. Without it, clicking away from a half-typed box submits it.

### `TextSize, TextScaled, RichText`

```luau
TextLabel.TextSize: number
TextLabel.TextScaled: boolean  -- default false
TextLabel.RichText: boolean  -- default false
```

`TextSize` is the height of a line of text. `TextScaled` ignores it and scales the text to fill the label, turning on wrapping. `RichText` reads markup tags in `Text` for bold, italics, colours and more.

> For on-screen UI the docs recommend `GuiObject.AutomaticSize`, which sizes the label to its text, over `TextScaled`, which can shrink text until it is unreadable. `TextScaled` suits a label in a size-scaled BillboardGui. TextButton and TextBox have the same three properties.

### `BillboardGui`

```luau
BillboardGui.Adornee: Instance?
BillboardGui.StudsOffset: Vector3
BillboardGui.AlwaysOnTop: boolean  -- default false
BillboardGui.MaxDistance: number  -- default inf
```

UI in the world that always faces the camera, placed at its parent BasePart or Attachment, or at `Adornee` when that is set. `StudsOffset` moves it in the camera's orientation, `AlwaysOnTop` draws it over 3D objects, and past `MaxDistance` from the camera it stops rendering.

> In a BillboardGui's `Size`, the scale components are studs and the offsets are pixels. Its buttons take input only when the BillboardGui is inside PlayerGui, so put it there and point `Adornee` at the part.

### `SurfaceGui`

```luau
SurfaceGui.Adornee: Instance?
SurfaceGui.Face: Enum.NormalId
```

UI drawn flat on one face of a part: its parent, or `Adornee` when that is set. The docs recommend scale values for its children, so they fit whatever size the face is.

> Its buttons take input only when three things hold: the SurfaceGui is inside PlayerGui, `Adornee` points at the part, and the part's `CanQuery` is true.

## UserInputService and the Input Action System

Both run only in client code. UserInputService reports raw input from every device; the Input Action System turns inputs into named actions you bind once per device.

### `InputBegan, InputEnded`

```luau
UserInputService.InputBegan(input: InputObject, gameProcessedEvent: boolean)  -- event
UserInputService.InputEnded(input: InputObject, gameProcessedEvent: boolean)  -- event
```

Fire when a key, button, click or touch starts and stops, while the Roblox window has focus. `gameProcessedEvent` is true when the engine already acted on the input, generally UI: a button was clicked or touched with it.

> Return early when `gameProcessedEvent` is true, or a keypress meant for the interface also triggers your game action.

### `KeyCode, UserInputType`

```luau
InputObject.KeyCode: Enum.KeyCode
InputObject.UserInputType: Enum.UserInputType
```

`UserInputType` is the kind of input: `Keyboard`, `MouseButton1`, `Touch`, `Gamepad1` and so on. `KeyCode` says which key or button; for mouse input it carries no information, so test clicks with `UserInputType`.

### `IsKeyDown`

```luau
UserInputService:IsKeyDown(keyCode: Enum.KeyCode) -> boolean
```

Whether that keyboard key is held right now. Use it to poll a key each frame instead of tracking `InputBegan` and `InputEnded` yourself.

### `PreferredInput`

```luau
UserInputService.PreferredInput: Enum.PreferredInput  -- KeyboardAndMouse | Gamepad | Touch | MicroGamepad
```

The input the player is most likely using, which changes as they plug in or pick up another device. The docs recommend it for choosing which prompts and on-screen buttons to show.

> `TouchEnabled` is true on touch-screen laptops, so it is not an "is this a phone" check. Watch `PreferredInput` with `GetPropertyChangedSignal`.

### `InputContext`

```luau
InputContext.Enabled: boolean  -- default true
InputContext.Priority: number  -- default 1000
InputContext.Sink: boolean  -- default false
```

A group of related actions, such as gameplay controls or menu navigation, that you turn on and off together. Contexts with a higher `Priority` run first, and a context with `Sink` on stops lower-priority contexts from receiving the inputs it binds.

> Contexts with the same priority all receive the input, whatever `Sink` says. A context nested inside another has no effect. The guide keeps contexts in a Folder in ReplicatedStorage and handles actions in `Client` Scripts.

### `InputAction`

```luau
InputAction.Type: Enum.InputActionType  -- Bool (default) | Direction1D | Direction2D | Direction3D | ViewportPosition
InputAction.Enabled: boolean
InputAction:GetState() -> any
```

One thing the player can do, such as Jump or Interact. It registers with its nearest `InputContext` ancestor, or a default context if it has none. `Bool` suits press-and-release actions, and `GetState` returns the current value, `true` or `false` for `Bool`.

### `Pressed, Released, StateChanged`

```luau
InputAction.Pressed()  -- event
InputAction.Released()  -- event
InputAction.StateChanged(value: any)  -- event
```

On a `Bool` action, `Pressed` fires when the state goes from false to true and `Released` when it goes back. `StateChanged` fires for every action type whenever the state changes, and passes the new value.

> None of them fires again for the same state. A thumbstick held at one angle fires `StateChanged` once, so for continuous input poll `GetState()` every frame.

### `InputBinding`

```luau
InputBinding.KeyCode: Enum.KeyCode  -- E, ButtonX, MouseLeftButton, …
InputBinding.UIButton: GuiButton?
```

Parented to an `InputAction`, a binding names one input that drives it, and an action can have several. The guide gives each action one binding for keyboard and mouse, one for gamepad and one for touch, where `UIButton` links an on-screen GuiButton to a `Bool` action.

> The input must suit the action's `Type`. A mismatch either never fires the action or hands `StateChanged` a converted value.

## Tool and Backpack

A Tool waits in the player's Backpack and moves into the character when equipped. Its input events happen on the client, and the server decides what they do.

### `Tool, Handle, RequiresHandle`

```luau
Tool.RequiresHandle: boolean  -- default true
tool:FindFirstChild("Handle")  -- the BasePart held in the right hand
```

A tool's handle is a child BasePart named `Handle`, which the character holds. A tool with no handle, such as a spell, needs `RequiresHandle` set to false.

> With `RequiresHandle` true and no `Handle`, the tool does not work and `Equipped` and `Unequipped` never fire.

### `Equipped, Unequipped`

```luau
Tool.Equipped(mouse: Mouse)  -- event
Tool.Unequipped()  -- event
```

Fire when the player takes the tool out of the Backpack and when they put it away or switch tools. Equipping moves the tool from the Backpack into the character model.

> The `mouse` argument is the legacy Mouse object; aim with UserInputService and the camera instead (see Camera).

### `Activated, Deactivated`

```luau
Tool.Activated()  -- event
Tool.Deactivated()  -- event
```

`Activated` fires when the player clicks, taps or presses the right trigger while the tool is equipped, and `Deactivated` when they let go. Neither passes where the player aimed.

> The tools guide says tool events fire only in client code, so most tools need a LocalScript, a server Script and a RemoteEvent between them. The server treats that remote like any other: it checks the tool is equipped and applies a cooldown.

### `Is it equipped?`

```luau
tool.Parent == player.Character  -- equipped
tool.Parent == player.Backpack  -- carried
```

The tools guide's pattern: its server Scripts handle `Equipped`, so they count on the server seeing the tool move into the character. Following it, a tool's remote handler compares `tool.Parent` with the firing player's `Character` to tell whether the tool is in use.

### `StarterPack, Backpack`

```luau
game:GetService("StarterPack")
player:WaitForChild("Backpack")
```

Tools in StarterPack are copied into each player's Backpack when their character spawns, and the Backpack shows as the hotbar. When the character dies, the Backpack is replaced and filled again from StarterPack.

> A tool the server clones into a player's Backpack is gone after the next death, because the new Backpack is filled only from the starter containers. Give it again on `CharacterAdded` if it should last.

## Camera

Each client has its own Camera, which never replicates. Client code reads it to aim, and can take it over.

### `CurrentCamera`

```luau
workspace.CurrentCamera: Camera
```

The Camera the local player is looking through. Use this property rather than searching Workspace for a child named `Camera`.

> Setting `CurrentCamera` destroys every other Camera in Workspace, the previous one included.

### `CameraType, CFrame`

```luau
Camera.CameraType: Enum.CameraType  -- Custom is the default player camera
Camera.CFrame: CFrame
```

The default camera scripts set `CFrame` every frame, so set `CameraType` to `Scriptable` first; they then leave the camera alone. Place it with `CFrame.lookAt(position, target)`.

> Set `CameraType` back to `Custom` to hand the camera back to the default scripts.

### `ViewportPointToRay, ScreenPointToRay`

```luau
Camera:ViewportPointToRay(x: number, y: number, depth: number?) -> Ray
Camera:ScreenPointToRay(x: number, y: number, depth: number?) -> Ray
```

A ray from the camera through a point on screen, in pixels. `ViewportPointToRay` measures from the top-left of the top bar; `ScreenPointToRay` measures from below it, the same coordinates GUI objects use.

> Both return a unit ray, one stud long. Pass `ray.Direction * range` to `workspace:Raycast`, not `ray.Direction`.

### `GetMouseLocation`

```luau
UserInputService:GetMouseLocation() -> Vector2
```

The mouse position in pixels from the top-left corner, without the GUI inset, so it pairs with `ViewportPointToRay`. An `InputObject.Position` from a mouse event already has the inset applied and pairs with `ScreenPointToRay`.

> The position is undefined when the mouse is off screen or the device has no mouse.

## WorldRoot queries, RaycastParams and OverlapParams

Ask the world what is along a line, along a moving shape, or inside a volume. Each query takes a reusable params object that says which parts count.

### `Raycast`

```luau
WorldRoot:Raycast(origin: Vector3, direction: Vector3, raycastParams: RaycastParams?) -> RaycastResult?
```

Casts a ray from `origin` and returns the first eligible part or Terrain cell it hits, or `nil` on a miss. The direction's length is the range, up to 15,000 studs.

> Anything further than the direction's length is not tested, so `(target - origin).Unit` checks only the first stud. Multiply a unit direction by the range you want.

### `RaycastResult`

```luau
RaycastResult.Instance: BasePart
RaycastResult.Position: Vector3
RaycastResult.Normal: Vector3
RaycastResult.Distance: number
RaycastResult.Material: Enum.Material
```

What was hit, the world point where the ray met its surface, that face's normal, how far from the origin it was, and the material there.

### `RaycastParams`

```luau
RaycastParams.new() -> RaycastParams
RaycastParams.ExcludeInstances: {Instance}?
RaycastParams.IncludeInstances: {Instance}?
```

Which parts a raycast or shapecast may hit. Each list covers the instances in it and all their descendants, and the two can be combined; an instance in both is excluded. Params can be changed and reused rather than rebuilt for every cast.

> `IncludeInstances = nil` allows everything, but `{}` allows nothing. The datatype docs call `FilterType` and `FilterDescendantsInstances` superseded by these two, although the raycasting guide still shows them.

### `Spherecast, Blockcast, Shapecast`

```luau
WorldRoot:Spherecast(position: Vector3, radius: number, direction: Vector3, params: RaycastParams?) -> RaycastResult?
WorldRoot:Blockcast(cframe: CFrame, size: Vector3, direction: Vector3, params: RaycastParams?) -> RaycastResult?
WorldRoot:Shapecast(part: BasePart, direction: Vector3, params: RaycastParams?) -> RaycastResult?
```

Sweep a sphere, a box or a part's own shape along `direction` and return the first hit. `Distance` is how far the shape travelled, and `Position` is the point of contact.

> They do not detect parts the shape already overlaps at the start. The shape may travel at most 1,024 studs, and a sphere's radius is capped at 256.

### `GetPartBoundsInRadius, GetPartBoundsInBox`

```luau
WorldRoot:GetPartBoundsInRadius(position: Vector3, radius: number, overlapParams: OverlapParams?) -> {BasePart}
WorldRoot:GetPartBoundsInBox(cframe: CFrame, size: Vector3, overlapParams: OverlapParams?) -> {BasePart}
```

Every part whose bounding box overlaps a sphere or a box. They test bounding boxes, not true shapes, which makes them fast but loose for spheres, cylinders and meshes; `GetPartsInPart` does an exact check.

> A character is many parts, so one character shows up many times. Map each part to its model and keep a set of the models you have seen.

### `OverlapParams`

```luau
OverlapParams.new() -> OverlapParams
OverlapParams.ExcludeInstances: {Instance}?
OverlapParams.IncludeInstances: {Instance}?
OverlapParams.MaxParts: number  -- 0 means no limit
```

The params for the overlap queries: the same include and exclude lists as `RaycastParams`, plus a cap on how many parts come back.

## PathfindingService, Path and Humanoid:MoveTo

PathfindingService computes a route around obstacles as a list of waypoints. Walking the route is up to you.

### `CreatePath`

```luau
PathfindingService:CreatePath(agentParameters: {
  AgentRadius: number?,      -- 2
  AgentHeight: number?,      -- 5
  AgentCanJump: boolean?,    -- true
  AgentCanClimb: boolean?,   -- false
  WaypointSpacing: number?,  -- 4
  Costs: {[string]: number}?,
}?) -> Path
```

A Path for an agent of the given size and abilities. Radius and height are the space the agent needs to fit through, and `Costs` makes some materials or regions dearer to cross.

### `ComputeAsync`

```luau
Path:ComputeAsync(start: Vector3, finish: Vector3)  -- yields
```

Computes the route. It is not called when the path is created, and must be called again whenever the route needs updating.

> The docs' sample wraps it in `pcall` and then checks `Status`. It fails when the straight-line distance exceeds 3,000 studs or the search runs out of nodes. Client code can also fail to reach a destination that has not streamed in; the server sees the whole world.

### `Status`

```luau
Path.Status: Enum.PathStatus  -- Success | NoPath
```

`Success` after a route was found. `NoPath` when obstacles leave no way through, or a point is inside a solid object.

> The enum's other four items are deprecated in the dump. Compare against `Success`.

### `GetWaypoints`

```luau
Path:GetWaypoints() -> {PathWaypoint}
PathWaypoint.Position: Vector3
PathWaypoint.Action: Enum.PathWaypointAction  -- Walk | Jump | Custom
```

The route as points from start to finish, each with the action for reaching it. The array is empty if no path was computed.

> When the next waypoint's `Action` is `Jump`, make the humanoid jump (`humanoid.Jump = true`) as you move towards it.

### `Blocked`

```luau
Path.Blocked(blockedWaypointIdx: number)  -- event
```

Fires when something blocks the computed route, with the index of the blocked waypoint in the `GetWaypoints` array.

> It also fires for blockages behind the agent. Compute again only when the index is at or after the waypoint you are heading to, and disconnect the old connection first.

### `MoveTo`

```luau
Humanoid:MoveTo(location: Vector3, part: BasePart?)
```

Makes the humanoid walk towards `location`, or towards that point relative to `part` if the part moves. The walk ends on arrival within about a stud, after 8 seconds without arriving, or when another `MoveTo`, a `Move` call or a new root `CFrame` replaces it.

> The 8-second limit is per call, so a long walk must be broken up; calling `MoveTo` again resets it. The pathfinding guide's sample drives a player's own character with `Humanoid:Move` instead.

### `MoveToFinished`

```luau
Humanoid.MoveToFinished(reached: boolean)  -- event
```

Fires when a `MoveTo` walk ends. `reached` is true if the humanoid got there, and false if it stopped after 8 seconds.

## Animator, AnimationTrack and the Audio API

An Animator plays animation tracks on a rig. Sound is a graph: players produce audio, wires carry it, and outputs or emitters make it heard.

### `Animator:LoadAnimation`

```luau
Animator:LoadAnimation(animation: Animation) -> AnimationTrack
Animation.AnimationId: ContentId  -- "rbxassetid://…"
```

Loads an `Animation` onto the rig's Animator, which sits inside its Humanoid (or AnimationController), and returns a playable track.

> Every call makes a new track. Load once and keep the track, or look one up with `GetTrackByAnimationId`. The Animator must be in Workspace, or the call errors.

### `Which side plays it`

```luau
-- a player's own character: play on that player's client
-- anything else (NPCs, props): play on the server
```

Tracks a client plays on the Animator of its own character replicate to the server and every other client. On any other rig, animations must be loaded and played on the server to replicate.

> The Animator itself must be created on the server. Tracks loaded on an Animator created on the client never replicate.

### `Play, Stop`

```luau
AnimationTrack:Play(fadeTime: number?, weight: number?, speed: number?)  -- 0.1, 1, 1
AnimationTrack:Stop(fadeTime: number?)  -- 0.1
```

`Play` fades the track in over `fadeTime` seconds at the given weight and speed. `Stop` fades it out over `fadeTime`.

> Calling `Play` on a track that is already playing restarts it. Check `IsPlaying` first.

### `Length, IsPlaying, Looped`

```luau
AnimationTrack.Length: number  -- read-only
AnimationTrack.IsPlaying: boolean  -- read-only
AnimationTrack.Looped: boolean
```

The track's length in seconds at speed 1, whether it is playing, and whether it repeats. `Looped` starts as the Animation Editor saved it, and a change made while playing takes effect when the current run ends.

> `Length` is 0 until the animation has loaded.

### `Stopped, Ended, markers`

```luau
AnimationTrack.Stopped()  -- event
AnimationTrack.Ended()  -- event
AnimationTrack:GetMarkerReachedSignal(name: string) -> RBXScriptSignal
```

`Stopped` fires when the track finishes playing, possibly still fading out; `Ended` fires once it no longer moves anything. The marker signal fires at each named marker set in the Animation Editor.

### `AudioPlayer`

```luau
AudioPlayer.Asset: ContentId
AudioPlayer:Play(atTime: number?)
AudioPlayer:Stop(atTime: number?)
AudioPlayer.IsPlaying: boolean  -- read-only
AudioPlayer.Looping: boolean
AudioPlayer.TimePosition: number
```

Loads and plays an audio asset. It produces a stream on its one `Output` pin and is silent until a Wire connects that pin to something heard.

> `Play` starts from the current `TimePosition` and `Stop` leaves it where it is, so set `TimePosition = 0` to start over. `IsPlaying` cannot be set; call the methods. `Ended` does not fire after `Stop` or while `Looping`.

### `Wire`

```luau
Wire.SourceInstance: Instance?
Wire.TargetInstance: Instance?
Wire.Connected: boolean  -- read-only
```

Carries a stream from the source's `Output` pin to the target's `Input` pin; `SourceName` and `TargetName` default to those pin names. `Connected` is true once both ends are set, both pins exist, and the graph has no cycle.

### `AudioDeviceOutput`

```luau
AudioDeviceOutput.Player: Player?
```

The real speakers or headphones: every stream wired to it is heard. For 2D sound the docs' recipe is an AudioPlayer, a Wire and an AudioDeviceOutput, placed in SoundService.

> `Player` names the one player the streams are meant for. Left `nil`, everyone hears them.

### `AudioEmitter`

```luau
AudioEmitter  -- parent to a BasePart, Attachment or Camera
```

A virtual speaker for 3D sound: streams wired to it are broadcast from its parent's position, and get quieter with distance. Wire an AudioPlayer into it.

> Under a parent that is not an Attachment, a Camera or a PVInstance, it is silent.

### `AudioListener`

```luau
AudioListener  -- parent to an Attachment, Camera or PVInstance
SoundService.DefaultListenerLocation: Enum.ListenerLocation  -- Default | None | Character | Camera
```

A virtual microphone that picks up nearby emitters; wire it to an AudioDeviceOutput to hear what it picks up. `DefaultListenerLocation` set to `Character` or `Camera` makes the engine create a listener there, with the output and wire, for each player.

> `Default` depends on the voice chat settings, so set `Character` or `Camera` in Studio if the place needs 3D sound. Game scripts cannot set it.

## Ecosystem

| Package | Role | Version | Install |
| --- | --- | --- | --- |
| [Blink](https://github.com/1Axen/blink) | Networking IDL: typed, compact remotes generated from one schema file | 0.18.8 | `rokit add 1Axen/blink` |
| [Fusion](https://github.com/dphfox/Fusion) | Reactive UI library, and Kalo's course on it | 0.3.0 | `Fusion = "elttob/fusion@0.3.0"` |
| [Vide](https://github.com/centau/vide) | Fine-grained reactive UI library, and Kalo's course on it | 0.4.1 | `vide = "centau/vide@0.4.1"` |
| [Fluid](https://github.com/ffrostfall/fluid) | Declarative UI library rewritten from Vide, and Kalo's course on it | 0.7.0-rc.7 | `fluid = "ffrostfall/fluid@=0.7.0-rc.7"` |

## Out of date elsewhere

Shapes you may still meet in older tutorials:

- **`TextScaled = true` on on-screen text.** The `TextLabel.TextScaled` docs recommend avoiding it for on-screen UI: set a `TextSize` and let `AutomaticSize` grow the element to fit.
- **`player:GetMouse()`, `mouse.Hit` and `mouse.Target`.** The dump does not deprecate them, but the docs call `Mouse` a legacy object that UserInputService supersedes. Aim with `UserInputService:GetMouseLocation()`, `camera:ViewportPointToRay(x, y)` and `workspace:Raycast`, which also give you real filtering.
- **`ContextActionService:BindAction(name, fn, true, Enum.KeyCode.E)`.** Not deprecated, and its page carries no notice. Roblox's input guides now teach the Input Action System for actions on every device, so this course binds an `InputAction` inside an `InputContext`, with one `InputBinding` per device, instead of a callback with a generated touch button.
- **`Sound`, `SoundGroup` and `SoundEffect`.** Not deprecated in the dump, but the audio objects guide says they are "now discouraged in favor of the more robust functionality of audio objects". Play an `AudioPlayer` wired to an `AudioDeviceOutput` for 2D sound, or to an `AudioEmitter` for 3D sound.
- **A tool's server Script listening for `Activated`.** Older gear does this. The tools guide says tool events only fire in client-side code, because only the player's device knows when input happens. Listen in a LocalScript, send the request through a RemoteEvent, and validate it in the server Script.
- **`humanoid:LoadAnimation(animation)`.** Deprecated. Load through the Animator instead: `humanoid:WaitForChild("Animator"):LoadAnimation(animation)`, which returns the same kind of AnimationTrack.
- **`workspace:FindPartOnRay(ray, ignore)`, `FindPartOnRayWithIgnoreList` and `FindPartOnRayWithWhitelist`.** Deprecated. They take a `Ray` and return a tuple. `workspace:Raycast(origin, direction, params)` replaces all three and returns a `RaycastResult`, or `nil` on a miss.
- **`Enum.RaycastFilterType.Blacklist` and `Whitelist`, and `FilterDescendantsInstances` with `FilterType`.** Blacklist and Whitelist are the legacy names of `Exclude` and `Include`. The docs now call `FilterType` and `FilterDescendantsInstances` superseded too: set `params.ExcludeInstances = { character }` or `params.IncludeInstances = { folder }` on `RaycastParams` and `OverlapParams`.
- **`workspace:FindPartsInRegion3(region)` and its `WithIgnoreList`, `WithWhiteList` and `IsRegion3Empty` variants.** Deprecated. `workspace:GetPartBoundsInBox(cframe, size, params)` replaces them, and its box can be rotated. `Region3` itself is still used by Terrain APIs.
- **`frame:TweenSize(…)`, `TweenPosition` and `TweenSizeAndPosition`.** Deprecated in favour of TweenService: `TweenService:Create(frame, TweenInfo.new(0.3), { Size = UDim2.fromScale(1, 0.1) }):Play()`. The old `override` argument has no replacement because a new tween always overrides one on the same property, and the callback becomes the tween's `Completed` event.
- **A LocalScript that changes the world and expects everyone to see it.** Tutorials from when `FilteringEnabled` could be switched off do this. The property is deprecated and no longer takes effect: a client's changes stay on that client. Ask the server with a remote, and let it validate and apply the change.
