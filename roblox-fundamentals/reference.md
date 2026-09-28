# Roblox Fundamentals reference

*Roblox Fundamentals 0.740.19.7400931, released September 22, 2026. Checked on September 27, 2026.*

---

## Instance

Everything in a place is an Instance in one tree rooted at `game`. These members create, find, copy, remove and label them.

### `Instance.new`

```luau
Instance.new(className: string, parent: Instance?) -> Instance
```

Creates a new instance of the class, with no parent unless you pass one. Abstract classes such as `BasePart`, and services, cannot be created.

> The docs call the `parent` argument discouraged. Set the properties first and `Parent` last, so the new object replicates once instead of once per property change.

### `Name, ClassName`

```luau
Instance.Name: string
Object.ClassName: string  -- read-only
```

`Name` is a non-unique label of up to 100 characters, and it is what dot indexing uses. `ClassName` is the class the object was created as, and scripts cannot change it.

> When siblings share a name, indexing by that name returns one of them, not always the one you want.

### `Parent`

```luau
Instance.Parent: Instance?
```

Where the instance sits in the tree. Setting it moves the instance and its descendants. Set to `nil`, the instance leaves the place but survives while a variable or property still refers to it.

### `IsA`

```luau
Object:IsA(className: string) -> boolean
```

True when the object's class is `className` or inherits from it, so `part:IsA("BasePart")` is true for a `Part`, a `MeshPart` or a `SpawnLocation`. Compare `ClassName` instead when inheritance must not count.

> In the 0.740 dump `IsA`, `ClassName`, `Changed` and `GetPropertyChangedSignal` belong to `Object`, the class above `Instance`. Every instance still has them.

### `FindFirstChild`

```luau
Instance:FindFirstChild(name: string, recursive: boolean?) -> Instance?
```

Returns the first child with that name, or `nil`. With `recursive` set to `true` it searches every descendant, not just the children.

> Dot indexing throws when the child is missing, and it also loses to any property of the same name: `part.Color` is the property even if the part has a child called `Color`. `FindFirstChild` returns `nil` instead of throwing, and indexing that `nil` is the `attempt to index nil with …` error.

### `FindFirstChildOfClass, FindFirstChildWhichIsA`

```luau
Instance:FindFirstChildOfClass(className: string) -> Instance?
Instance:FindFirstChildWhichIsA(className: string, recursive: boolean?) -> Instance?
```

Find a child by class rather than name. `OfClass` matches `ClassName` exactly; `WhichIsA` uses `IsA`, so `"BasePart"` finds any kind of part.

> Only `WhichIsA` takes a `recursive` argument.

### `FindFirstAncestorOfClass`

```luau
Instance:FindFirstAncestorOfClass(className: string) -> Instance?
Instance:FindFirstAncestorWhichIsA(className: string) -> Instance?
```

Walks up from `Parent` towards the DataModel and returns the first ancestor whose class matches, or `nil`. `part:FindFirstAncestorOfClass("Model")` finds the model a part belongs to.

### `GetChildren, GetDescendants`

```luau
Instance:GetChildren() -> {Instance}
Instance:GetDescendants() -> {Instance}
```

An array of the direct children, or of every descendant at any depth.

> The docs say neither array is sorted in any particular order. Sort it yourself if order matters.

### `WaitForChild`

```luau
Instance:WaitForChild(childName: string, timeOut: number?) -> Instance?
```

Returns the named child, yielding until it exists. It does not yield if the child is already there. With `timeOut` it gives up after that many seconds and returns `nil`.

> With no `timeOut`, a call still waiting after 5 seconds prints a warning to the Output: `Infinite yield possible on …`. It is a warning, not an error: the thread keeps waiting.

### `Clone, Archivable`

```luau
Instance:Clone() -> Instance?
Instance.Archivable: boolean
```

`Clone` copies an instance and all its descendants, skipping any that are not `Archivable`. The copy's `Parent` is `nil`, so set it last, as with `Instance.new`.

> Cloning an instance whose own `Archivable` is `false` returns `nil`. A reference property that points inside the cloned tree points at the copy; one that points outside keeps its target.

### `Destroy`

```luau
Instance:Destroy()
```

Sets `Parent` to `nil`, locks `Parent`, disconnects all the instance's connections and calls `Destroy` on every child. The docs call it the correct way to dispose of an object you no longer need.

> Setting `Parent` on a destroyed instance errors, because it is locked. To take an object out and put it back later, set `Parent = nil` instead of destroying it.

### `Attributes`

```luau
Instance:SetAttribute(attribute: string, value: any)
Instance:GetAttribute(attribute: string) -> any
Instance:GetAttributes() -> {[string]: any}
```

Your own named values on any instance, saved with the place and replicated. `SetAttribute` creates or changes one, and `nil` removes it. `GetAttribute` returns `nil` when the attribute is not set.

> The allowed types are string, boolean, number, `UDim`, `UDim2`, `BrickColor`, `Color3`, `Vector2`, `Vector3`, `CFrame`, `NumberSequence`, `ColorSequence`, `NumberRange`, `Rect` and `Font`. Any other type, an Instance included, errors. A name may use only letters, digits, `.`, `-`, `/` and `_`, may be at most 100 characters, and may not start with `RBX`.

### `Tags`

```luau
Instance:AddTag(tag: string)
Instance:HasTag(tag: string) -> boolean
Instance:RemoveTag(tag: string)
```

String labels on an instance. `AddTag` does nothing if the tag is already there, and `RemoveTag` does not error if it is missing. Adding or removing a tag fires the matching `CollectionService` signal.

