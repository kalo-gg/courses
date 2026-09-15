# Scopes and imports

*Group definitions with scope, split files with import, and never import the same file twice.*

Blink · Blink in a Real Project · 10 min

---

One file stops scaling somewhere around thirty declarations, and the first thing that suffers is the reader. Blink has two answers: `scope` groups declarations inside a file, and `import` pulls another file in as a scope of its own, named after the file.

**net.blink**

```blink
import "./crosswire"

scope Match {
    type Round = u8
    event Started {
        From: Server,
        Type: Reliable,
        Call: ManyAsync,
        Data: crosswire.Team
    }
}

event Damage {
    From: Client,
    Type: Reliable,
    Call: ManyAsync,
    Data: crosswire.Health
}
```

The two come out differently. A scoped *event* nests in the returned table, so you reach it at `Net.Match.Started.On(...)`. A scoped *type* flattens into one name with an underscore, because Luau has no nested type namespaces: `Match_Round`, `crosswire_Health`. An import behaves exactly like a scope with the file's name, and `as` renames it.

**what the compiler generated**

```luau
server: Match.Started.FireAll = function(Value: ("Red" | "Blue")): ()
server: Damage.On = function(Listener: (Player: Player, Value: number) -> ()): () -> ()
client: Match.Started.On = function(Listener: (Value: ("Red" | "Blue")) -> ()): () -> ()
client: Damage.Fire = function(Value: number): ()
type: crosswire_Health = number
type: crosswire_Team = ("Red" | "Blue")
type: Match_Round = number
```

Events nest. Types flatten with an underscore. The import's scope is named `crosswire`, after the file.

## The alias is not a string

`as` takes a bare identifier: `import "./combat" as Fight`. The documentation's imports page writes it as `import "./external" as "Common"`, with quotes, and that does not parse: it fails with `[E2002] Error: Unexpected token` and `Expected one of "As", "Import", "Boolean", "Keyword", "Identifier", found "String"`. The path stays quoted; the alias never is. The extension is optional either way.

> **Never import the same file twice**
>
> `import "./common"` and `import "./common" as Alt` in one file produce two independent scopes, and Blink numbers their events separately: `Queue.Reliable[0]` and `Queue.Reliable[1]` in the generated client. Every event in that file then exists twice, costs an id each, and a listener connected through one scope never hears anything fired through the other.

> **Scopes see outwards, not inwards**
>
> A top-level type is visible inside a scope without qualification, so a scope can use the file's shared types freely. The reverse is not true: a type declared inside a scope is reachable from outside only as `Scope.Name`.

## Your turn

**Key points**

- `scope` groups declarations in a file; `import` pulls a file in as a scope named after it.
- Scoped events nest (`Net.Match.Started`); scoped types flatten with an underscore (`Match_Round`).
- `as` takes a bare identifier, never a quoted string, and the file extension is optional.
- Importing one file twice duplicates every event in it, ids and listener tables included.
