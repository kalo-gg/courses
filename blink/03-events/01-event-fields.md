# The fields of an event

*Three required fields describe an event; Data and the undocumented Poll are the optional two.*

Blink · Events & Functions · 10 min

---

An `event` declaration is a name and a small block of key-value fields. There is no free-form configuration in it: the parser holds a closed list of field names, and for each one a closed list of values, so a typo is caught where you wrote it rather than three weeks later in a listener that never fires.

**combat.blink**

```blink
event Damage {
    From: Server,
    Type: Reliable,
    Call: ManyAsync,
    Data: u8
}
```

Four fields: who fires it, how it travels, what the listening side gets, and what rides along.

| Field | Values | Required |
| --- | --- | --- |
| `From` | `Client`, `Server` | yes |
| `Type` | `Reliable`, `Unreliable` | yes |
| `Call` | `SingleSync`, `SingleAsync`, `ManySync`, `ManyAsync`, `Polling` | yes |
| `Poll` | `true`, `false` | no |
| `Data` | any type, or a parenthesised type pack | no |

The `EVENT` table in `src/Parser.luau`. Nothing else is accepted.

> **Poll is real, and undocumented**
>
> `Poll` is the fifth entry in the parser's `EVENT` list and appears on no documentation page. `Poll: true` gives that one event the polling API whatever its `Call` says, which is the per-event version of the `UsePolling` option: the generated client gets `Iter` and the deprecated `Next` instead of `On`.

## How the keys are spelled

The parser accepts a field key spelled exactly as declared (`From`) or entirely in lower case (`from`), and nothing in between. Line 1073 of `src/Parser.luau` is `if Key == Value or string.lower(Key) == Value then`, so `FRom` and `FROM` are both rejected. There are two spellings, not a case-insensitive match.

Field *values* are a different story: they are case-sensitive, and the failure names the wrong thing. `From: client` reports `[E2003] Error: Unknown option "client"` with `Expected one of "Client" or "Server"` under the caret. An option error for an event field reads like a compiler bug the first time you meet it; it is not, and the message under the caret still tells you the answer.

**net.blink**

```blink
event Hit {
    From: Client,
    Type: Reliable,
    Call: ManyAsync,
    Data: u8
}

event Healed {
    from: Server,
    type: Reliable,
    call: ManyAsync,
    data: u8
}
```

Both compile. Mixing the two spellings in one file is legal and unkind; pick one.

## Your turn

**Key points**

- `From`, `Type` and `Call` are required; `Data` and `Poll` are not.
- Every field value is checked against a closed list, so a typo fails at compile time.
- A key may be written as declared or entirely in lower case, and nothing else.
- A lower-case *value* reports `[E2003] Unknown option`, which names the field's legal values under the caret.