> Whatever you set up for a tagged instance (connections, table entries, loops) should be cleaned up when the tag is removed or the instance is destroyed. The docs call this out to prevent leaks.

### `CollectionService:GetTagged`

```luau
CollectionService:GetTagged(tag: string) -> {Instance}
```

Every instance with the tag that is currently a descendant of the DataModel, in no guaranteed order. A tagged instance whose `Parent` is `nil` is not returned.

> It sees only what is tagged now. To handle instances tagged later as well, pair it with `GetInstanceAddedSignal` (see Signals and connections).

## Services and containers

`game` is the DataModel. Its children are services, and the container services decide whether a script runs and who can see an object.

### `game, workspace, script`

```luau
game: DataModel
workspace: Workspace
script: LuaSourceContainer
```

Three globals. `game` is the root of the tree, `workspace` is the Workspace service, and `script` is the Script, LocalScript or ModuleScript whose code is running.

### `GetService`

```luau
game:GetService(className: string) -> Instance
```

Returns the service with that class name, creating it if it does not exist yet. It looks up by class, so it works for services whose `Name` differs from their class: RunService's name is "Run Service". It is the only way to create some services.

> It returns `nil` for a class that exists but is not a service. By convention, get each service once at the top of the script and name the variable after it.

### `Workspace`

```luau
game:GetService("Workspace")
```

Holds the 3D world. A `BasePart` is rendered and simulated only while it is a descendant of Workspace. A server `Script` runs here.

> Parts that fall below `Workspace.FallenPartsDestroyHeight` are cleaned up automatically.

### `ServerScriptService`

```luau
game:GetService("ServerScriptService")
```

Server-only and never replicated to clients. Scripts here run, which makes it the place for the entry `Script` and its server ModuleScripts.

### `ServerStorage`

```luau
game:GetService("ServerStorage")
```

Server-only and not replicated. Scripts here do not run, but ModuleScripts can be required. It is the place for templates you `Clone`, and large objects such as maps until they are needed.

### `ReplicatedStorage`

```luau
game:GetService("ReplicatedStorage")
```

Replicated to every client. It is the place for ModuleScripts that both server and client require. A default `Script` and a `LocalScript` do not run here.

### `StarterPlayer, StarterGui`

```luau
game:GetService("StarterPlayer")
game:GetService("StarterGui")
```

Starter containers are copied into each player. `StarterPlayer` sets default properties for each Player, such as `CharacterUseJumpPower`, and holds `StarterPlayerScripts` (copied to each player on join) and `StarterCharacterScripts` (copied into the character on each spawn). `StarterGui` holds UI copied into each player's `PlayerGui`.

> A `LocalScript` placed in a Starter container runs as its copy, on the player's device.

### `ReplicatedFirst, Lighting`

```luau
game:GetService("ReplicatedFirst")
game:GetService("Lighting")
```

`ReplicatedFirst` replicates to clients before anything else, which suits the minimum needed for a loading screen. `Lighting` controls the place's global lighting.

### `Script`

```luau
Script.RunContext: Enum.RunContext  -- Legacy | Server | Client | Plugin
```

Runs Luau on the server. A new Script's run context is `Legacy`: it runs only while it is enabled and inside a server container such as `Workspace` or `ServerScriptService`. Set `RunContext` to `Server` and it can also run in ReplicatedStorage, which the docs' "Script locations" guide advises against. Every exercise in this course runs as a `Server` Script in ServerScriptService, the layout the guide recommends.

> Scripts cannot set `RunContext` (it needs plugin security), so set it in Studio. The docs' recommended layout is one Script with an explicit `Server` run context in ServerScriptService; Roblox Intermediate covers run contexts.

### `LocalScript`

```luau
LocalScript  -- runs on the player's device
```

Runs on the client, where it can reach client-only objects such as the camera. It runs from a player's copies of the Starter containers, from the character and from `ReplicatedFirst`, but not from ReplicatedStorage, ServerStorage or ServerScriptService.

### `ModuleScript, require`

```luau
require(module: ModuleScript) -> any
```

A ModuleScript returns exactly one value, usually a table of functions. It runs once per Luau environment: later `require` calls on the same side get the same value back. Server and client each run their own copy.

> Two modules that require each other never finish loading. The docs disagree on how that fails: the class reference says the thread hangs with no error, and the modules guide says it errors with `Requested module was required recursively`.

## Signals and connections

Every event is an `RBXScriptSignal`. Connecting a function to it returns an `RBXScriptConnection`, which is how you stop listening.

### `Connect`

```luau
RBXScriptSignal:Connect(func: (...any) -> ()) -> RBXScriptConnection
```

Calls `func` every time the event fires, with the event's arguments. Returns the connection.

### `Once`

```luau
RBXScriptSignal:Once(func: (...any) -> ()) -> RBXScriptConnection
```

Like `Connect`, but only the first firing is delivered. The connection is disconnected before `func` is called.

> Under deferred signals, several firings can queue up before a handler gets to disconnect itself. The docs suggest `Once` for taking only the first.

### `Wait`

```luau
RBXScriptSignal:Wait() -> ...any
```

Yields the current thread until the event next fires, then returns that firing's arguments.

### `Disconnect, Connected`

```luau
RBXScriptConnection:Disconnect()
RBXScriptConnection.Connected: boolean
```

`Disconnect` stops the handler, and `Connected` becomes `false`. Disconnecting also drops any handler calls that are still queued.

> Destroying the instance disconnects its signals too, but unlike `Disconnect` it still runs handler calls that were already queued.

### `SignalBehavior`

```luau
Workspace.SignalBehavior: Enum.SignalBehavior  -- Default | Immediate | Deferred | AncestryDeferred
```

