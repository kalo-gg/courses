# Tuning the linter

*Start from nothing with recommended = false, set options for one lint, and read a lint's own explanation.*

Larvae · Formatting & Linting · 10 min

---

The defaults are a starting point, not a verdict. Two projects want opposite things from them: one wants to switch on the eleven that start at `allow`, the other wants exactly one lint and silence from the rest. Both are two lines, and the second starts with a key `larvae init` already wrote into your config.

**`recommended = false`**

```toml
[lint]
recommended = false      # every lint starts at allow

[lint.rules]
shadowing = "warn"       # one lint on, the other sixty-three off
```

Three states: absent and `true` both mean larvae's defaults apply; `false` starts every lint at `allow`, so a project gets the lints it names and no others. A level the project wrote wins in either state. `larvae init` writes `recommended = true` into the config it generates, so you see the key on day one rather than finding it in the docs later.

## Settings for one lint

**`[lint.options.<lint>]`**

```toml
[lint.options.unused_variable]
parameters = false        # a parameter is part of a signature the caller decides
loop_variables = false
ignore_pattern = "^_"

[lint.options.restricted_globals]
loadstring = "compiles a string at runtime, ship the code instead"
getfenv = "turns the optimizer off for this function"

[lint.options.deprecated]
additional = { "myOldHelper" = "myNewHelper" }
```

selene calls this table `config`, and that spelling is accepted too — write one or the other, not both. Note the shape of `restricted_globals`: a name and *the reason a reader is shown*, because a bare list would report "restricted" and leave them guessing. `restricted_module_paths` takes the same shape under `paths`.

Which globals exist before a file runs is a setting too: `std = "roblox"` is the default, `"luau"` and `"none"` narrow it, and `globals` adds the names your own runtime defines so `undefined_variable` stops reporting them. selene calls these the same things, and an existing `selene.toml` is read first with `[lint]` layered over it — so the paste is literal, with one translation: selene's `error` is spelled `deny` here, because there is no `error` level.

**selene's spellings, accepted**

```toml
[lint]
std = "roblox"
globals = ["shared", "_TESTING"]

[lint.rules]
global_usage = "deny"          # selene.toml said "error"; larvae has no such level

[lint.config]                  # selene's name for [lint.options]: the same table
unused_variable = { ignore_pattern = "^_" }
```

Write `[lint.options]` or `[lint.config]`, not both.

`larvae lint --explain <name>` is the last piece: it prints what a lint looks for, in the lint's own words, and takes a worm's dotted name as readily as a builtin's.

**Ask the lint**

```bash
larvae lint --explain undefined_variable       # what the lint is, in its own words
larvae lint --explain luaux.useless_fragment   # a worm's lint, by its dotted name
```

**Key points**

- `recommended = false` starts every lint at `allow`; the project names what it wants.
- `[lint.options.<lint>]` holds a lint's settings; selene's `[lint.config]` spelling is the same table.
- `std` says which globals exist; `globals` adds your own; selene's `error` is `deny`.
- `larvae lint --explain <name>` describes any of the 64, worm lints included.
