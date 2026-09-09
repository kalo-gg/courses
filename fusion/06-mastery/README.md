# 6. Shipping a Fusion App

*Components that read like Fusion's own, values shared without threading, lifetimes you can debug, and a shop that ships.*

Fusion · Hard

---

The last mile. A component is a function taking a scope and props, and merging them into the scope makes them read exactly like New. Contextual carries a theme or a player down the call stack without props drilling. Then what separates a demo from a shipped screen: reading Fusion's lifetime warnings and errors before they become bugs, and laying an app out as a state module, components and one owning scope, the plan the final project follows.

## Lessons

1. [Components](01-components.md) — A component is a function taking a scope and some props. Here is how to make yours read like Fusion's.
2. [Contextual](02-contextual.md) — Pass a theme or the local player down ten components without threading it through ten props.
3. [Reading Fusion's warnings](03-lifetimes.md) — possiblyOutlives, poisoned scopes and destroyedTwice: what Fusion tells you before it becomes a bug.
4. [Laying out an app](04-architecture.md) — A state module, a folder of components, one owning scope: the shape the final project builds.