With `Deferred`, a handler does not run inside the code that fired the event. It runs at the next resumption point, such as `task.wait` resuming or a RunService event. With `Immediate`, it runs at once. The docs recommend `Deferred`, and template places are set to it.

> The `Default` item currently means `Immediate` and will eventually mean `Deferred`. The property is not scriptable: set it in Studio's Properties window.

### `GetPropertyChangedSignal, Changed`

```luau
Object:GetPropertyChangedSignal(property: string) -> RBXScriptSignal
Object.Changed(property: string)  -- event
```

`GetPropertyChangedSignal` fires when one property changes and passes no arguments, so read the new value yourself. `Changed` fires for any property and passes its name. The docs prefer the first to filtering names inside `Changed`.

> Neither fires when physics moves a part: `CFrame`, `Position`, `Orientation` and the assembly velocities change silently. A property that changes very often may not fire every time.

### `Changed on a value object`

```luau
IntValue.Changed(value: number)  -- event
```

On `IntValue`, `StringValue` and the other `ValueBase` classes, `Changed` fires only when `Value` changes, and it passes the new value, not a property name.

> To watch a value object's other properties, use `GetPropertyChangedSignal`.

### `Attribute signals`

```luau
Instance:GetAttributeChangedSignal(attribute: string) -> RBXScriptSignal
Instance.AttributeChanged(attribute: string)  -- event
```

The attribute versions of the property signals. `GetAttributeChangedSignal` fires for one attribute with no arguments; `AttributeChanged` fires for any attribute and passes its name.

### `ChildAdded, ChildRemoved, DescendantAdded`

```luau
Instance.ChildAdded(child: Instance)  -- event
Instance.ChildRemoved(child: Instance)  -- event
Instance.DescendantAdded(descendant: Instance)  -- event
```

`ChildAdded` and `ChildRemoved` fire after a direct child is added or removed. `DescendantAdded` fires at any depth. Parenting a whole subtree fires it once for each object in it.

### `Touched, TouchEnded`

```luau
BasePart.Touched(otherPart: BasePart)  -- event
BasePart.TouchEnded(otherPart: BasePart)  -- event
```

Fire on both parts when they start or stop touching, and only through physical movement. Setting `CFrame` so that parts overlap fires nothing, and at least one of the two parts must be unanchored.

> One contact fires `Touched` many times in quick succession: once for each touching part, over and over. A handler that must act once per contact needs a debounce, such as a per-part cooldown.

### `Tag signals`

```luau
CollectionService:GetInstanceAddedSignal(tag: string) -> RBXScriptSignal
CollectionService:GetInstanceRemovedSignal(tag: string) -> RBXScriptSignal
```

The added signal fires with the instance when the tag is added to something in the DataModel, or when a tagged instance enters it. The removed signal is the mirror image, which makes it the place to clean up. Every call with the same tag returns the same signal.

> Neither fires for instances that were already tagged. Loop over `GetTagged` first, then connect the added signal, and no instance is missed.

## Player, Character, Humanoid and Teams

A `Player` is the connection. Its `Character` is a Model that is rebuilt on every spawn, and the `Humanoid` inside the character holds health and movement.

### `PlayerAdded, PlayerRemoving`

```luau
Players.PlayerAdded(player: Player)  -- event
Players.PlayerRemoving(player: Player, reason: Enum.PlayerExitReason)  -- event
```

`PlayerAdded` fires when a player enters. `PlayerRemoving` fires just before a player leaves, before `ChildRemoved` fires on Players, so the player's data is still there to save or clear.

> In a solo playtest the player can exist before your script connects, so `PlayerAdded` never fires for them. Put the handler in a function and also call it for each player in `GetPlayers()`.

### `GetPlayers`

```luau
Players:GetPlayers() -> {Player}
```

Every player currently connected. Loop over it after connecting `PlayerAdded`, and no player is missed.

### `GetPlayerFromCharacter`

```luau
Players:GetPlayerFromCharacter(character: Model) -> Player?
```

The player whose `Character` is this model, or `nil` if it is not a player's character, such as an NPC.

> From a `Touched` part, find the model first with `FindFirstAncestorOfClass("Model")`. A hat's `Handle` is inside an `Accessory`, which is not a Model, so `otherPart.Parent` is not always the character.

### `User, Name, DisplayName`

```luau
Player.User: User
Player.Name: string
Player.DisplayName: string
```

`User` is the player's identity in this experience, and the docs make it the standard identifier in new code; `User.Id` is the number to key by. `Player.UserId` still works as the older numeric form. `DisplayName` is what other players see; it is not unique, and it can change.

> Key per-player tables and saved data by `player.User.Id` (docs: "Users and domain-scoped user IDs").

### `Character, CharacterAdded`

```luau
Player.Character: Model?
Player.CharacterAdded(character: Model)  -- event
```

`Character` is `nil` until the first spawn. `CharacterAdded` fires on every spawn and respawn. The docs say it fires before the character is parented to Workspace, and that the Humanoid and body parts already exist on the server at that point; accessories and clothing can take a few seconds more.

> Each respawn is a new Model, so connect `CharacterAdded` rather than reading `Character` once. The docs advise against watching `Character` with `GetPropertyChangedSignal`.

### `The character model`

```luau
character:WaitForChild("Humanoid")
character:WaitForChild("HumanoidRootPart")  -- also the model's PrimaryPart
```

A player's character is a Model with a `Humanoid`, a `HumanoidRootPart` and the body parts. On an R15 rig these include `UpperTorso`. Character models have `PrimaryPart` set to the HumanoidRootPart.

