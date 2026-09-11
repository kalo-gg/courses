# 4. Metatables

*The hook under every table: what a metatable is, and the class pattern it makes possible.*

Luau Intermediate · Medium

---

A table does what you tell it until you give it a metatable, at which point it can answer for keys it does not have, respond to operators, and behave like an object. This module builds up from the one metamethod that matters most, then uses it for the class pattern that most Luau codebases eventually reach for, including the inheritance that follows.

## Lessons

1. [The table behind the table](01-what-is-a-metatable.md) — setmetatable attaches a second table of behaviour, consulted only when the first one comes up short.
2. [Reads that miss and writes that are new](02-index-and-newindex.md) — One handles reads that miss, the other handles writes to keys that are absent, and neither is symmetric.
3. [The class pattern](03-classes.md) — A table of methods, an index pointing at itself, a new that sets the metatable, and self.
4. [Inheritance, and when not to](04-inheritance.md) — A metatable on the class table itself gives you a parent, and composition usually beats it anyway.
