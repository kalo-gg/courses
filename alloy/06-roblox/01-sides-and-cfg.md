# Sides and @cfg

*Say which side a script runs on with its file name, and which side a function, statement or local belongs to with @cfg.*

Alloy · Alloy on Roblox · 10 min

---

A game starts from one shared `startup` that calls a server half and a client half. In Luau each half opens with `if RunService:IsServer()`, and the checks drift. A server-only function called from the client then returns nil, and the error shows up three calls later.

Alloy puts the side on the declaration with `@cfg`.

## One startup, two sides

**src/shared/startup.aly**

```alloy
@cfg(server)
local function start_server() -> ()
  print('the server starts')
end

@cfg(client)
local function start_client() -> ()
  print('the client starts')
end

export function startup() -> ()
  start_server()
  start_client()
end
```

**ship (lines 1–9)**

```luau
local __alloy = require("@alloy")
local function start_server(): ()
  if not (__alloy.cfg.server()) then return end print('the server starts')
end

local function start_client(): ()
  if not (__alloy.cfg.client()) then return end print('the client starts')
end
```

Each `@cfg` line goes blank, and the function's first line gets a guard. `__alloy.cfg.server()` reads RunService when the code runs, so nothing is removed at build time.

One rule covers every form. Code that gives nothing back does nothing on the wrong side. Code that owes the caller a value raises there, with a message such as `round_length is @cfg(server) and cannot run here`. A silent nil would break somewhere else. A function that returns a value counts as one with a value, whether or not you annotate it.

The conditions are `server`, `client`, `studio`, `edit`, `running` and `test`. Join them with `not`, `and`, `or`, `any(…)` and `all(…)`.

| Under `@cfg(server)` | On the client | Emit, on the same line |
| --- | --- | --- |
| A function that returns nothing | Returns at once | `if not (__alloy.cfg.server()) then return end` |
| A function that returns a value | Raises, naming the function | `if not (__alloy.cfg.server()) then error(…, 2) end` |
| A statement | Skipped | `if __alloy.cfg.server() then … end` |
| A local | Nil | `(if __alloy.cfg.server() then … else nil)` |

Two `@cfg` lines over one statement join with `and`. On a namespace, each member is nil on the other side.

The file name says where a script runs. `main.server.aly` builds a Script and `main.client.aly` a LocalScript. Any other file is a shared module. The style guide keeps entry scripts short: each imports `startup` from `src/shared/startup.aly` and calls it.

> **Recognising older code**
>
> Luau code checks sides by hand: `if RunService:IsServer() then start_server() end`. The style guide asks for `@cfg(server)` on the function instead.

**Key points**

- `.server.aly` builds a Script and `.client.aly` a LocalScript; any other file is shared.
- `@cfg` is checked at run time through RunService. Nothing is removed at build time.
- On the wrong side, a function with no value returns at once, and a function with a value raises.
- A `@cfg` statement is skipped, and a `@cfg` local is nil, on the other side.