### `RespawnLocation`

```luau
Player.RespawnLocation: SpawnLocation?
```

When set, the player respawns at this SpawnLocation. It must be a descendant of Workspace, and either `Neutral` or matching the player's `TeamColor`.

> The docs list two alternatives: change the player's `Team`, or move the character yourself with `PivotTo`.

### `SpawnLocation, ForceField`

```luau
SpawnLocation.Duration: number  -- default 10
SpawnLocation.Neutral: boolean  -- default true
```

A SpawnLocation is a `Part` that players spawn on. `Duration` is how many seconds a character spawning there is protected by a ForceField; with `0`, no ForceField is created. A `Neutral` spawn serves every team.

> A ForceField only protects against `Humanoid:TakeDamage`, explosions, and code that checks for one itself. Setting `Health` directly ignores it.

### `LoadCharacterAsync, CharacterAutoLoads, RespawnTime`

```luau
Player:LoadCharacterAsync()  -- yields
Players.CharacterAutoLoads: boolean  -- default true
Players.RespawnTime: number  -- default 5
```

`LoadCharacterAsync` builds a new character and removes the old one. It also clears the player's Backpack and PlayerGui. With `CharacterAutoLoads` on, characters respawn by themselves `RespawnTime` seconds after death; with it off, only `LoadCharacterAsync` spawns them.

> It replaced the deprecated `LoadCharacter`. The docs advise not calling it again for the same player until `CharacterAppearanceLoaded` has fired.

### `Health, MaxHealth`

```luau
Humanoid.Health: number
Humanoid.MaxHealth: number  -- default 100
```

The docs say `Health` is restricted to the range 0 to `MaxHealth`. At 0 the humanoid dies and `Health` stays locked at 0. By default a player character regenerates 1% of `MaxHealth` each second.

> Setting `Health` directly bypasses any ForceField. That is how lava kills a player who has just spawned.

### `TakeDamage`

```luau
Humanoid:TakeDamage(amount: number)
```

Lowers `Health` by `amount` unless the humanoid is protected by a ForceField. It is protected when the ForceField is the humanoid's sibling, is on its `RootPart`, or is under an ancestor other than Workspace. A negative amount heals, again only without a ForceField.

> The spawn ForceField (10 seconds by default) makes `TakeDamage` do nothing to a player who has just spawned. That is the point of the ForceField, and the reason to use `TakeDamage` for ordinary damage.

### `Died`

```luau
Humanoid.Died()  -- event
```

Fires when the humanoid dies, usually because `Health` reached 0. It fires only while the humanoid is a descendant of Workspace, and not at all if the `Dead` state has been disabled.

### `WalkSpeed, JumpPower, JumpHeight`

```luau
Humanoid.WalkSpeed: number  -- default 16
Humanoid.UseJumpPower: boolean
Humanoid.JumpPower: number  -- default 50
Humanoid.JumpHeight: number  -- default 7.2
```

`WalkSpeed` is in studs per second. `UseJumpPower` picks the jump property: when `true`, `JumpPower` sets the upward force; when `false`, `JumpHeight` sets the height in studs. Each spawn copies `StarterPlayer.CharacterUseJumpPower`, which defaults to `true`.

> Neither jump property is deprecated. `WalkSpeed` can be changed on the client, so the docs warn against using it as a security measure.

### `Teams`

```luau
game:GetService("Teams")
Team.TeamColor: BrickColor
Team.AutoAssignable: boolean  -- default true
Player.Team: Team?
```

`Team` objects must be children of the Teams service. A joining player goes onto the auto-assignable team with the fewest players, or becomes `Neutral` if there is none. Setting `Player.Team` moves a player.

> Auto-assignment evens teams only as players join; nothing rebalances when they leave. Much of the built-in team behaviour, spawns included, keys on `TeamColor`, so the docs recommend a unique colour per team.

### `leaderstats`

```luau
local leaderstats = Instance.new("Folder")
leaderstats.Name = "leaderstats"
leaderstats.Parent = player
```

A Folder named `leaderstats` under a Player drives the in-game leaderboard. Each `IntValue`, `NumberValue` or `StringValue` inside it is a column named after the value's `Name`, and you update it through `.Value`.

> The name must be exactly `leaderstats`, all lowercase, or the player is not added to the leaderboard.

### `PlayerCharacterDestroyBehavior`

```luau
Workspace.PlayerCharacterDestroyBehavior: Enum.PlayerCharacterDestroyBehavior  -- Default | Disabled | Enabled
```

With `Enabled`, the engine calls `Destroy` on a player's old character when it is replaced and on the Player when they leave. `Disabled` turns that off. It is not scriptable; set it in Studio.

> The docs say only that `Default` uses "the engine-default behavior". Write cleanup that works under every setting: disconnect your own per-player connections, and clear per-player tables on `PlayerRemoving`.

## Vector3

A point, a direction or a size in 3D. The same type means "where" or "which way and how far", depending on how you use it.

### `Vector3.new`

```luau
Vector3.new(x: number?, y: number?, z: number?) -> Vector3
```

A vector from three components, each defaulting to 0.

### `Constants`

```luau
Vector3.zero, Vector3.one
Vector3.xAxis, Vector3.yAxis, Vector3.zAxis
```

`(0, 0, 0)`, `(1, 1, 1)`, and 1 on a single axis. `Vector3.yAxis` is world up.

### `X, Y, Z`

```luau
Vector3.X: number
Vector3.Y: number
Vector3.Z: number
```

The components.

### `Arithmetic`

```luau
a + b, a - b, a * b, a / b  -- component by component
v * n, n * v, v / n
```

