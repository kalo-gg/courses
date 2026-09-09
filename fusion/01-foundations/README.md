# 1. Fusion Foundations

*What Fusion does for you, how to install 0.3, and the scope that owns everything you create.*

Fusion · Easy

---

Fusion is a Luau companion library for describing interfaces instead of updating them by hand. This module covers the idea, installs Fusion 0.3 and teaches you to tell it apart from the 0.2 tutorials still online, then spends most of its time on scopes: the array of destructors that owns every object you make, cleans up in reverse, and decides whether your UI leaks. It ends with your first interface: a counter that builds and destroys itself.

## Lessons

1. [What is Fusion?](01-what-is-fusion.md) — Describe the interface for a given state and let Fusion keep every instance in sync.
2. [Installing Fusion 0.3](02-installing-fusion.md) — Get the release into ReplicatedStorage, then learn to tell 0.3 apart from the tutorials that are not.
3. [Scopes own what you create](03-scopes.md) — A scope is an array of things to destroy. Everything you make goes in one, and it is cleaned up in reverse.
4. [Your first interface](04-your-first-interface.md) — New, Children and OnEvent: a counter button that builds itself and cleans itself up.
