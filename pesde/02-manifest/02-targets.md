# Targets: lib, bin and build_files

*What a package exports for each environment, and the field a Roblox target cannot be published without.*

pesde · The Manifest · 9 min

---

`[target]` answers two questions: which runtime the package is for, and what it exports. `lib` is what `require` resolves to; `bin` is what runs. A package may have either or both — a formatter that is also a library, say — and the environment decides which are allowed.

The environment is also why one name and version can exist four times on the registry: `luau`, `lune`, `roblox` and `roblox_server` are four separate publications of the same package, each with its own `[target]` and its own install directory on the consumer's disk.

Resolution picks the one that matches the consumer's project unless their specifier names a `target`, so a wrong `environment` never fails loudly — it publishes something nobody asks for, and the consumer is told only that no build exists for their target.

**What the package exports**

```toml
[target]
environment = "luau"
lib = "init.luau"
bin = "main.luau"
```

`lib` is what `require` gets. `bin` is what runs. A package may have either or both.

| Environment | lib | bin | build_files | [target.scripts] |
| --- | --- | --- | --- | --- |
| `luau` | allowed | allowed | no | allowed |
| `lune` | allowed | allowed | no | allowed |
| `roblox` | allowed | **no** | **required** | no |
| `roblox_server` | allowed | **no** | **required** | no |

Everything in this table is enforced at publish time; a Roblox target with no `build_files` is refused.

## Roblox needs `build_files`

> **A Roblox package also needs `build_files`**
>
> It lists the top-level files and directories a sync tool should bring into Roblox — usually just `build_files = ["src"]`. It is **required**: `pesde publish` refuses a Roblox target whose `build_files` is missing or empty. You never list `roblox_packages`, which is always synced. Roblox Projects covers why this exists.

> **Roblox packages cannot export a `bin`**
>
> `bin` is allowed only under the `luau` and `lune` environments — there is no sense in which a Roblox package is an executable. `lib` is allowed everywhere.

**A Roblox target**

```toml
[target]
environment = "roblox"
lib = "src/init.luau"
build_files = ["src"]
```

No `bin`, and `build_files` is not optional. It lists top-level files and directories — never `roblox_packages`, which is always synced — and it is what the consumer's `roblox_sync_config_generator` reads.

**Key points**

- `lib` for require, `bin` for run; both allowed in `luau` and `lune`.
- A Roblox target cannot have `bin` and cannot publish without `build_files`.
- `build_files` lists top-level files and directories — never `roblox_packages`.
- The environment also decides the install directory consumers see.
