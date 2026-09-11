# Getting your types back

*Link modules are bare require thunks, so exported types vanish until you put them back.*

Wally · Wally in a Real Project · 10 min

---

You install a package, require it, and your editor knows nothing. No autocomplete on its methods, no error when you pass a string where it wanted a number, and `Signal.Connection` reported as an unknown type. The package definitely exports it — you can open the file in `_Index` and read it. Something between there and here is eating it, and it is the link module.

**_Index/sleitnick_signal@2.1.0/signal/init.luau**

```luau
export type Connection = {
	Connected: boolean,
	Disconnect: (self: Connection) -> (),
}

local Signal = {}
Signal.__index = Signal

function Signal.new()
	return setmetatable({}, Signal)
end

return Signal
```

The real module, deep inside `_Index`. It exports a type and returns a table.

**Packages/Signal.lua**

```luau
return require(script.Parent._Index["sleitnick_signal@2.1.0"]["signal"])
```

The link module Wally writes for the alias `Signal`. That one line is the entire file — and note the extension: `.lua`, not `.luau`.

The link is a **value** expression. `require` hands back the table the module returned, and a table is a runtime thing: it carries no type exports with it. `export type` is a statement, and statements do not travel through a return. So the module on the far side exports `Connection` and the module you actually require exports nothing at all, because it has no export statements in it.

This is not a Luau bug and it is not fixable by writing the require differently. Any module that is only a forwarding `return require(...)` loses every type the target exported.

## Putting them back

**Three commands, in this order**

```bash
$ wally install
$ rojo sourcemap default.project.json --output sourcemap.json
$ wally-package-types --sourcemap sourcemap.json Packages/
```

`wally-package-types` is its own tool: `rokit add JohnnyMorganz/wally-package-types`. Run the three together or not at all.

The order is forced by what each step needs. `wally install` has to run first because it is the thing that creates the link modules. `rojo sourcemap` reads your project file and writes a JSON tree tying every file on disk to the instance path it will occupy in the place. `wally-package-types` needs that map to follow each link across to the real module, read what it exports, and rewrite the link so the exports come back out the other side.

> **Re-run it after every install**
>
> `wally install` deletes `Packages/`, `ServerPackages/` and `DevPackages/` **entirely** before it writes them, so every rewritten link module is destroyed by the next install. This is not a one-off setup step you do once and forget: put all three commands in one script, or one CI job, and never call `wally install` on its own again.

**What a link has to look like to carry a type**

```luau
local REQUIRED_MODULE = require(script.Parent._Index["sleitnick_signal@2.1.0"]["signal"])

export type Connection = REQUIRED_MODULE.Connection

return REQUIRED_MODULE
```

You never write this by hand — it is generated. The point is the shape: a type has to be re-exported by name, because nothing rides along with the returned value.

**Key points**

- A link module is one line, `return require(...)`, and a returned value carries no exported types.
- Reaching past the link into `_Index` works and hardcodes a resolved version into your source. Do not.
- `wally install`, then `rojo sourcemap`, then `wally-package-types --sourcemap sourcemap.json Packages/`.
- `install` wipes the package directories, so the restore belongs in the same script — every time, not once.
