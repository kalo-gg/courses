# Structuring a screen

*Lay a Fluid screen out before you write it, and plan the one you are about to build.*

Fluid · Mastery & Migration · 11 min

---

Fluid gives you no structure, which means the structure is a decision. The one that holds up is boring: state in its own module, one file per component, one root per screen, and a single `cleanup` at the top of the tree. Every rule in this course has a place in it, and the reason to be explicit is that none of them are enforced.

| File | Holds | Never holds |
| --- | --- | --- |
| `state.luau` (or the caller) | Sources, and the functions that write them | Instances |
| `components/<Name>.luau` | One component function, taking props | A `fluid.root` of its own |
| `screen.luau` | The layout: which components, in what order, inside what frame | Game logic |
| `main.client.luau` | The mount, and the teardown that pairs with it | Anything reusable |

## The plan for the round-end screen

The final project is a round-end screen with four moving parts, one per module of this course: a panel that springs in from off-screen, a profile that arrives late through an async node, a countdown driven by an interval, and the scoreboard you already wrote — reused unchanged, because a mid project's finished file is the final project's read-only one.

**The tree**

```text
main.client.luau     mount into PlayerGui, and unmount
screen.luau          the panel: the spring, the layout, the four children
profile.luau         fluid.async + a state-aware label
countdown.luau       fluid.interval at 1 Hz
scoreboard.luau      read-only: exactly what Project 1 finished with
Packages/fluid.luau  read-only: the re-export
default.project.json read-only: the Rojo tree the place is built from
```

Four editable files, three read-only. The read-only ones are the two you should not have to think about and the one you already finished.

> **One root per screen, not per component**
>
> A component that opens its own root cannot be torn down by its parent, which is the whole point of the arrangement. Nest a root only when a piece genuinely outlives the screen — a toast that must survive the panel closing, for instance.

**screen.luau**

```luau
local fluid = require("./Packages/fluid")
local Stat = require("./components/Stat")

local function Screen(props)
	local panel = fluid.create("Frame")({
		Name = "Panel",

		fluid.create("UIListLayout")({}),
		Stat({ name = "Score", order = 1, value = props.score }),
	})

	fluid.cleanup(panel) -- the caller's scope now owns the whole tree
	return panel
end

return Screen
```

There is no `fluid.root` in this file, and that is the point: `main.client.luau` opens the one root, and this single `cleanup` at the top of the tree is what makes that root's destructor *remove* the screen rather than only stop it.

Props over globals, for the same reason: a component that reads a module-level source can only ever be used once. Pass the readable in, and the same component serves the live screen, a preview and a test with three different sources behind it.

> **Before 1.0**
>
> This course teaches `0.7.0-rc.7`, which is ahead of the stable **v0.6.5**. Pin the exact version with `=`, read the changelog before upgrading, and treat anything you find written about Fluid online as possibly describing a different version than the one you installed.

**The whole course**

- `source` holds state and is read by calling it; `derive` computes lazily; `effect` reacts eagerly; `deferred` waits for the flush; none of them may yield — that is what `async` is for.
- `root` owns the reactive graph and nothing else. Instances go only through `fluid.cleanup`.
- `create` takes properties and children in one mixed table, connects signal-valued properties, and applies no defaults of its own.
- `for_values` maps a table, keyed by value, unordered, and an entry can ask for time before it goes.
- `spring` and `lerp` animate different sets of types, and `provide_scheduler` hands you the frame loop.
