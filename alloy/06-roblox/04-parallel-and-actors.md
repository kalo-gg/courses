# Parallel Luau

*Run a block in the parallel phase, let the compiler refuse what the engine would, and talk to actors with typed messages.*

Alloy · Alloy on Roblox · 11 min

---

Scanning thousands of parts every frame is work Roblox can spread across cores, with **Parallel Luau**. A script under an `Actor` calls `task.desynchronize()`, reads in parallel, then calls `task.synchronize()` before it writes. Forget the second call, or write a property in between, and the engine errors at run time.

Alloy turns the pair into a block, and checks the block at compile time.

## parallel do

```alloy
const parts: { BasePart } = {}
const hits: { BasePart } = {}

parallel do
  for _, part in parts do
    if part.Position.Y < 0 then
      table.insert(hits, part)
    end
  end
end

for _, part in hits do
  part.Position = new Vector3(0, 10, 0)
end
```

**ship (lines 4–10)**

```luau
task.desynchronize() do
  for _, part in parts do
    if part.Position.Y < 0 then
      table.insert(hits, part)
    end
  end
task.synchronize() end
```

The header's line gets `task.desynchronize()` and the `end`'s line gets `task.synchronize()`. The writes come after the block, in the serial phase.

The check is a guard rail, not a proof. It reads this file and goes no further. A call into another module is not followed, and a write through a value whose type the file does not know is not seen. A loop variable over an unannotated table is one. The engine still errors on those, at run time.

## Actors and messages

`worker.server.actor.aly` builds an `Actor` named `worker` around its Script, with an `init.meta.json`. One file makes one Actor, so clone it for a pool. A `message` is a typed channel to an Actor. `fire` sends to the Actor passed first. `on` binds on the calling script's own Actor, so it needs an `.actor.` file. A message with `reply(…)` gives the handler `respond`, and any script can listen with `replied`.

**src/server/worker.server.actor.aly**

```alloy
import { Scan } from './jobs'

-- jobs.aly: export message Scan(job: number, parts: { BasePart }) reply(job: number, fallen: number)

Scan.on(function(job, parts, respond)
  local fallen = 0
  parallel do
    for _, part in parts do
      if part.Position.Y < 0 then fallen += 1 end
    end
  end
  respond(job, fallen)
end)
```

The main script clones the Actor, binds `Scan.replied(function(job, fallen) … end)`, and sends work with `Scan.fire(worker, 1, parts)`. Nothing matches a reply to its request, so the job number travels both ways.

> **Messages carry no serializer**
>
> An actor boundary is not a network boundary, so arguments cross as they are. A struct, a payload enum or a `HashMap` keeps its methods on a metatable, which the engine drops, so a message parameter of one of those types is a `WireType` error. In this sandbox every file compiles as one plain script, so `Scan.on` cannot compile here. The exercise runs `parallel do` under `sim.runInActor` instead.

**Key points**

- `parallel do … end` emits `task.desynchronize()` and `task.synchronize()` on the lines of its header and its `end`.
- `ParallelError` refuses Instance writes, `new Instance`, `Destroy`, `Clone`, `await` and an early `return` that it can see in this file.
- It is a guard rail, not a proof: the engine still has the last word at run time.
- `x.server.actor.aly` builds an Actor. A `message` fires to an Actor, and `on` binds inside an `.actor.` script.
