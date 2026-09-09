# Meta files

*Name.meta.json adds properties and attributes to its neighbor; init.meta.json can also change a Folder's class.*

Rojo · Sync Rules · 10 min

---

File names carry a class and a name, nothing more. To set a property on a script (`Disabled`), add attributes, or make a directory something other than a Folder, put a **meta file** beside it. `Name.meta.json` describes the sibling **file** that would be called `Name`; it never becomes an instance itself.

**src/gui/Hud/Controller.meta.json**

```json
{
  "properties": { "Disabled": true },
  "attributes": { "Version": { "Int32": 3 } }
}
```

Applies to `Controller.client.luau` in the same directory. `3` alone would be a `Float64`, so the `Int32` wrapper is what makes it an integer.

It has to be a file: put `Stuff.meta.json` next to a `Stuff/` directory and nothing reads it — the directory's meta file is `Stuff/init.meta.json`. The same is true beside a `.model.json`, an `.rbxm` or an `.rbxmx`, which carry their own properties. In every one of those cases Rojo says nothing at all; the properties just never appear.

## Directories: init.meta.json

For a directory, the meta file is `init.meta.json` and it gains one extra key: `className`. A directory that would be a Folder can become a `ScreenGui`, a `Model`, a `Configuration`, anything, and keep its files as children. The class swap only works when the directory *would* have been a Folder; a directory with `init.server.luau` is already a Script and `className` there is an error.

**src/gui/Hud/init.meta.json**

```json
{
  "className": "ScreenGui",
  "properties": { "ResetOnSpawn": false }
}
```

> **`className` first, then `properties`**
>
> Rojo applies `className` before it resolves `properties`, and it resolves each property against the class the instance has *at that moment*. Leave `className` out of the file above and `ResetOnSpawn` is checked against `Folder`, which has no such property, so the whole build stops with `Unknown property Folder.ResetOnSpawn` — the directory does not quietly become a Folder with a stray property, you get nothing. If a build dies on `Unknown property Folder.Something`, a meta file is missing its class.

| Key | In `Name.meta.json` | In `init.meta.json` |
| --- | --- | --- |
| `properties` | yes | yes |
| `attributes` | yes | yes |
| `ignoreUnknownInstances` | yes | yes |
| `id` | yes (for `Ref` links, 7.5.0) | yes |
| `className` | no | yes, Folder directories only |

**Key points**

- `Name.meta.json` adds `properties`, `attributes`, `ignoreUnknownInstances` or `id` to the sibling **file** named `Name`; beside a directory or a model file it is read by nobody.
- `init.meta.json` does the same for a directory and can set `className` when the directory would be a Folder.
- `className` is applied before `properties`, so a property that only exists on the new class needs the `className` line — otherwise Rojo stops with `Unknown property Folder.X`.
- Meta files never create instances on their own, and they ignore keys they do not recognize, so a typo costs you the property in silence.
