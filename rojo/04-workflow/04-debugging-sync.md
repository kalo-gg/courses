# When sync goes wrong

*Read Rojo's errors, warnings and silences, then fix a project that fails to build for five different reasons.*

Rojo · The Rojo Workflow · 10 min

---

Something did not show up in Studio. Before touching the project file, look in three places. The **terminal** running `rojo serve` or `rojo build` prints errors, which stop the build, and warnings, which do not. The **plugin** shows a confirmation before its first sync and refuses connections it is not allowed to make. And **silence** is a diagnostic too: a missing optional path, a meta file beside the wrong thing or a misspelled meta key produce no message at all, only an absence.

**A build with one error and two warnings**

```text
Property 'Name' cannot be set manually, ignoring. Attempted to set in 'Baseplate' at default.project.json
default.project.json: Keys starting with '$' are reserved by Rojo to ensure forward compatibility. This project uses the key '$propertes', which should be renamed.
error: default.project.json: "$propertes" has no $className and no $path
```

The warnings are the real clue: a misspelled `$` key is not dropped, it becomes a child instance with no class, and that is the error.

## Errors, warnings and silence

| Message or symptom | Cause | Fix |
| --- | --- | --- |
| `Rojo project referred to a file using $path that could not be turned into a Roblox Instance by Rojo.` / `Check that the file exists and is a file type known by Rojo.` / `Project path: …` / `File $path: …` (error) | A misspelled or missing path | Compare with the file list; wrap in `{ "optional": … }` only if missing is legitimate |
| `Instance "X" is missing some required information.` / `One of the following must be true:` / `- $className must be set to the name of a Roblox class` / `- $path must be set to a path of an instance` / `- The instance must be a known service, like ReplicatedStorage` (error) | A pure node with an un-inferable name, or a misspelled `$` key that became a child | Add `$className`, or fix the key |
| `If $className and $path are both set, $path must refer to a Folder` (error) | A class on a node whose path is already a script or model | Drop `$className`, or move the files into a plain directory |
| `Unknown property Folder.ResetOnSpawn` (error) | An `init.meta.json` with `properties` but no `className` | Add `className` — it is applied first |
| `Property 'Name' cannot be set manually, ignoring` (warning) | `Name` or `Parent` in `$properties` | Rename the key or the file |
| `Keys starting with '$' are reserved by Rojo` (warning) | A misspelled `$` key | Fix the spelling; the phantom child goes away with it |
| Nothing printed, instance missing | An optional path is absent, or a meta file sits beside a directory / `.model.json` / `.rbxm`, or a meta key is misspelled | Check the folder exists; use `init.meta.json` inside directories; check spelling |
| `Model at path … had a top-level Name field` (warning) | `name` at the top of a `.model.json` | Rename the file instead |
| A `$` typo whose value is not an object, e.g. `"$ignoreUnknown": false` (error, and the project never loads) | Every unrecognized key becomes a child node, so a boolean where an instance description belongs is a load error before any tree is built — no reserved-key warning, no partial build | Fix the spelling. The in-browser build reports it as `"$ignoreUnknown" must be an object describing an instance` |

The first two rows are Rojo's own multi-line output; the in-browser build paraphrases each into a single line — `default.project.json: $path "src/shard" for "Shared" does not exist` and `default.project.json: "X" has no $className and no $path` — and prints every other row's wording verbatim.

> **A typo behaves three different ways**
>
> Rojo's project struct rejects unknown fields, so a misspelled **top-level** key (`"emitLegacyScript"`) is a hard parse error you cannot miss. Inside `tree` there is no such list: every key that is not one of the six `$` keys is a **child instance name**. Drop the `$` from `$properties` and Rojo tries to build a child called `properties`, then complains that it “is missing some required information” — an error naming something you never meant to create. Misspell a `$` key and you also get “Keys starting with '$' are reserved by Rojo” in the log. Meta files are the quietest of all: they do not reject unknown fields, so `"propertys"` in a `.meta.json` is simply ignored and your property never appears. When something is not syncing, check the spelling before anything else.

## The plugin side

When the terminal is clean and Studio still does nothing, the problem is the connection. The plugin refuses a CLI whose protocol version does not match its own, so `rokit install` and `rojo plugin install` are the first move after any upgrade. It refuses a place whose id is not in `servePlaceIds` or is in `blockedPlaceIds`; that is the setting doing its job, not a bug.

A wrong port means the plugin's Connect panel and the `rojo serve` banner disagree — read both. The confirmation dialog is worth a slow read every time: a long remove list on an existing place usually means a `$path` node points at a folder that does not yet hold that place's scripts. In Team Create, keep one person connected at a time.

**Before the exercise: the broken project**

```json
"Workspace": {
  "$propertes": { "FilteringEnabled": true },
  "Baseplate": {
    "$className": "Part",
    "properties": { "Anchored": true, "Size": [512, 20, 512] },
    "$properties": { "Name": "Ground" }
  }
}
```

Two spellings of one key, three diagnostics: the reserved-key warning for `$propertes`, then a class-less-child error for it and another for `properties`. Both typos hold an **object**, which is why the build gets far enough to complain about the children rather than refusing to load the file at all.

**Key points**

- Errors stop the build; warnings change it; optional paths, misplaced meta files and misspelled meta keys say nothing at all.
- A misspelled `$` key or a missing `$` becomes a child instance with no class — the reserved-key warning is the hint.
- `Unknown property Folder.X` means a meta file needs its `className`.
- When the terminal is clean, check the plugin: versions, `servePlaceIds`, the port, and the confirmation's remove list.
