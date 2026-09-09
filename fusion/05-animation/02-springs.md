# Springs

*Motion with weight: a speed, a damping ratio, and a value that can overshoot its goal.*

Fusion · Animation · 11 min

---

A tween takes a fixed time to travel a fixed path. A spring does not: it accelerates towards its goal and settles, and if you move the goal mid-flight it keeps its momentum.

That is what makes spring motion feel physical where a tween feels scheduled. The two numbers you hand it shape that motion: `speed` is the spring's angular frequency, scaling how fast time passes for the simulation, so doubling it halves how long the same journey takes, while `damping` decides how the arrival is shaped.

**Speed and damping**

```luau
local Fusion = require(game:GetService("ReplicatedStorage").Fusion)
local scope = Fusion.scoped(Fusion)
local peek = Fusion.peek

local goal = scope:Value(0)
local size = scope:Spring(goal, 30, 1)
--                              ^   ^
--                          speed   damping

goal:set(50)
step(60)
print(math.floor(peek(size))) --> 50
```

Damping of 1 is critically damped: it arrives without overshooting. Below 1 it bounces past and comes back; above 1 it crawls in.

| Damping | Behavior | Good for |
| --- | --- | --- |
| `< 1` | Overshoots and oscillates back | Playful motion, pop-in effects |
| `= 1` | Arrives as fast as possible without overshooting | Most UI: the safe default |
| `> 1` | Slows into place, never overshoots | Heavy or deliberate movement |

## Speed and damping are state too

Both numbers are `UsedAs` too, so a spring's speed can itself be a state object — a slider that changes how snappy the interface feels is a spring whose speed reads a `Value`.

**A snappiness slider, mid-flight**

```luau
local snappiness = scope:Value(5)
local goal = scope:Value(0)

local x = scope:Spring(goal, snappiness, 1)

goal:set(100)
step(30)
print(math.floor(peek(x))) -- part-way there at speed 5

snappiness:set(40)         -- the player drags the slider mid-flight
step(30)
print(math.floor(peek(x))) -- the rest of the distance, faster
```

A spring re-reads its goal, its speed and its damping on every update step. Change the speed while it is travelling and it keeps the position and velocity it already had — it just closes the remaining distance differently. Nothing restarts.

> **Nothing checks these two numbers**
>
> Fusion 0.3 *declares* messages for bad spring parameters — `invalidSpringSpeed`, `invalidSpringDamping`, `mistypedSpringSpeed` and `mistypedSpringDamping` are all in `Logging/messages.luau` — and never uses one of them. `Spring.luau` reads `peek(self._speed)` and `peek(self._damping)` straight into the coefficient maths with no check at all. So a negative damping ratio diverges instead of settling, quietly and forever; a speed of zero simply never moves; and a value that is not a number is a raw Luau arithmetic error thrown from inside Fusion, with no Fusion message attached to tell you where it came from. If a spring is misbehaving, check the two numbers you gave it first, because nothing else will.

**Key points**

- `scope:Spring(goal, speed, damping)` is a state object that accelerates towards its goal.
- Damping below 1 overshoots, 1 arrives cleanly, above 1 eases in; speed sets how fast the whole shape plays.
- Speed and damping are `UsedAs`, so either can be a `Value` the player controls, and changing one mid-flight keeps the motion going.
- A spring keeps its momentum when the goal moves mid-flight; a tween restarts.
