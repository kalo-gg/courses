# Tweens

*A goal, a TweenInfo, and a value that walks the curve between them one frame at a time.*

Fusion · Animation · 10 min

---

`Tween` takes a goal and a `TweenInfo` and gives you a state object. Read it and you get where the value is right now; change the goal and it starts moving there. In that `TweenInfo` the first argument is how long the move takes and the second is the easing style, the curve the value follows on the way there, which is `Linear` in the example below and so puts the value at half the distance after half the time.

Nothing else in your code has to know an animation is happening — the property was already reading a state object.

**Moving over one second**

```luau
local Fusion = require(game:GetService("ReplicatedStorage").Fusion)
local scope = Fusion.scoped(Fusion)
local peek = Fusion.peek

local goal = scope:Value(0)
local position = scope:Tween(goal, TweenInfo.new(1, Enum.EasingStyle.Linear))

goal:set(100)

print(peek(position)) --> 0
step(30)              -- half a second, at 60 frames per second
print(peek(position)) --> 50
step(30)
print(peek(position)) --> 100
```

`step(frames)` only exists in this sandbox. In a game, Fusion is driven by RunService and the value moves on its own.

## The goal can be anything

The goal is `UsedAs<T>`, the same type properties take. That means it can be a plain number, a `Value`, or a `Computed` worked out from other state — so an animation can chase something derived, without an intermediate variable.

**Animating towards derived state**

```luau
local base = scope:Value(10)
local doubled = scope:Computed(function(use)
	return use(base) * 2
end)

local following = scope:Tween(doubled, TweenInfo.new(0.5))

base:set(25) -- the computed becomes 50, and the tween sets off towards it
```

> **Why `step` exists here**
>
> Fusion asks its host to run an update step every frame — `RunService:BindToRenderStep` on the client in a real game, and `Heartbeat` on the server. The browser has no frame loop, so the sandbox hands you `step(frames)` instead. It makes animation lessons reproducible: the same `step(30)` gives the same number every time, which a real frame loop could not promise.

> **Number goals here, springs for the rest**
>
> In this sandbox a `Tween` animates numbers and `UDim`s. Over a `UDim2`, `Vector2` or `Color3` it errors, because the stand-in value types have no `Lerp` method — a limitation of the browser environment, not of Fusion, which animates every type in its `Animatable` list. Tween a number (`BackgroundTransparency`, `Rotation`, `TextSize`) or reach for a `Spring`, which works over all of them.

**Key points**

- `scope:Tween(goal, TweenInfo.new(seconds, style))` is a state object that walks to its goal along a curve.
- The goal is `UsedAs`: a constant, a `Value` or a `Computed`.
- The value only moves when the update step runs — `RunService` in a game, `step(frames)` here.
- Setting the goal does not set the value; it sets where the value is heading.
- In the browser sandbox, tween numbers; spring the rest.
