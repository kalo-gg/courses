# Controlling a spring

*setPosition, setVelocity and addVelocity move a spring without moving the goal it is chasing.*

Fusion · Animation · 10 min

---

A spring has two numbers you can reach for directly, and they are not the goal. `setPosition` puts the spring somewhere immediately, and `setVelocity` gives it a push.

Neither changes the *goal*, which leads to a behavior worth seeing before it surprises you. Both are methods on the spring object itself rather than state you can bind, so they belong in event handlers and setup code, never inside a `Computed`.

**setPosition does not change where it is going**

```luau
local goal = scope:Value(50)
local spring = scope:Spring(goal, 30, 1)

step(60)
print(math.floor(peek(spring))) --> 50, settled

spring:setPosition(0)
print(math.floor(peek(spring))) --> still 50: the move lands on the next step

step(1)
print(math.floor(peek(spring))) --> 4, and climbing

step(60)
print(math.floor(peek(spring))) --> 50, back where the goal is
```

Teleporting a spring is temporary unless you move the goal as well. The goal is what it wants; the position is only where it is.

## Three ways to push it

| Method | What it changes | Reach for it when |
| --- | --- | --- |
| `spring:setPosition(v)` | Where the value is now; velocity untouched | Starting an entrance off-screen, or snapping after a teleport |
| `spring:setVelocity(v)` | The current velocity, replacing whatever it was | Handing the spring a known speed, such as a flick gesture's |
| `spring:addVelocity(v)` | Adds to the current velocity | A nudge on top of motion already happening: a hit, a click kick |

None of the three touches the goal, and that is the whole idea worth taking away. The goal is the only thing that says where the spring wants to be; position and velocity are the state of the journey. Change a goal and the spring re-plans. Change a position and it simply finds itself somewhere else, still heading to the same place.

**An entrance, in four lines**

```luau
local open = scope:Value(false)

local goal = scope:Computed(function(use)
	return if use(open) then 0 else 40
end)
local offset = scope:Spring(goal, 25, 1)

offset:setPosition(-300) -- it comes from off-screen
open:set(true)           -- and the goal says where it belongs

print(math.floor(peek(offset))) --> 40: the move lands on the next step, not this line
step(1)
print(math.floor(peek(offset))) --> -278: travelling in from off-screen
step(60)
print(math.floor(peek(offset))) --> 0: the goal it was heading for all along
```

The goal never mentions the entrance. One call says where the value *is*, and the spring closes the gap exactly as it would for any other movement. Two details: place the spring **before** you change the goal — a `setPosition` made while a goal change is still waiting for the next update step is overwritten when the spring reconfigures itself — and note that the goal here is a state object, a `Computed`, which matters, because a spring built on a plain constant goal never gets a stopwatch and calling `setPosition` on one raises `springTypeMismatch`.

> **The entrance pattern**
>
> The goal says where the element belongs; `setPosition` says where it comes from — and the `setPosition` goes first, so a goal change still waiting for the next update step cannot overwrite it. The spring does the rest, and no code has to know the difference between an entrance and any other movement. The same trick in reverse — goal off-screen, then leave it — is an exit, though a `Visible` binding has to wait for it (see Showing and hiding).

**Key points**

- `setPosition`, `setVelocity` and `addVelocity` move a spring; none of them moves its goal.
- `addVelocity` stacks on the motion already happening; `setVelocity` replaces it.
- The change lands on the next update step, not on the line that made it.
- Goal where it belongs, `setPosition` where it comes from: that is an entrance.
- A spring bound to a numeric property is the cheapest way to make a screen feel alive.
