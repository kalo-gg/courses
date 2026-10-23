# Extension methods

*Add methods to `Vector3`, Instance classes and `string` with `export impl`, and see how each call is dispatched.*

Alloy · Types, traits and extensions · 10 min

---

Placement code flattens positions everywhere: `new Vector3(pos.X, 0, pos.Z)`. A helper, `flat(pos)`, reads inside out next to `pos.Unit` and `pos:Dot(dir)`, and you cannot add a method to `Vector3`: Roblox owns the type.

`export impl` can. It adds methods to a type you do not own, such as a datatype, an Instance class or a primitive like `string`, and every file in the project can call them.

## Methods on someone else's type

**placement.aly**

```alloy
export impl Vector3
  function flat(self) -> Vector3
    return new Vector3(self.X, 0, self.Z)
  end
end

const pos = new Vector3(4, 10, 2)
print(pos:flat())
```

**The ship artifact**

```luau
local __alloy = require("@alloy") do local __impl = __alloy.impl_for("Vector3")
  function __impl.flat(self): Vector3
    return Vector3.new(self.X, 0, self.Z)
  end
end

const pos = Vector3.new(4, 10, 2)
print(__alloy.call(pos, "flat")) return nil
```

Line 1 fetches the registry table for `Vector3`, and the method goes into it. Line 8's call becomes `__alloy.call(pos, "flat")`, still on line 8.

The impl fills a registry when its file runs. Each call by an extension's method name becomes `__alloy.call`, which finds the method at run time.

The check artifact, which the editor and `alloy flux` read, keeps `pos:flat()` and declares `flat` on `Vector3`, so it types and completes like a built-in. A primitive has no class to declare it on, so for `string` the check calls a typed helper table instead: `__alloy_string.shout(('hi'))`.

## How a call finds its method

`__alloy.call` reads the receiver's kind: `typeof` for a datatype or a primitive, and the most specific class for an Instance. If that kind has the extension, it runs. If not, the receiver's own method runs, so a struct with its own `flat` keeps it.

**Another file in the project**

```alloy
export impl BasePart
  function lift(self, dy: number) -> ()
    self.Position += new Vector3(0, dy, 0)
  end
end

struct Route
  stops: number
end

impl Route
  function flat(self) -> string
    return 'route'
  end
end

part:lift(4) -- a Part or a MeshPart: both are BaseParts
print(new Route { stops = 3 }:flat()) -- route
```

Extension names are collected across the project, so `flat` from `placement.aly` routes `route:flat()` through `__alloy.call` here too. A `Route` has no extension, so its own method runs.

> **Don't reuse a built-in name**
>
> On its own kind, an extension wins. An `export impl Vector3` that writes `Cross` replaces Roblox's `Cross` for every `Vector3` in the project. Give extensions names Roblox does not use.

**Key points**

- `export impl Type` adds methods to a type you do not own: a datatype, an Instance class or a primitive.
- The ship turns each call by an extension name into `__alloy.call(receiver, "name")`; the check artifact keeps the method call, through a typed helper table for a primitive.
- The dispatcher runs the extension for the receiver's kind, else the receiver's own method.
- On its own kind an extension wins, so never reuse a built-in method name.
