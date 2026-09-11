# 2. Typing Data

*Describing tables, optionals, unions and the aliases that let a shape have a name.*

Luau Intermediate · Easy

---

Most Luau data is tables, so most Luau types describe tables. This module covers the table type and its indexers, the optional type that models a value which may be absent, unions and the literal types that make them precise, and the aliases that let a shape be named once and used everywhere, including across module boundaries.

## Lessons

1. [Typing a table](01-table-types.md) — Field by field for records, an indexer for lists, and the difference that catches people out.
2. [Optionals and nil](02-optionals.md) — A question mark means the value may be nil, and the checker will not let you use it until you have checked.
3. [Unions and literal types](03-unions.md) — A value that is one of several shapes, and the string literals that let you tell them apart.
4. [Aliases and export type](04-aliases.md) — Name a shape once with type, share it between files with export type, and give it parameters.
