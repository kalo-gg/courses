# 2. Instances

*Classes and properties, finding what you need in the tree, cloning templates, and attaching your own data with attributes and tags.*

Roblox Fundamentals · Easy

---

Everything in a place is an instance, and this module is about handling them well. It starts with classes: what `ClassName` and `IsA` tell you, and the types a property can hold. Then it finds instances by name, by class and by ancestry without tripping over `attempt to index nil`, copies templates kept in ServerStorage with `Clone` and removes instances for good with `Destroy`. It ends with attributes and tags, the two ways to attach your own data to any instance and find it again.

## Lessons

1. [Classes and properties](01-classes-and-properties.md) — Every instance has a class: `IsA` asks about the whole family, `ClassName` only the exact class, and each property holds one type.
2. [Finding instances](02-finding-instances.md) — Dot indexing, `FindFirstChild` and its class-based relatives, searching every depth, and where `attempt to index nil` comes from.
3. [Cloning and destroying](03-cloning-and-destroying.md) — Copy templates kept in ServerStorage with `Clone`, and remove instances for good with `Destroy`.
4. [Attributes and tags](04-attributes-and-tags.md) — Attach your own values to any instance with attributes, label instances with tags, and find every tagged one.
