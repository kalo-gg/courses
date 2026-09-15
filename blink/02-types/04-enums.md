# Unit enums and tagged enums

*One byte for a state, and a tagged enum is the union Luau will not give you.*

Blink · The Type System · 10 min

---

A unit enum is a closed set of names: a round is warming up, live, in overtime or done, and never anything else. A tagged enum is a closed set of *variants*, each of which carries its own struct of fields. Both cost one byte for the tag, because the tag is an index into a list the compiler wrote into both modules.

**match.blink**

```blink
enum Phase = { Warmup, Live, Overtime, Done }

enum MatchEvent = "Kind" {
    Chat { Text: string(1..140) },
    Scored { By: f64, Points: u8 }
}
```

One of each. The tagged form is the one with a string after the `=`.

That string is the **tag field name**. It becomes the discriminating key in the generated Luau type, and it is what your game code writes when it constructs a value and what it switches on when it receives one. Pick a name that reads well at the call site: `Kind`, `Type`, `State`. It is not a label for the enum, it is a key you will type a hundred times.

**Types.luau (generated, read-only)**

```luau
export type Phase = ("Warmup" | "Live" | "Overtime" | "Done")
export type MatchEvent = { Kind: "Chat", Text: string } | { Kind: "Scored", By: number, Points: number }
```

A unit enum becomes a union of string literals; a tagged enum becomes a union of tables sharing one literal key.

## The union Blink does not have

> **There is no union type, and that is on purpose**
>
> Blink has no `A | B` type, and its documentation says so and points at tagged enums instead. The reason is the wire: a union with no tag cannot be read back, because the reader has no way to know which branch it is looking at. A tagged enum is a union with the tag written down, which is the only kind that survives serialisation.

There is a case where a tagged enum is the wrong reach. If every variant carries the same fields and only the name differs, a struct with a unit enum field is the same size on the wire and a good deal easier to read: one shape, one state field, no branching in the handler at all.

## Collapsing three events into one

**Key points**

- A unit enum is a closed list of names and costs one byte, whatever the names are.
- A tagged enum is a closed list of variants, each carrying its own fields, and also costs one byte for the tag.
- The string after `=` names the discriminating key in the generated Luau type; pick one your game code reads well.
- Blink has no union type, because an untagged union cannot be read back off the wire.
- When every variant carries the same fields, a struct with a unit enum field is smaller to read and the same size to send.
