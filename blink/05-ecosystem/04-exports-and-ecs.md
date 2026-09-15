# Exports and ECS replication

*export gives a type its own Read and Write, which is how a component gets replicated.*

Blink · Tooling & Ecosystem · 11 min

---

Put `export` in front of a type declaration and Blink adds that type to the returned table with two methods instead of event methods: `Read(buffer)` gives you the value back, and `Write(value)` gives you a buffer. No event, no remote, no id. It is the serialiser Blink was already generating for your events, handed to you to call yourself.

**entity.blink**

```blink
export struct Entity {
    id: u8,
    pos: vector,
    angle: u16
}
```

> **There is no `=`**
>
> The documentation's Exports section writes `export struct MyInterface = { ... }`. The parser rejects it: `[E2002] Error: Unexpected token ... Expected "OpenBraces", found "Assign"`. The source is right and the docs are wrong; write the struct exactly as you would without the keyword.

**Client.luau (generated)**

```luau
Read = function(Buffer: buffer): (Entity)
    -- ...
    -- Read BLOCK: 15 bytes
    local BLOCK_START = Read(15)
    local Value = {} :: any
    Value.id = buffer.readu8(RecieveBuffer, BLOCK_START + 0)
    local X = buffer.readf32(RecieveBuffer, BLOCK_START + 1)
    local Y = buffer.readf32(RecieveBuffer, BLOCK_START + 5)
    local Z = buffer.readf32(RecieveBuffer, BLOCK_START + 9)
    Value.pos = Vector3.new(X, Y, Z)
    Value.angle = buffer.readu16(RecieveBuffer, BLOCK_START + 13)
    -- ...
    return Value
end
```

`u8` + three `f32` + `u16` = 15 bytes, and there is no event id byte, because an export is not an event.

Reading that block comment is how you price a component. It is the only honest number: the compiler wrote it, it changes when you change a field, and `vector` is three `f32`s unless you narrow the encoding. Twelve of those fifteen bytes are one position at full precision.

> **Exports cannot hold `Instance` or `unknown`**
>
> Both of those ride a side channel that belongs to a RemoteEvent: the instance array and Roblox's own serialiser. An export has no remote, so there is nowhere for them to ride. Everything else, including nested structs, maps and enums, is fair game.

## Why an ECS wants this

A replication library, replecs or one you wrote, produces a buffer per component per tick and needs something to fill it. Exports are that something: a component table goes in, bytes come out, and the same pair runs in reverse on the client. Pair them with a `Call: Polling` event so the incoming buffers queue up and get drained inside a system, on your schedule, rather than in a callback on Blink's.

**Key points**

- `export` adds `Read(buffer)` and `Write(value)` to the returned table for that type.
- Write it like a plain struct: the docs' `=` does not parse.
- An export has no event id byte, so its block is exactly the sum of its fields.
- `Instance` and `unknown` cannot be exported: they need a remote that an export does not have.
- Exports plus a `Call: Polling` event are the two halves of ECS replication over Blink.
