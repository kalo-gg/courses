# Options

*Thirteen options, and the error message that lists them is the only complete reference.*

Blink · Blink in a Real Project · 10 min

---

Options sit at the top of the file, one per line, written `option Name = Value`. The compiler validates the name against a closed list and the value against that name's kind: the `OPTIONS` table in `src/Parser.luau` maps each of the thirteen to `Identifier`, `String` or `Boolean`, so a quoted boolean or a bare string is caught the same way a misspelled name is.

**terminal**

```luau
[E2003] Error: Unknown option "Nonsense"
    ╭─[net.blink:1:5]
    │
003 │ option Nonsense = true
    ┆        ────┬────
    ┆            │
    ┆            ╰── Expected one of "Typescript" or "PromiseLibrary" or "ClientOutput" or "RemoteScope" or "Casing" or "WriteValidations" or "SyncValidation" or "ServerOutput" or "FutureLibrary" or "TypesOutput" or "UsePolling" or "ManualReplication" or "UseColon"
    │
────╯
```

The `[net.blink:1:5]` trailer is the source window the compiler printed, not a line and column: line 1 through the file's last line plus one.

That list is the complete reference, and it is more complete than the documentation: the options page names twelve. The thirteenth, `UseColon`, is only in the parser.

| Option | Kind | Default | Group |
| --- | --- | --- | --- |
| `ClientOutput` | String | none; the CLI stops without it | where the output goes |
| `ServerOutput` | String | none; the CLI stops without it | where the output goes |
| `TypesOutput` | String | none; the types stay in the two modules | where the output goes |
| `Typescript` | Boolean | `false` | where the output goes |
| `Casing` | Identifier | `Pascal` | what it is called |
| `RemoteScope` | String | `BLINK` | what it is called |
| `UsePolling` | Boolean | `false` | run time |
| `ManualReplication` | Boolean | `false` | run time |
| `SyncValidation` | Boolean | `true` | run time |
| `WriteValidations` | Boolean | `false` | run time |
| `FutureLibrary` | String | none | libraries |
| `PromiseLibrary` | String | none | libraries |
| `UseColon` | Boolean | `false` | inert |

> **UseColon does nothing**
>
> The parser accepts it as a boolean and no other file in `src/` reads it: `grep -rn UseColon src/` returns two hits, the type and the table. Generated output with and without it is byte for byte identical. It is in the diagnostic's list because the parser knows the name, not because it has an effect.

## Casing

**net.blink**

```blink
option Casing = Camel

event Damage {
    From: Server,
    Type: Reliable,
    Call: ManyAsync,
    Data: u8
}
```

With `Camel`, `On` becomes `on`, `FireAll` becomes `fireAll` and `StepReplication` becomes `stepReplication`. `Snake` gives `fire_all` and `step_replication`.

Eleven names are renamed, not two. The `Cases` table in `src/Settings.luau` lists `Fire`, `FireAll`, `FireList`, `FireExcept`, `On`, `Invoke`, `StepReplication`, `Next`, `Iter`, `Read` and `Write`. Pick the casing once, at the start of a project: changing it later rewrites every call site you have.

## Your turn

**Key points**

- Thirteen options; the `[E2003]` diagnostic lists all of them and the docs list twelve.
- Each name has a kind, so a quoted boolean fails the same way a misspelled name does.
- `Casing` renames eleven generated methods, not two, so pick it before you have call sites.
- `RemoteScope` is what lets two copies of Blink share one place; `UseColon` does nothing at all.
