# Scripts and suffixes

*.server.luau is a Script, .client.luau a LocalScript, plain .luau a ModuleScript; the suffix is stripped from the name.*

Rojo · Sync Rules · 9 min

---

Rojo checks a file's name against a list of patterns, top to bottom, and the first match decides the class. The instance's name is the file name with the matched suffix removed, so `Shop.server.luau` becomes a `Script` called `Shop`. The old `.lua` extension works exactly like `.luau`.

| File | Instance | Notes |
| --- | --- | --- |
| `Name.server.luau` | `Script` **Name** | Runs on the server |
| `Name.client.luau` | `LocalScript` **Name** | Runs on the client |
| `Name.plugin.luau` | `Script` **Name**, marked as a plugin script | For Studio plugins (7.5.0) |
| `Name.luau` | `ModuleScript` **Name** | Everything that is not server, client or plugin |

**Four files through the rules**

```text
src/server/Shop.server.luau    → Script       "Shop"
src/client/Camera.client.luau  → LocalScript  "Camera"
src/shared/Inventory.luau      → ModuleScript "Inventory"
src/shared/notes.md            → nothing: no rule matches, the file is ignored
```

## Names, not folders

Rojo never looks at the folder to choose a class. A file called `Camera.server.luau` is a `Script` even inside `src/client`, and a `Util.client.luau` in `src/shared` is a `LocalScript` sitting in `ReplicatedStorage`, where it will never run. The folder decides *where* an instance lands, because the project file maps folders; the name decides *what* it is. Keep the two in your head separately and the odd-looking cases stop being odd.

> **Which suffix for a module the server requires?**
>
> None. Only *where* the module lives matters for who can require it, not its suffix. A ModuleScript in ServerScriptService or ServerStorage is server-only; one in ReplicatedStorage is reachable from both sides.

The stripped name is what scripts see at runtime. `Shop.server.luau` is `script.Name == "Shop"`, and a module in `src/shared/Inventory.luau` is required as `ReplicatedStorage.Shared.Inventory`, never `Inventory.luau`. The old `.lua` extension follows the same rules with the same stripping.

**src/server/Shop.server.luau**

```luau
local ReplicatedStorage = game:GetService("ReplicatedStorage")
-- Inventory.luau synced as ModuleScript "Inventory"; the suffix is gone
local Inventory = require(ReplicatedStorage.Shared.Inventory)

print(script.Name) -- "Shop", not "Shop.server"
```

**Key points**

- The first matching pattern wins: `.server.luau` → Script, `.client.luau` → LocalScript, `.plugin.luau` → plugin Script, `.luau` → ModuleScript.
- The matched suffix is removed to make the instance name.
- Location comes from the project file, class from the file name; `.lua` and `.luau` behave the same.
