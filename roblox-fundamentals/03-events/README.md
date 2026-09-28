# 3. Events

*Signals and connections, watching for change, touches and prompts, and one script that drives every tagged part.*

Roblox Fundamentals · Medium

---

Most of what happens in a place is not on a schedule you write: a player clicks a bell, a lever flips a door's attribute, a character steps on a pad, a script tags a new platform. Events are how your code hears about it. This module covers signals and the connections they return, and what Deferred signal behaviour means for when a handler runs; the change signals for properties, attributes and children; touches, the cooldown every touch pad needs, and proximity prompts; and it ends with one script that drives every tagged part in the place, including parts tagged later and parts whose tag is removed.

## Lessons

1. [Signals and connections](01-signals-and-connections.md) — Connect a function to an event, keep the connection it returns, and know when the handler actually runs.
2. [Watching for change](02-watching-for-change.md) — React when a property, an attribute or a child changes, using the signal that names exactly what you care about.
3. [Touches and prompts](03-touches-and-prompts.md) — Make a pad react once per touch with a cooldown, and give players a prompt they press on purpose.
4. [One script, every tagged part](04-tagged-behaviour.md) — Drive every part with a tag from one script: the parts tagged now, the parts tagged later, and none after the tag goes.
