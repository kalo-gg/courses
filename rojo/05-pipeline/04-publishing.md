# Publishing places and plugins

*Push a built place to Roblox with rojo upload, and build a Studio plugin straight into the plugins folder.*

Rojo · Packages, Syncback & CI · 9 min

---

A repository produces two kinds of artifact. A **place** goes to Roblox: `rojo upload` builds the project and publishes it to an existing place id, through the Open Cloud API with an API key or, with no key, through your Studio login cookie. A **plugin** goes to Studio: a plugin is just a Script with its modules as children, saved in the local plugins folder, and `rojo build --plugin` writes it there for you.

```bash
rojo upload --asset_id 1234567890 --api_key "$ROBLOX_API_KEY" --universe_id 987654321   # Open Cloud (CI)
rojo upload --asset_id 1234567890                                                        # Studio cookie, found automatically
```

The three id flags are spelled with underscores. Both forms build the project first; there is no separate build step.

## Plugins

`rojo init --kind plugin` writes a project whose root is the plugin script rather than a `DataModel`, and its entry file is `init.server.luau` — a legacy `Script`, which is all Studio needs to run a plugin. Two newer rules exist: since 7.5.0 a `Name.plugin.luau` file becomes a `Script` with `RunContext = Plugin`, and since 7.7.0 `init.plugin.luau` does the same for a directory.

`rojo build --plugin MyPlugin.rbxm` then writes the model into Studio's local plugins folder — a bare `.rbxm`/`.rbxmx` name, nothing else. Studio has no reload-plugins button: enable PluginDebugService in Studio's settings, then right-click your plugin under **PluginDebugService** in the Explorer and choose **Save and Reload Plugin** (or **Save and Reload All Plugins in Debugger**, Ctrl+Shift+L).

**A plugin project**

```json
// default.project.json
{ "name": "Hello", "tree": { "$path": "src" } }
// src/init.plugin.luau   → Script "Hello", RunContext Plugin (the root)
// src/Toolbar.luau       → ModuleScript "Toolbar", its child
// rojo build --plugin Hello.rbxm
```

> **Two ways to be quiet**
>
> `rojo build --plugin` with a path like `C:\plugins\x.rbxm` stops with “plugin flag path cannot be absolute”; that one you will see. `init.plugin.luau` on 7.5.x or 7.6.x is the quiet one: the directory builds as if the file were not there. Check `rojo --version` before blaming the file.

**Key points**

- `rojo upload --asset_id … --api_key … --universe_id …` publishes through Open Cloud; without a key it uses your Studio cookie.
- A plugin project's root is the plugin Script; `.plugin.luau` (7.5.0) and `init.plugin.luau` (7.7.0) give it `RunContext = Plugin`.
- `rojo build --plugin Name.rbxm` writes straight into Studio's plugins folder; bare name, model formats only.
- Keys are secrets; cookies never leave your machine.
