# A test place for Studio

*Where the project file does belong: a place that mounts your package and its dependencies for Studio and Luau LSP.*

pesde · Roblox Projects · 9 min

---

You still want to open your package in Studio, and you still want Luau LSP to understand it. Both come from a project file that describes a **place to test in**, kept separate from the package itself.

**test-place.project.json**

```json
{
	"tree": {
		"$className": "DataModel",
		"ReplicatedStorage": {
			"package": {
				"$className": "Folder",
				"src": {
					"$path": "src"
				},
				"roblox_packages": {
					"$path": "roblox_packages"
				}
			}
		}
	}
}
```

Both the source and the installed dependencies are mounted, so requires resolve in Studio.

**Serving it**

```bash
rojo serve test-place.project.json
```

> **Point Luau LSP at the same file**
>
> Set `luau-lsp.sourcemap.rojoProjectFile` to `test-place.project.json`. The language server builds its sourcemap from that project, which is what gives you go-to-definition and types across your package *and* its dependencies.

## What ends up in the datamodel

Studio sees the tree the project file describes. `src/init.luau` becomes a ModuleScript named `src` with its siblings as children — `init.luau` names its own directory, so there is never an instance called `init`.

`roblox_packages` is a Folder whose direct children are the **linker** ModuleScripts, one per alias, and whose `.pesde` folder holds the installed packages themselves. A test script requires the alias, never the `.pesde` path.

**The built tree**

```text
ReplicatedStorage
  package [Folder]
    src [ModuleScript]          <- src/init.luau
      Table [ModuleScript]      <- src/Table.luau
    roblox_packages [Folder]
      signal [ModuleScript]     <- roblox_packages/signal.luau (linker)
      .pesde [Folder]           <- the installed packages
```

**Key points**

- The project file lives beside the package, under a name pesde never reads, and is not in `includes`.
- `rojo serve test-place.project.json`; point `luau-lsp.sourcemap.rojoProjectFile` at it.
- Mount both `src` and `roblox_packages` so requires resolve in Studio.
- Require linkers by alias; `.pesde` is an implementation detail.
