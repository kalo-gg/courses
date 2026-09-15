# Blink or Zap

*Two IDLs with the same idea, and the differences that actually decide it.*

Blink · Tooling & Ecosystem · 10 min

---

Zap is the other answer to the same problem, and it is the one most Roblox projects compare Blink against. Both compile a description into a server module and a client module that pack fields into buffers, both validate on receipt, and both replace a folder of RemoteEvents with two. Blink's README credits Zap for the range and array syntax, which is why the two languages look so alike on the page.

|  | Blink 0.18.8 | Zap 0.6.x |
| --- | --- | --- |
| Written in | Luau, built into executables by Lune | Rust |
| Where you write it | a terminal, or the Studio plugin | a terminal, or the playground at zap.redblox.dev |
| Polling | `Call: Polling`, drained with `Event.Iter()` | `call: Polling`, drained with `event.iter()`, since 0.6.18 |
| Licence | MIT | MIT |
| Maintained by | 1Axen | the red-blox organisation |

Blink's column was read from its sources at 0.18.8. Zap's was read from its own documentation and repository on 2026-09-15; check them again before you quote a version number at anyone.

Bandwidth is a tie. On Blink's own Entities benchmark, run 2025-04-30 on the author's machine, the median is 41.81 Kbps for Blink, 41.71 for Zap and 41.64 for ByteNet. The number worth quoting is the other column: plain Roblox remotes carrying the same data measured 559,364 Kbps. Pick either tool over remotes; do not pick between them on bytes.

## Porting a description

**The same event, twice**

```text
-- Zap
event Damage = {
    from: Server,
    type: Reliable,
    call: ManyAsync,
    data: u8(0..100),
}

-- Blink
event Damage {
    From: Server,
    Type: Reliable,
    Call: ManyAsync,
    Data: u8(0..100)
}
```

Three differences, all mechanical: Zap assigns the body with `=`, spells the four field names in lower case, and tolerates a trailing comma. The range in the middle is identical, because that is the part Blink borrowed.

Neither tool ships a migration command, and neither needs one. A typical description is twenty or thirty declarations of four lines each, the call modes are spelled the same, and the types that differ announce themselves by failing to parse. Half an hour, once.

**crosswire.zap (the three events to port)**

```text
event Spawn = {
    from: Server,
    type: Reliable,
    call: ManyAsync,
    data: Vector3,
}

event Aim = {
    from: Client,
    type: Unreliable,
    call: SingleSync,
    data: u16,
}

event Chat = {
    from: Client,
    type: Reliable,
    call: ManyAsync,
    data: string(1..200),
}
```

**Key points**

- Same idea, different spelling: a description compiled into two modules of packed buffers.
- Blink's ranges and arrays come from Zap, which is why half a port is a copy.
- The rest is mechanical: drop the `=` and capitalise the four field names.
- Bandwidth is a tie within half a percent; both are orders of magnitude under plain remotes.