Each operation works component by component, with another Vector3 or a number on either side. `target - origin` is the offset from origin to target: which way, and how far.

### `Magnitude`

```luau
Vector3.Magnitude: number
```

The length, `math.sqrt(X^2 + Y^2 + Z^2)`. `(a - b).Magnitude` is the distance between two points.

### `Unit`

```luau
Vector3.Unit: Vector3
```

A copy with the same direction and a magnitude of 1: the direction without the distance.

> The zero vector has no direction, so its `Unit` has NaN components. Check `Magnitude` first whenever the vector can be zero.

### `Dot`

```luau
Vector3:Dot(other: Vector3) -> number
```

`a.X*b.X + a.Y*b.Y + a.Z*b.Z`. For unit vectors it is 1 when they point the same way, 0 when perpendicular and -1 when opposite, so `look:Dot((target - origin).Unit) > 0` means the target is in front.

### `Lerp`

```luau
Vector3:Lerp(goal: Vector3, alpha: number) -> Vector3
```

The point `alpha` of the way from this vector to `goal`, in a straight line. `alpha` is not limited to 0 to 1.

## CFrame

A coordinate frame: a position plus an orientation. Angles are in radians, and multiplication order matters.

### `CFrame.new`

```luau
CFrame.new(pos: Vector3) -> CFrame
CFrame.new(x: number, y: number, z: number) -> CFrame
CFrame.new() -> CFrame  -- same as CFrame.identity
```

A CFrame at a position with no rotation: its axes line up with the world's.

### `CFrame.lookAt`

```luau
CFrame.lookAt(at: Vector3, lookAt: Vector3, up: Vector3?) -> CFrame
```

A CFrame at `at`, facing `lookAt`. `up` defaults to `(0, 1, 0)`.

> It replaced the two-vector `CFrame.new(pos, lookAt)`, which the docs keep only for backward compatibility.

### `CFrame.Angles`

```luau
CFrame.Angles(rx: number, ry: number, rz: number) -> CFrame
```

A rotation from angles in radians, applied in X, Y, Z order. It is the same as `CFrame.fromEulerAnglesXYZ`, which the docs define as the rotation about X, times the rotation about Y, times the rotation about Z.

> It takes radians. `CFrame.Angles(0, 90, 0)` turns by 90 radians, about 117° once the full turns are taken out; write `math.rad(90)`.

### `Position and direction vectors`

```luau
CFrame.Position: Vector3
CFrame.LookVector: Vector3
CFrame.RightVector: Vector3
CFrame.UpVector: Vector3
```

The position, and the unit vectors pointing forward, right and up in the frame's own orientation. `LookVector` is the negated Z axis. `cf + cf.LookVector` is `cf` moved one stud forward.

### `Operators`

```luau
cf * offset -> CFrame  -- offset in cf's own space
cf * v -> Vector3  -- point from object to world space
cf + v, cf - v -> CFrame  -- moved in world space
```

`cf * offset` applies `offset` relative to `cf`, so `part.CFrame * CFrame.new(0, 0, -5)` is 5 studs in front of the part whichever way it faces. `cf * v` turns a point in the frame's space into a world point. `cf + v` moves the position in world space and keeps the orientation.

> `a * b` applies `b` in `a`'s space. Swapping the operands gives a different answer whenever `a` is rotated.

### `ToWorldSpace, ToObjectSpace`

```luau
CFrame:ToWorldSpace(...: CFrame) -> ...CFrame
CFrame:ToObjectSpace(...: CFrame) -> ...CFrame
```

Convert CFrames between this frame's space and world space. `ToWorldSpace(c)` is `cf * c`; `ToObjectSpace(c)` is `cf:Inverse() * c`, which is where `c` sits relative to `cf`.

### `PointToWorldSpace, PointToObjectSpace`

```luau
CFrame:PointToWorldSpace(...: Vector3) -> ...Vector3
CFrame:PointToObjectSpace(...: Vector3) -> ...Vector3
```

The point versions. `PointToWorldSpace(v)` is `cf * v`; `PointToObjectSpace(v)` is `cf:Inverse() * v`. A negative Z in object space means the point is in front.

### `Inverse`

```luau
CFrame:Inverse() -> CFrame
```

The transformation that undoes this one, reversing both rotation and translation. `cf * cf:Inverse()` is the identity.

### `BasePart.CFrame and Position`

```luau
BasePart.CFrame: CFrame
BasePart.Position: Vector3
```

Setting `CFrame` places the part exactly, orientation included, with no overlap checking, and parts joined to it move too. Setting `Position` moves only that part: welds to it recalculate their offset instead of dragging the others along.

> To move a whole model, use `PivotTo` (see Model and pivots).

## RunService and task

Each frame fires `PreSimulation`, runs physics, then fires `PostSimulation` and `Heartbeat`. The task library schedules code against the same clock.

### `PreSimulation`

```luau
RunService.PreSimulation(deltaTimeSim: number)  -- event
```

Fires every frame before the physics simulation, with the time this frame will step physics. It is the place to adjust velocities or forces before they apply.

> It replaced `Stepped`.

### `PostSimulation`

```luau
RunService.PostSimulation(deltaTimeSim: number)  -- event
```

Fires every frame after physics, for final adjustments to its result. `Heartbeat` follows it.

### `Heartbeat`

```luau
RunService.Heartbeat(deltaTime: number)  -- event
```

Fires every frame at the end of the frame, after physics, with the time since the previous frame. Threads waiting on the task library resume here.

> Scale motion by `deltaTime`, as in `angle += speed * dt`, so it moves at the same speed at 30 and at 60 frames a second.

