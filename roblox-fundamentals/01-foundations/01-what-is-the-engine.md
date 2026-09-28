# What the engine is

*A place is one tree of instances rooted at game. Scripts change the tree, and the engine does everything else.*

Roblox Fundamentals · Foundations · 9 min

---

Your obby has a door that should open when a round starts. Roblox has no `openDoor` command. The door is a part, and to open it a script finds that part and changes two of its properties.

That is all a Roblox script does: it finds objects and changes them. The engine draws the result, simulates it and sends it to every player.

## One tree, rooted at `game`

**A place as Studio's Explorer shows it (names on the left, classes on the right)**

```text
game                        DataModel
├── Workspace               Workspace
│   ├── Camera              Camera
│   ├── Terrain             Terrain
│   ├── Baseplate           Part
│   └── Door                Part
├── Players                 Players
├── Lighting                Lighting
├── ReplicatedStorage       ReplicatedStorage
├── ServerScriptService     ServerScriptService
│   └── Main                Script
└── ServerStorage           ServerStorage
```

Every object in a place is an **instance**. Each instance has a `Name`, a `ClassName` that says what kind of thing it is, and a `Parent` that says where it sits. The root of the tree is `game`, whose class is **DataModel**. Its children are **services**. Workspace is the service that holds the 3D world, and you can also reach it through the global `workspace`.

**Opening the door**

```luau
print(game.ClassName)           --> DataModel
print(workspace.Parent == game) --> true

local door = workspace.Door
door.Transparency = 0.5
door.CanCollide = false
```

These two property changes give every player a faded door that they can walk through. Nothing here draws anything.

## Studio: edit, test, stop

Studio is the editor for this tree. In edit mode nothing runs: you place parts and write scripts. A playtest starts the engine, so scripts run, physics moves unanchored parts and players join. Stopping the test resets every instance to how it was before, so anything your scripts changed is gone.

| Mode | Shortcut | What happens |
| --- | --- | --- |
| **Test** | F5 | Starts the game and puts your avatar at a SpawnLocation, or near (0, 100, 0) if there is none. |
| **Run** | F8 | Starts the game with no avatar. You move around with the Studio camera. |
| **Stop** | Shift+F5 | Ends the test and resets every instance to how it was before. |

Choose Test or Run from the dropdown next to Studio's Play button. Older guides call Test *Play*.

| Window | What it shows |
| --- | --- |
| Explorer | The tree, by Name. Click an instance to select it. |
| Properties | Every property of the selected instance, which you can edit here. |
| Output | Lines from `print` and `warn`, and errors with the script and line that raised them. |
| Developer Console | The same log in a running game, with separate Client and Server views. |

A published game has no Output window. Press **F9**, or type `/console` in chat, to open the Developer Console. Only the owner and group members who can edit the game see its Server log.

> **In the sandbox**
>
> There is no Studio and no Play button here. Your code runs as a server Script named `Main` in ServerScriptService, and `print` lines appear under the editor. Before your first line runs, each exercise builds a scene that you cannot see.

**Key points**

- A place is one tree of instances. Its root is `game`, the DataModel, and its children are services.
- Every instance has a `Name`, a `ClassName` and a `Parent`.
- Scripts change instances. The engine draws, simulates and replicates whatever the tree says.
- Test (F5) and Run (F8) start the engine. Stop resets the tree to how it was before the test.
- `print` writes to the Output in Studio. In a live game, open the Developer Console with F9.
