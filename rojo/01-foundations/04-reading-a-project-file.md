# Reading a project file

*The keys of tree, service inference, $path, and the top-level settings around them.*

Rojo · Rojo Foundations · 9 min

---

A project file is JSON with one required key, `tree`, describing the root instance. Inside it, every key that starts with `$` configures the instance itself, and every other key is a **child** named after the key. That rule nests: a child can have its own `$` keys and children, as deep as you like.

**One node, annotated**

```json
"ReplicatedStorage": {            // a child of the root, named after the key
  "Shared": {                     // a child of ReplicatedStorage
    "$path": "src/shared",        // $ keys configure Shared itself
    "Remotes": { "$className": "Folder" }   // no $: another child
  }
}
```

| Key | Meaning | Example |
| --- | --- | --- |
| `$className` | The instance's class | `"$className": "Part"` |
| `$path` | Files or a folder to pull in, relative to the project file | `"$path": "src/shared"` |
| `$properties` | Property values to set | `"$properties": { "Anchored": true }` |
| `$ignoreUnknownInstances` | Leave instances Rojo did not create alone | What Rojo owns |
| anything else | A child instance with that name | `"Shared": { … }` |

## Where classes come from

An instance needs a class, and Rojo finds it in one of three ways. An explicit `$className` always wins. Otherwise the class comes from the `$path`: a directory is a `Folder`, `init.server.luau` makes it a `Script`, and so on. Failing both, Rojo **infers** from the name: under `DataModel`, a child named `ReplicatedStorage` or `Workspace` becomes that service; under `StarterPlayer`, `StarterPlayerScripts` and `StarterCharacterScripts` are inferred; under `Workspace`, `Terrain`.

> **No class, no instance**
>
> A node with neither `$className` nor `$path` and a name Rojo cannot infer is an error. `"Assets": {}` under ReplicatedStorage fails; write `"Assets": { "$className": "Folder" }`.

Around `tree` sit optional settings: `name` (what the root instance is called in a built model; since 7.4.1 `default.project.json` may omit it and takes its folder's name, other project files still need it), `servePort`, `servePlaceIds`, `globIgnorePaths`, `emitLegacyScripts` and `syncRules`. Project Files in Depth and The Rojo Workflow use each of them.

**The settings around tree**

```json
{
  "name": "arena",
  "servePort": 34872,
  "globIgnorePaths": ["**/*.spec.luau"],
  "emitLegacyScripts": true,
  "tree": { "$className": "DataModel" }
}
```

Top-level keys are a fixed list; a misspelled one is a parse error. Inside `tree`, an unknown key is a child instance.

**Key points**

- Inside `tree`, `$` keys configure an instance; every other key is a child named after it.
- Class resolution: `$className`, else the class the `$path` produces, else inference from the name (services under DataModel).
- A node with no class, no path and an un-inferable name is an error.
- Top-level settings (`name`, `servePort`, `globIgnorePaths`, `emitLegacyScripts`, `syncRules`) sit beside `tree`; comments are allowed since 7.6.1.