### `PreRender`

```luau
RunService.PreRender(deltaTimeRender: number)  -- event, client only
```

Fires every frame before rendering. It exists only on the client (in a LocalScript, or a Script with a `Client` run context), so a server Script cannot use it.

> It replaced `RenderStepped`.

### `task.wait`

```luau
task.wait(duration: number?) -> number
```

Yields for at least `duration` seconds (default 0), resumes on the next Heartbeat after that, and returns the time that actually passed.

> Unlike the deprecated global `wait()`, it does not throttle: it resumes on the first Heartbeat after the time is up.

### `task.spawn`

```luau
task.spawn(functionOrThread: (...any) -> () | thread, ...: any) -> thread
```

Runs the function (or resumes the thread) immediately through the engine's scheduler, passing the extra arguments. Returns the thread.

> Use it for a loop that must run alongside the rest of the script, such as one loop per tagged part.

### `task.defer`

```luau
task.defer(functionOrThread: (...any) -> () | thread, ...: any) -> thread
```

Like `task.spawn`, but deferred until the end of the current resumption point, for when the work need not run immediately.

### `task.delay`

```luau
task.delay(duration: number, functionOrThread: (...any) -> () | thread, ...: any) -> thread
```

Runs the function on the first Heartbeat after `duration` seconds, without yielding the caller. A duration of 0 means the very next Heartbeat.

### `task.cancel`

```luau
task.cancel(thread: thread)
```

Cancels a thread returned by `task.spawn`, `task.defer` or `task.delay` so it is never resumed.

> Some threads cannot be cancelled, such as the one currently running, and the call errors for them.

### `Debris:AddItem`

```luau
Debris:AddItem(item: Instance, lifetime: number?)  -- default 10
```

Destroys `item` after `lifetime` seconds without yielding. It runs outside the calling script, so the item is destroyed even if that script is disabled or destroyed first. A `task.delay` callback would not run in that case.

> Debris holds at most 1,000 items. Past that, the oldest is destroyed at once, so treat `lifetime` as a maximum.

## Model and pivots

Models and parts are both `PVInstance`s, and every PVInstance has a pivot. Move a model by its pivot.

### `PivotTo`

```luau
PVInstance:PivotTo(targetCFrame: CFrame)
```

Moves the instance and every PVInstance inside it so that its pivot lands on `targetCFrame`. The docs name it the primary way to move a model from a script, and parts have it too.

> Models cache their parts' offsets, so repeated `PivotTo` calls do not build up floating-point drift. Moved parts fire change signals for `CFrame`, not for `Position` or `Orientation`.

### `GetPivot`

```luau
PVInstance:GetPivot() -> CFrame
```

The pivot's current CFrame. `model:PivotTo(model:GetPivot() * CFrame.Angles(0, math.rad(90), 0))` turns a model a quarter turn in place.

### `PrimaryPart`

```luau
Model.PrimaryPart: BasePart?
```

The part the model's pivot follows. It is not set by default. A model moved by physics needs one, or its pivot stays where it was while the parts move. Character models use the HumanoidRootPart.

> It must be a descendant of the model; any other part is reset to `nil` on the next simulation step. Setting the primary part's `CFrame` moves only the parts joined to it, while `PivotTo` moves everything.

### `WorldPivot`

```luau
Model.WorldPivot: CFrame
```

The pivot of a model with no `PrimaryPart`, ignored when there is one. A new model's pivot is the centre of its bounding box until `WorldPivot` is first set, and `PivotTo` sets it.

### `GetBoundingBox`

```luau
Model:GetBoundingBox() -> (CFrame, Vector3)
```

The centre and size of a box containing the model's parts, oriented like the model's pivot. It matches the selection box Studio draws.

> The docs warn the size may be inaccurate at runtime while constraints are acting on the parts.

## TweenService

A tween moves properties of one instance from their current values to goal values over time, along an easing curve.

### `TweenInfo.new`

```luau
TweenInfo.new(
  time: number?,                           -- 1
  easingStyle: Enum.EasingStyle?,          -- Quad
  easingDirection: Enum.EasingDirection?,  -- Out
  repeatCount: number?,                    -- 0
  reverses: boolean?,                      -- false
  delayTime: number?                       -- 0
) -> TweenInfo
```

How a tween runs: its duration in seconds, its curve, how many extra times it repeats (`-1` repeats forever), whether each run plays back in reverse, and the delay before it starts. A TweenInfo cannot be changed after it is created.

### `EasingStyle, EasingDirection`

```luau
Enum.EasingStyle: Linear | Sine | Back | Quad | Quart | Quint | Bounce | Elastic | Exponential | Circular | Cubic
Enum.EasingDirection: In | Out | InOut
```

The shape of the curve, and which end of it the easing applies to.

### `TweenService:Create`

```luau
TweenService:Create(instance: Instance, tweenInfo: TweenInfo, propertyTable: {[string]: any}) -> Tween
```

A tween for one instance, whose `propertyTable` maps property names to goal values. It can tween numbers, booleans, `CFrame`, `Rect`, `Color3`, `UDim`, `UDim2`, `Vector2`, `Vector2int16`, `Vector3` and EnumItems. Creating it does not start it.

> A tween belongs to its instance. Tweening another object means calling `Create` again.

### `Play, Pause, Cancel`

```luau
Tween:Play()
Tween:Pause()
Tween:Cancel()
```

`Play` starts the tween, and does nothing if it is already playing. `Pause` stops it and keeps its progress. `Cancel` stops it and resets its progress, but not the property, which stays where it was; a later `Play` takes the full duration.

