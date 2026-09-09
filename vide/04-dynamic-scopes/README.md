# 4. Dynamic Scopes

*Show, switch and list UI in response to data with show, switch, indexes and values.*

Vide · Medium

---

Menus open and close, tabs switch, lists grow and shrink. Each of those means creating UI when data appears and destroying it when data goes away. Vide's dynamic scope functions manage those lifetimes for you: they create a stable scope per item, destroy it when the item leaves, and hand you a source of instances you can drop straight into a property table as children.

## Lessons

1. [show and switch](01-show-and-switch.md) — Render a component while a condition holds, or pick one of several by a key.
2. [Lists with indexes](02-indexes.md) — One scope per table index: rows are reused when values change and destroyed when indexes vanish.
3. [Lists with values](03-values.md) — One scope per distinct value: rows follow their item when the list is reordered.
4. [Delayed destruction](04-delayed-destruction.md) — Return a delay from a dynamic component to play an exit animation before it is destroyed.
