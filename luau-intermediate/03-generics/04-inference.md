# When not to annotate

*The checker infers most of it; annotate the boundaries and leave the middle of a function alone.*

Luau Intermediate · Generics & Functions · 9 min

---

Three lessons of syntax leave a temptation to use all of it. Resist that. An annotation on a value whose type is already obvious costs a reader a moment and buys nothing, and a file where every local is annotated is a file where the ones that matter no longer stand out.

The rule is about **boundaries**. Annotate where a value crosses into your code or out of it. Leave the interior of a function to inference, which is very good at it.

**Over-annotated, and the same code trusting inference**

```luau
--!strict
-- Every annotation here is already obvious from the line it is on.
local names: { string } = { "lamp", "rope" }
local count: number = #names
for i: number, name: string in names do
	print(i, name)
end

-- The same code. The checker infers all four types identically.
local items = { "lamp", "rope" }
local total = #items
for i, item in items do
	print(i, item)
end
```

Both halves print `1  lamp` and `2  rope`. The checker knows `count` and `total` are numbers for exactly the same reason you do: `#` returns one.

| Annotate | Why | Leave it inferred |
| --- | --- | --- |
| A module's exported functions | Callers are checked against the signature, not the body | Locals holding an obvious literal |
| Function parameters | Strict mode will not infer them, and they are the contract | Loop variables from a typed list |
| An empty table | There is nothing to infer from | The result of a call to a typed function |
| A table whose fields are filled in later | Same reason: the literal does not show the shape | A concatenation, a comparison, a length |

The left column is the boundary; the right column is the interior. Nearly every annotation worth writing is in the left.

## The empty table problem

`local t = {}` is the one line where inference has nothing to work with. Luau gives it an empty table type, and what happens next depends on the mode and on how the table is used — sometimes the shape is learned from the assignments that follow, sometimes it is not, and either way the reads are not checked against anything you meant.

So say what it is. `local slots: { string } = {}` costs four words and makes every insert and every read a checked one.

**The one annotation you should almost never skip**

```luau
--!strict
-- Inferred from nothing.
local seen = {}
seen["lamp"] = 2

-- Says what it is, so reads and writes are both checked.
local counts: { [string]: number } = {}
counts["lamp"] = 2

print(seen.lamp, counts.lamp)
```

Prints `2  2` — identical at run time, which is the point: the difference is entirely in what the checker can tell you before you get there.

There is one more habit in the same family. When you delete code, delete its annotations too — a parameter that is now unused and still typed is a claim nobody checks, and a local that has been reduced to a constant does not need `: number` any more.

Annotations are documentation that the checker enforces. Like documentation, the stale ones are worse than none.

**Key points**

- Annotate **boundaries** — exported functions, parameters, returns — and leave the interior to inference.
- An annotation restating something obvious costs a reader attention and buys nothing.
- The empty table is the exception: `local t = {}` has nothing to infer from, so say what goes in it.
- Loop variables, lengths, comparisons and literals never need help.
- Delete annotations along with the code they described; a stale one is worse than none.
