# What is Alloy?

*A strict superset of Luau that compiles each construct to fixed Luau on the same line.*

Alloy · Foundations · 10 min

---

Roblox code spends a lot of lines asking whether something is nil. A profile may not have loaded, and its settings may be missing. Each guard is a line to write and a line to check.

**Alloy** is a strict superset of Luau: every Luau file is already an Alloy file. It adds what Luau code keeps writing by hand, starting with safe access, and compiles each construct to plain Luau.

## Same line in, same line out

**Luau, by hand**

```luau
local function volume_of(profile)
  if profile and profile.settings and profile.settings.volume then
    return profile.settings.volume
  end
  return 0.5
end
```

**volume.aly**

```alloy
local function volume_of(profile) -> number
  return profile?.settings?.volume ?? 0.5
end

print(volume_of(nil)) -- 0.5
```

`?.` reads a field only when the value on its left is not nil, and `??` supplies a value for nil. `-> number` is the return type.

**What the compiler emits**

```luau
local function volume_of(profile): number
  local _1 = (if profile == nil then nil else profile.settings) local _2 = (if _1 == nil then nil else _1.volume) return (if _2 == nil then 0.5 else _2)
end

print(volume_of(nil)) return nil -- 0.5
```

Five lines in, five lines out. Line 2 holds the whole chain: a temp for each step and an `if` for each guard. `-> number` became Luau's `: number`, and a module with no exports ends by returning nil, as Roblox requires of a ModuleScript.

The compiler writes two artifacts from each file. The **ship** artifact is what Roblox runs. The **check** artifact is what the type checker, luau-lsp, reads. It keeps what the ship leaves out, such as inline tests, which the ship blanks line for line.

Alloy's standard library is one runtime module, `alloy.luau`, written at the output root. A file that uses it, through an array literal for example, gains `local __alloy = require(…)` on its first line of code.

| File | What it is |
| --- | --- |
| `x.aly` | An Alloy module. It builds to `x.luau`. |
| `x.server.aly`, `x.client.aly` | A Script or a LocalScript (Module 6). |
| `x.alx` | A module with markup (Module 5). |
| `x.d.aly` | Declarations that every file sees. The build writes nothing for it. |
| `x.luau` | Plain Luau, copied to the output as it is. |

All of these can require each other.

## Errors name their book section

**Terminal**

```text
$ alloy check
src/main.aly:2:1 error(6.1): ConstError: `MAX_HP` is a `const`; its value is set once and a reassignment is an error
```

Path, line and column, the level with its section, then the kind and the message. The editor shows the same code as `Alloy(6.1)`. Line 1 declared `const MAX_HP = 100` and line 2 assigned to it.

Every diagnostic names the section of Alloy's book that explains it. `error(6.1)` is section 6.1, and `alloy doc 6.1` prints it in the terminal.

Lints are separate, and never stop a build. Pasted Luau still compiles, and the lints point to the Alloy form: `manual_safe_access` reads `profile and profile.settings` and suggests `profile?.settings`.

**Key points**

- Alloy is a strict superset of Luau: every Luau file is already an Alloy file.
- Each construct emits fixed Luau on the line you wrote it, so line N of the output is line N of the source.
- The ship artifact is what Roblox runs; the check artifact is what luau-lsp reads.
- A file that uses the std gains a runtime `require` on its first line of code, and the build writes `alloy.luau` at the output root.
- A diagnostic names its book section, as in `error(6.1)` and `alloy doc 6.1`. Lints never stop a build.