> Two tweens may run on one object, but not on the same property: the newer one cancels the older.

### `Completed`

```luau
Tween.Completed(playbackState: Enum.PlaybackState)  -- event
```

Fires when the tween finishes or is cancelled, and passes its `PlaybackState` so you can tell which. Pausing does not fire it. `tween.Completed:Wait()` yields until the tween ends.

## Physics properties, collision groups and constraints

Anchored parts stay put. Everything else is simulated as assemblies, and constraints move assemblies the way physics expects.

### `Anchored`

```luau
BasePart.Anchored: boolean
```

An anchored part never moves because of physics: not from gravity, collisions or overlaps. A script can still move it by setting `CFrame` or `Position`. An unanchored part welded to an anchored one acts anchored too.

> Two anchored parts never fire `Touched` on each other.

### `CanCollide, CanTouch, CanQuery`

```luau
BasePart.CanCollide: boolean
BasePart.CanTouch: boolean
BasePart.CanQuery: boolean
```

`CanCollide` decides whether other parts pass through; touch events still fire when it is off. `CanTouch` decides whether `Touched` and `TouchEnded` fire, and both parts need it on. `CanQuery` decides whether spatial queries and raycasts see the part, and it takes effect only when `CanCollide` is off.

> Connecting `Touched` on a part whose `CanTouch` is off throws an error, and turning it off later disconnects the event. An unanchored part with `CanCollide` off can fall out of the world.

### `Massless`

```luau
BasePart.Massless: boolean
```

The part adds no mass or inertia to its assembly, as long as it is welded to a part that has mass. It is ignored for the assembly's root part.

### `WeldConstraint`

```luau
WeldConstraint.Part0: BasePart?
WeldConstraint.Part1: BasePart?
WeldConstraint.Enabled: boolean
```

Once both parts are set and it is enabled, the weld locks their relative position and orientation; they need not touch. Parts joined this way form one assembly, which physics moves as a single rigid body.

> Setting a welded part's `CFrame` carries the parts welded to it along. Setting its `Position` moves only that part, and the weld keeps the new offset.

### `Collision groups`

```luau
workspace:RegisterCollisionGroup(name: string)
workspace:CollisionGroupSetCollidable(name1: string, name2: string, collidable: boolean)
workspace:IsCollisionGroupRegistered(name: string) -> boolean
BasePart.CollisionGroup: string  -- default "Default"
```

Register a group by name (anything except `"Default"`), set whether two groups collide, and put a part in a group by setting its `CollisionGroup` to the name. A group set not to collide with itself stops its own parts colliding with each other.

> `CollisionGroupSetCollidable` errors if either group is unregistered. Registering costs more the more parts the world holds, so the docs recommend registering groups at edit time in Studio's Collision Groups editor. These are `WorldRoot` methods; the docs have deprecated the PhysicsService versions.

### `Attachment`

```luau
Attachment.CFrame: CFrame  -- offset from the parent part
Constraint.Attachment0: Attachment?
Constraint.Attachment1: Attachment?
```

A point and orientation relative to the part it sits in. Most constraints connect two attachments rather than two parts, so the attachments' positions and axes define the joint. `WeldConstraint` is the exception: it takes the parts directly.

### `HingeConstraint`

```luau
HingeConstraint.ActuatorType: Enum.ActuatorType  -- None | Motor | Servo
HingeConstraint.AngularVelocity: number  -- radians per second
HingeConstraint.MotorMaxTorque: number
```

Rotation about one axis: the two attachments share a position and their X axes point the same way. As a `Motor` it drives towards `AngularVelocity`, limited by `MotorMaxTorque` and `MotorMaxAcceleration`. As a `Servo` it turns to `TargetAngle` at `AngularSpeed`.

> `MotorMaxTorque` defaults to 0 in the API dump, and it caps the torque the motor may apply. Raise it, or the motor cannot turn anything.

### `PrismaticConstraint`

```luau
PrismaticConstraint.ActuatorType: Enum.ActuatorType
PrismaticConstraint.TargetPosition: number  -- studs
PrismaticConstraint.Speed: number  -- studs per second
PrismaticConstraint.ServoMaxForce: number
```

Sliding along one axis with no rotation: the attachments' X axes are collinear and point in opposite directions. As a `Servo` it moves the attachments to the `TargetPosition` separation at `Speed`, limited by `ServoMaxForce`.

> These properties are inherited from `SlidingBallConstraint`, which is where the docs describe them.

### `AlignPosition, LinearVelocity, AngularVelocity`

```luau
AlignPosition.Mode: Enum.PositionAlignmentMode  -- TwoAttachment | OneAttachment
LinearVelocity.VectorVelocity: Vector3
AngularVelocity.AngularVelocity: Vector3
```

The mover constraints. `AlignPosition` pulls `Attachment0` to `Attachment1`, or to its `Position` in `OneAttachment` mode. `LinearVelocity` applies force to hold a constant velocity, and `AngularVelocity` applies torque to hold a constant spin.

> They replace the deprecated body movers. `AngularVelocity.MaxTorque` defaults to 0 in the API dump, the same trap as a hinge motor.

## ProximityPrompt and ClickDetector

Two built-in ways for a player to interact with something in the world. Both hand the server the player who did it.

### `ProximityPrompt`

```luau
Instance.new("ProximityPrompt")  -- parent to a BasePart, Attachment or Model
```

Shows an on-screen prompt when a player's character comes near. It works when parented to a BasePart, an Attachment, or a Model with `PrimaryPart` set, in Workspace.

### `Triggered`

```luau
ProximityPrompt.Triggered(playerWhoTriggered: Player)  -- event
```

