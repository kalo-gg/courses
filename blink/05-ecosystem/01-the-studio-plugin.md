# The Studio plugin

*The same compiler inside Studio, for people who would rather not open a terminal.*

Blink · Tooling & Ecosystem · 10 min

---

Blink's compiler is Luau, so it can be compiled into a plugin. That is exactly what the Studio plugin is: an editor widget wrapped around the same parser and the same generator the CLI runs, bundled by the same build config. A description that fails in the terminal fails in Studio with the identical `[E####]` text. Install it from the Creator Store asset `77231976488966`, or drop the `blink-plugin.rbxm` attached to a release into your plugins folder.

**plugin/src/init.server.luau**

```luau
local ScriptEditorService = game:GetService("ScriptEditorService")

local Editor = require("@self/Editor")
local Profiler = require("@self/Profiler")
local State = require("@self/State")

local Generator = require("@compiler/Generator")
local Parser = require("@compiler/Parser")
```

`@compiler` is the repository's `src/`. The plugin adds an editor and a permissions prompt; the compiler underneath it is the one you already know.

|  | CLI | Plugin |
| --- | --- | --- |
| Where the description lives | a `.blink` file in the repository | inside the place |
| Output paths | the `ClientOutput` and `ServerOutput` options | a destination you pick in the explorer |
| Imports | any relative path | sibling `./` imports only |
| Version control | the file is in git | the place file is |

> **It asks for script injection, and it has to**
>
> Generating means writing ModuleScripts into the DataModel, which is script injection by Studio's definition. The plugin's own docs say so plainly. Grant it to the plugin you installed from the Creator Store listing above and to nothing else wearing the name.

Then pick where to generate. The modules have to sit somewhere both halves of the game can require, which in practice means `ReplicatedStorage`. Put them under `ServerScriptService` and the client's `require` never resolves. The plugin's documentation carries this as a warning because it is the mistake people make first.

## Plugin or CLI

The two are not complementary on one description. The plugin's copy lives in the place file and the CLI's lives in git, nothing keeps them in step, and after a week nobody can say which one produced the modules the game is running. Pick one per description and let the other project use the other.

**What the plugin cannot resolve**

```blink
-- Fine in both: the file sits beside this one.
import "./crosswire"

-- Fine from the CLI, which walks the file system.
-- The plugin resolves sibling imports only, so this one has nowhere to go.
import "../shared/crosswire"
```

A description written for the plugin has a flat layout. That is a real constraint on a growing project, and the reason most teams that already run Rojo stay with the CLI.

**Key points**

- The plugin is the same compiler: same parser, same generator, same `[E####]` text.
- Generating writes ModuleScripts into the DataModel, which is why it asks for script injection.
- Generate somewhere both sides can require, which in practice means ReplicatedStorage.
- The plugin resolves sibling imports only, so a plugin-first description stays flat.
