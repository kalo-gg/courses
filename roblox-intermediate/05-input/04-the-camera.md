# The camera

*Each client has its own camera: take it over with a Scriptable CameraType, and turn a point on the screen into a ray in the world.*

Roblox Intermediate · Input · 10 min

---

A building game needs to know which part the player is pointing at, and its lobby should show the arena from above until the round starts. Both jobs are about the camera: where it is, and what lies under a point on the screen.

The camera belongs to the client. Each player has their own, it never replicates, and the server cannot see where anyone is looking.

## Taking over the camera

**Client script**

```luau
local camera = workspace.CurrentCamera

camera.CameraType = Enum.CameraType.Scriptable
camera.CFrame = CFrame.lookAt(Vector3.new(0, 80, 80), Vector3.zero)

-- later, hand it back to the default camera scripts
camera.CameraType = Enum.CameraType.Custom
```

Roblox's default camera scripts set `CFrame` every frame. `Scriptable` tells them to leave the camera alone, and `Custom` hands it back.

`workspace.CurrentCamera` is the camera this client looks through; use the property rather than searching Workspace for a child named Camera.

A point on the screen stands for a line into the world. The camera gives you that line as a `Ray`: an `Origin` at the camera and a `Direction` one stud long. Multiply the direction by a range before you raycast along it, which the World module covers.

## From the mouse to a ray

```luau
local UserInputService = game:GetService("UserInputService")
local camera = workspace.CurrentCamera

local function mouseRay(): Ray
	local position = UserInputService:GetMouseLocation()
	return camera:ViewportPointToRay(position.X, position.Y)
end

local ray = mouseRay()
print(ray.Origin, ray.Direction) -- Direction is a unit vector
```

`GetMouseLocation` measures from the very top-left of the window, top bar included, which is what `ViewportPointToRay` expects.

Roblox has two kinds of screen position. *Viewport* positions start at the top-left of the whole window. *GUI* positions start below the top bar, and the gap between them is the GUI inset, which `GuiService:GetGuiInset()` reports. Each camera method expects one kind, so pair them:

| Position from | Measured from | Ray with |
| --- | --- | --- |
| `UserInputService:GetMouseLocation()` | The window's top-left (viewport) | `camera:ViewportPointToRay(x, y)` |
| `input.Position` in an input event | Below the top bar (GUI) | `camera:ScreenPointToRay(x, y)` |

> **Recognising older code**
>
> Older scripts aim with `local mouse = player:GetMouse()` and read `mouse.Hit` or `mouse.Target`. The API does not deprecate them, but the docs call `Mouse` a legacy object that UserInputService supersedes. Use `GetMouseLocation`, a camera ray and `workspace:Raycast`, which also lets you filter what the ray can hit.

**Key points**

- `workspace.CurrentCamera` is this client's own camera; it never replicates.
- Set `CameraType` to `Scriptable` before setting `CFrame`, and back to `Custom` to return control.
- `GetMouseLocation` pairs with `ViewportPointToRay`; an input's `Position` pairs with `ScreenPointToRay`.
- Both return a unit ray: multiply `Direction` by a range before raycasting.
- `player:GetMouse()` and `mouse.Hit` are the older shape.