Fires when the player presses the prompt's key or button, or after holding it for `HoldDuration` seconds when that is set. A server handler receives the player.

### `Prompt properties`

```luau
ProximityPrompt.ActionText: string  -- default "Interact"
ProximityPrompt.ObjectText: string
ProximityPrompt.HoldDuration: number  -- default 0
ProximityPrompt.MaxActivationDistance: number  -- default 10
ProximityPrompt.KeyboardKeyCode: Enum.KeyCode  -- default E
```

What the prompt says, how long it must be held, how close the character must be, and which key triggers it.

> `RequiresLineOfSight` defaults to `true`, so a prompt blocked by a wall does not show.

### `ClickDetector`

```luau
ClickDetector.MouseClick(playerWhoClicked: Player)  -- event
ClickDetector.MaxActivationDistance: number  -- default 32
```

Pointer input on a 3D object. It works when parented to a BasePart, a Model or a Folder. Touch taps fire it too, and its events fire on both server and client.

> When several detectors could take a click, only the deepest fires; between siblings, the first wins.

## Ecosystem

| Package | Role | Version | Install |
| --- | --- | --- | --- |
| [Roblox Studio](https://create.roblox.com/docs/studio/setup) | The engine's editor, where places are built, run and tested | 0.740.19.7400931 | `Download from create.roblox.com/docs/studio/setup` |
| [Engine API reference](https://create.roblox.com/docs/reference/engine) | The behaviour of every class, datatype, enum and library | 0.740.19.7400931 | `Nothing to install: read it at create.roblox.com/docs/reference/engine` |
| [Roblox-Client-Tracker](https://github.com/MaximumADHD/Roblox-Client-Tracker) | The API dump: every member, its signature, default and deprecation tag | 0.740.19.7400931 | `Download API-Dump.json or Full-API-Dump.json from the repository's roblox branch` |

## Out of date elsewhere

Shapes you may still meet in older tutorials:

- **`player.UserId` in new code.** It still works, but since Roblox introduced domain-scoped user IDs the docs name `Player.User` the standard identifier in new code, and key data stores within one game by `player.User.Id`. A player who first joins after the change gets an id scoped to your game.
- **A Script left in the `Legacy` run context.** It runs only inside a server container such as `Workspace` or `ServerScriptService`. The docs' "Script locations" guide recommends one Script with an explicit `RunContext` of `Server` in ServerScriptService, one with `Client` in ReplicatedStorage, and ModuleScripts for the rest.
- **`wait()`, `spawn()` and `delay()`.** Deprecated globals that throttle. The docs mark each one superseded by the `task` library: `task.wait(1)`, `task.spawn(fn)` and `task.delay(2, fn)`, which do not throttle.
- **`Instance.new("Part", workspace)`.** The docs call the parent argument discouraged. Set the properties, then write `part.Parent = workspace` last, so the part replicates once.
- **`:connect()` and other camelCase members.** Old code writes `part.Touched:connect(fn)` or `model:clone()`. The docs say camelCase APIs are mostly deprecated (the dump marks `clone`, `children` and `Player.userId`), so write `:Connect`, `:Clone`, `:GetChildren` and `UserId`. `:remove()` and `:Remove()` are deprecated too: use `:Destroy()`, or set `Parent = nil` to keep the object for later.
- **`BodyVelocity`, `BodyPosition` and `BodyGyro`.** Every body mover is deprecated. Use the constraint movers: `LinearVelocity`, `AlignPosition` and `AlignOrientation`. Likewise `BodyAngularVelocity` becomes `AngularVelocity`, `BodyForce` and `BodyThrust` become `VectorForce`, and `RocketPropulsion` becomes `LineForce`.
- **Collision groups through `PhysicsService`.** `CreateCollisionGroup`, `SetPartCollisionGroup` and `BasePart.CollisionGroupId` are deprecated in the dump. `PhysicsService:RegisterCollisionGroup` and `CollisionGroupSetCollidable` are not tagged yet, but the docs deprecate the whole service in favour of per-world groups and forward its calls to Workspace. Write `workspace:RegisterCollisionGroup("Carts")`, `workspace:CollisionGroupSetCollidable("Carts", "Players", false)` and `part.CollisionGroup = "Carts"`.
- **`model:SetPrimaryPartCFrame(cf)` and `model:GetPrimaryPartCFrame()`.** Both are deprecated. `model:PivotTo(cf)` and `model:GetPivot()` replace them; the docs call `PivotTo` more performant, with no change in behaviour.
- **`player:LoadCharacter()`.** Deprecated and superseded by `player:LoadCharacterAsync()`, which loads the character the same way.
- **Anything about `FilteringEnabled`.** `Workspace.FilteringEnabled` is deprecated, and the docs say it no longer takes effect. Advice to turn it on or off is obsolete.
- **`RunService.Stepped` and `RenderStepped`.** The dump does not deprecate them, but the docs say `PreSimulation` and `PreRender` supersede them for new work. `Stepped` passes `(time, deltaTime)`; `PreSimulation` passes only the frame's delta.
- **`CFrame.new(pos, lookAt)`.** The docs say `CFrame.lookAt(pos, lookAt)` replaced this overload, which remains for backward compatibility. `lookAt` also takes an optional up vector.
- **`CollectionService:AddTag(part, "Lava")`, `HasTag` and `RemoveTag`.** Not deprecated, and some guides still use them. This course writes the Instance methods, `part:AddTag("Lava")`, `part:HasTag(…)` and `part:RemoveTag(…)`, which fire the same `CollectionService` signals. `GetTagged` and the added and removed signals stay on `CollectionService`.
