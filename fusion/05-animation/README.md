# 5. Animation

*Tweens follow a curve, springs follow physics, and both are just state objects with a goal.*

Fusion · Medium

---

Fusion animates by wrapping a goal in a state object that reports where the value is now. Because that goal is UsedAs, it can be a constant, a Value or a Computed, so animation composes with everything from the earlier modules. This module covers tweens, springs and the damping that shapes them, feeding a property from a Computed goal so a hover or a selection animates itself, and taking a spring off its rails with setPosition and setVelocity.

## Lessons

1. [Tweens](01-tweens.md) — A goal, a TweenInfo, and a value that walks the curve between them one frame at a time.
2. [Springs](02-springs.md) — Motion with weight: a speed, a damping ratio, and a value that can overshoot its goal.
3. [Controlling a spring](03-controlling-springs.md) — setPosition, setVelocity and addVelocity move a spring without moving the goal it is chasing.
4. [Animated properties](04-animated-properties.md) — Give an animation a Computed goal and hover, selection and health all animate themselves.
