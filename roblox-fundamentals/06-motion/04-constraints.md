# Constraints

*Attachments mark the joint, a constraint says how the parts may move, and an actuator lets physics drive them.*

Roblox Fundamentals · Motion · 11 min

---

The obby needs a bar that sweeps round a post and knocks players off. You could turn it from `Heartbeat`, but setting `CFrame` teleports the part a small step each frame, and nothing in physics is pushing it. The docs even say `Touched` does not fire when a CFrame change moves one part into another.

A **constraint** works with physics instead. The part stays simulated and the engine drives it with a torque or a force, so collisions and friction treat it like anything else that moves.

## Attachments and a hinge motor

**A bar that sweeps round a post**

```luau
local post = workspace.Post -- anchored
local bar = workspace.Bar -- unanchored, resting on top of the post

local top = Instance.new("Attachment")
top.CFrame = CFrame.new(0, post.Size.Y / 2, 0) * CFrame.Angles(0, 0, math.rad(90))
top.Parent = post

local axle = Instance.new("Attachment")
axle.CFrame = CFrame.new(0, -bar.Size.Y / 2, 0) * CFrame.Angles(0, 0, math.rad(90))
axle.Parent = bar

local hinge = Instance.new("HingeConstraint")
hinge.Attachment0 = top
hinge.Attachment1 = axle
hinge.ActuatorType = Enum.ActuatorType.Motor
hinge.AngularVelocity = math.rad(180) -- radians per second
hinge.MotorMaxTorque = 100_000
hinge.Parent = bar
```

A hinge turns about its attachments' **X** axes, so both are rotated to point X straight up. `MotorMaxTorque` starts at 0: leave it there and the motor has no strength at all.

Most constraints join two **attachments**, not two parts. An attachment is a point and an orientation inside its part, and the constraint reads both: a hinge holds its attachments at one point and turns about their X axes.

The actuated constraints share one property, `ActuatorType`. `None` leaves the joint free, `Motor` drives it at a speed, and `Servo` drives it to a target.

## A platform on a slider

**A platform that shuttles along a rail**

```luau
local rail = workspace.Rail -- anchored
local platform = workspace.MovingPlatform -- unanchored

local slider = Instance.new("PrismaticConstraint")
slider.Attachment0 = rail.Start -- attachments placed in Studio, X along the rail
slider.Attachment1 = platform.Slide
slider.ActuatorType = Enum.ActuatorType.Servo
slider.Speed = 8 -- studs per second
slider.ServoMaxForce = 100_000
slider.TargetPosition = 20 -- studs along the axis
slider.Parent = platform

while true do
	task.wait(3)
	slider.TargetPosition = if slider.TargetPosition == 0 then 20 else 0
end
```

A `PrismaticConstraint` slides along one axis without turning. As a servo it moves the attachments to `TargetPosition` studs apart at `Speed`, with at most `ServoMaxForce`. `Speed` and `ServoMaxForce` both start at 0. Flipping the target every three seconds is the obby's moving platform.

The mover constraints drive one part towards a goal instead of joining it to another: a position, a velocity or a spin.

| Constraint | Holds its part to | Set |
| --- | --- | --- |
| `AlignPosition` | A position: `Attachment1`, or `Position` in `OneAttachment` mode | `MaxForce`, `Responsiveness` |
| `LinearVelocity` | A constant velocity | `VectorVelocity`, `MaxForce` |
| `AngularVelocity` | A constant spin | `AngularVelocity` (a Vector3), and `MaxTorque`, which starts at 0 |

> **Recognising older code**
>
> Old scripts and free models push parts with `BodyVelocity`, `BodyPosition`, `BodyGyro` and `BodyAngularVelocity`. Every body mover is deprecated. Their replacements are `LinearVelocity`, `AlignPosition`, `AlignOrientation` and `AngularVelocity`.

> **Test the ride in Studio**
>
> A platform moved by a constraint is simulated: physics moves it, and friction can act on whatever stands on it. How well a character rides one, or a tweened one, is behaviour to try in Studio with a character before you build a level around it.

**Key points**

- Constraints move simulated parts with forces and torques; setting `CFrame` teleports a part instead.
- Most constraints join two attachments, and a hinge turns about their shared X axis.
- `ActuatorType`: `None` is free, `Motor` drives at a speed, `Servo` drives to a target.
- `MotorMaxTorque`, `ServoMaxForce` and `AngularVelocity.MaxTorque` start at 0: raise them or nothing moves.
- `LinearVelocity`, `AlignPosition` and `AngularVelocity` replace the deprecated body movers.
