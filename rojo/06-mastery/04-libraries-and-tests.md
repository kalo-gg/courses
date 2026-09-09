# Libraries and test places

*Ship a library as a model whose root is a ModuleScript, and build a test place beside it that keeps the specs in.*

Rojo · Structuring a Real Game · 10 min

---

A library is a model project: its root is the instance people drop into their game, usually a ModuleScript from `src/init.luau`, and it is built with a model extension so the root is included — `rojo build -o Util.rbxm`. `rojo init --kind model` writes the template. Games take it in two ways: through Wally, whose `_Index` entry is exactly this repository, or with a `$path` straight at the library's project file. Spec files live beside the code; `globIgnorePaths` keeps them out of the model.

**default.project.json (the library) and its build**

```json
{
  "name": "Util",
  "globIgnorePaths": ["**/*.spec.luau"],
  "tree": { "$path": "src" }
}
// src/init.luau → ModuleScript "Util" (the root); src/Clamp.luau → its child
// rojo build -o Util.rbxm
```

## A test place beside the library

Tests run in a place, so a library repository carries a second project file, `test.project.json`: a `DataModel` root, the library mounted under `ReplicatedStorage`, and a runner script under `ServerScriptService` that requires a test framework and the specs. Build it with `rojo build test.project.json -o test.rbxl` and open it in Studio, or hand it to a headless runner.

The trap is how the library is mounted. Pointing `$path` at `default.project.json` nests the library project, and `globIgnorePaths` accumulate across nested projects — so the specs are skipped exactly where you need them. Point `$path` at `src` instead: the same files, no inherited ignore.

**test.project.json**

```json
{
  "name": "test",
  "tree": {
    "$className": "DataModel",
    "ReplicatedStorage": {
      "Util": { "$path": "src" }
    },
    "ServerScriptService": {
      "Run": { "$path": "test/Run.server.luau" }
    }
  }
}
```

`Run` is a single file behind `$path`; the node's key names it. Nothing here ignores specs, so `Util.Clamp.spec` is in the tree.

> **Ignore globs flow downhill**
>
> `globIgnorePaths` is the one setting that survives a nested project file: every glob a parent declares keeps applying inside the projects it reaches. A test place that nests the library's project file inherits `**/*.spec.luau` and builds without a single spec, with nothing printed. Map the directory, not the project.

**Key points**

- A library is a model project: root from `src/init.luau`, built with `-o Name.rbxm`.
- `test.project.json` is a place that mounts `src` and adds a runner.
- `globIgnorePaths` accumulate into nested projects; mount the directory to keep the specs.
- `rojo build test.project.json -o test.rbxl` builds the test place; the library build ignores specs.
