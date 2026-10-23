# Cleanup

*`delete` cleans up whatever it is given, `destroy` and `after` wait, and a `Scope` ends a round in one line.*

Alloy · Modules, Results and async · 10 min

---

A round of a game makes a part, connects to an event, starts a countdown thread and registers an end-of-round callback. Ending it takes four different calls: `Destroy`, `Disconnect`, `task.cancel`, and a call to the callback. Forget the second one and the handler keeps running in the next round.

## delete

**One word for every cleanup**

```alloy
const part = new Instance('Part') { Parent = workspace }
const connection = part.Touched:Connect(function(hit) print(hit.Name) end)
const thread = task.delay(5, function() print('later') end)
const stop = function() print('stopped') end

delete connection -- disconnects it
delete thread -- cancels it
delete stop -- calls it
delete part -- destroys it
```

Each line emits `__alloy.delete(x)`, which looks at the value at run time and picks the cleanup. `delete t.field` also sets the field to nil afterwards, so the table never holds something already destroyed.

`destroy x` calls the value's destroy method and nothing else, so it says exactly what happens to an Instance. `destroy coin after 10` waits first. `after 3 do … end` runs a block later, and `after 3 where alive do` reads its condition when the timer fires, not when it was scheduled. `delete` takes no timer.

**Cleanup on a timer**

```alloy
const coin = new Instance('Part') { Name = 'Coin', Parent = workspace }
destroy coin after 10

local alive = true
after 3 where alive do
  print('respawn')
end
alive = false
```

`destroy coin after 10` emits `game:GetService("Debris"):AddItem(coin, 10)`, which outlives the script. `after` emits `task.delay(3, function() if alive then … end end)`. `return` inside an `after` block leaves the block, not the function around it.

## Scope and Signal

A `Scope` from `@alloy/std/async` is a bag of cleanups. `scope:add(x)` takes anything `delete` accepts and hands it back, so it wraps a line without changing it. `delete scope` cleans the lot, newest first. The std `Signal`, from `@alloy/std/signal`, has the shape of a Roblox signal, with snake_case twins such as `connect` and `fire`.

**round.aly**

```alloy
import { Scope } from '@alloy/std/async'
import { Signal } from '@alloy/std/signal'

local scope = new Scope()
const damaged = scope:add(new Signal<<Player, number>>())

scope:add(damaged:Connect(function(player, amount)
  print(`{player.Name} took {amount}`)
end))
scope:add(function() print('round over') end)

damaged:Fire(sim.join('Ada'), 10)
delete scope
```

`delete scope` runs the function, then disconnects the handler, then destroys the signal. `scope:add(connection, part)` names an Instance the item belongs to, and `delete part` then disconnects the item before the `Destroy`.

**Key points**

- `delete x` picks the cleanup at run time: destroy, disconnect, cancel, call, or close a `Scope`.
- `destroy x` calls only the destroy method. `destroy x after n` uses `Debris` for an Instance.
- `after n do … end` is `task.delay`, and its `where` condition is read when the timer fires.
- A `Scope` from `@alloy/std/async` holds cleanups. `add` returns its argument, and `delete scope` cleans newest first.
- The std `Signal`, from `@alloy/std/signal`, has a Roblox signal's shape with snake_case twins.
