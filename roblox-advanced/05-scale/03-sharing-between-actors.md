# Sharing between actors

*Send an Actor a message, which copies its arguments, or share a SharedTable, which every Actor sees at once.*

Roblox Advanced · Scale · 11 min

---

Four squads of NPCs each run under their own Actor, and all four need to know which player each NPC is chasing. A plain table will not do: each Actor has its own Luau VM, so a table in one script is invisible to the others, and a ModuleScript loads a separate copy in each.

There are two ways across: send a **message** to an Actor, or share a **SharedTable**.

## Messages

`actor:SendMessage(topic, ...)` returns at once. A script under that Actor receives the arguments in a callback bound with `BindToMessage`, which runs serially, or `BindToMessageParallel`.

The arguments are **copied** across the VM boundary. A large table is copied in full on every send, and a change the receiver makes to its copy never reaches the sender. Functions cannot be sent.

**A director and a squad**

```luau
-- The director: a Script under no Actor
local Workspace = game:GetService("Workspace")

for _, squad in Workspace.Squads:GetChildren() do
	squad:SendMessage("Hunt", 101) -- a User.Id: cheap to copy
end

-- A Script inside each squad's Actor
local actor = script:GetActor()

actor:BindToMessageParallel("Hunt", function(userId: number)
	-- the parallel phase, with its own copy of the arguments
	print(actor.Name, "hunts", userId)
end)
```

Each send reaches one Actor, which can bind several callbacks to a topic. Only scripts under that Actor can bind to its messages.

## SharedTable

A `SharedTable` is one table that every Actor sees, with nothing copied: a write from one Actor is visible to all at once, even from the parallel phase. Keys are strings or non-negative integers. Values are booleans, numbers, vectors, strings, other SharedTables or serialisable data types. Share one by name through `SharedTableRegistry`, or send it in a message, which passes the table itself.

**One table, every Actor**

```luau
local SharedTableRegistry = game:GetService("SharedTableRegistry")

-- Any script, under any Actor or none: the same table by name
local targets = SharedTableRegistry:GetSharedTable("Targets")

targets.Grunt1 = 101 -- every Actor sees this at once
print(targets.Grunt1, SharedTable.size(targets))

-- Read-then-write can interleave with another Actor: change it atomically
targets.Kills = 0
SharedTable.increment(targets, "Kills", 1)
SharedTable.update(targets, "Grunt1", function(current)
	return if current == 101 then 102 else current
end)
```

`GetSharedTable` creates the table the first time a name is used. `update` may call its function more than once while other Actors write, so keep the function free of side effects. `increment` is the faster form for numbers.

|  | `SendMessage` | `SharedTable` |
| --- | --- | --- |
| The data | Copied on every send | One table, never copied |
| Who sees it | The one Actor you send to | Every script that holds it |
| A change by the receiver | Stays in its copy | Visible to all at once |
| Good for | Commands and small arguments | Large or shared state |

`SharedTable.clone` also avoids a full copy: it shares structure and gives a consistent snapshot while others write. `cloneAndFreeze` makes that snapshot read-only.

**Key points**

- Each Actor has its own Luau VM: plain tables and module state are not shared between Actors.
- `SendMessage` copies its arguments to one Actor, which receives them through `BindToMessage` or `BindToMessageParallel`.
- A SharedTable is shared, not copied: every Actor sees each write at once, even from the parallel phase.
- Get one by name with `SharedTableRegistry:GetSharedTable`. When several Actors write one key, use `SharedTable.update` or `increment`.
