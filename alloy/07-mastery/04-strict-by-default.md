# Strict by default

*The contracts Alloy holds at compile time, Luau strict mode on top of them, and the one directive for an error you mean.*

Alloy · The toolchain and mastery · 11 min

---

In Luau, a missing branch, an unset field or a half-written interface shows up at run time, in a server log, for whichever player hit it first. Alloy moves those three to compile time. The file doesn't build, and the message names the line, the rule and the book section.

## The compile-time contracts

| Contract | Broken, it is | Section |
| --- | --- | --- |
| A `match` covers every variant, or has a `default` | `ExhaustiveMatch` | 4.2 |
| `new Name { … }` sets every field without a default, and no unknown one | `ConstructorError`, `StructError` | 3.6 |
| `impl Trait for S` writes every method the trait leaves abstract | `TraitContract` | 3.6 |
| `a->b` and `a=>b` read from an `Instance` | `AlloyError` when the type is known | 4.1 |
| A `remote` carries only data: no function, thread, Future or Signal | `WireType` | 4.3 |

`@sealed` belongs on this list too, but it's checked differently: a write to an undeclared field raises at run time, and the type checker reports it under `alloy flux`.

**alloy check, on the file in the exercise below**

```text
src/gear.aly:18:6 error(3.6): TraitContract: `impl Wearable for Helmet` does not write `label`; the trait declares it
src/gear.aly:25:10 error(4.2): ExhaustiveMatch: this match is not exhaustive: `Slot` has no arm for `Feet`; add it or a `default` arm
src/gear.aly:31:18 error(3.6): ConstructorError: `new Helmet { ... }` leaves `armor` unset; a field without a default needs a value
```

Each line gives the place, then the book section, then the kind. `alloy doc 3.6` explains the first and the last.

Types come on top of the contracts. `alloy init` writes a `.config.luau` with `languagemode = "strict"`, so every file type-checks strictly under `alloy flux` and in the editor. A `--!nonstrict` line at the top of one file still wins for that file. `[lint] strict = true` is a separate setting: it turns on the pedantic lint group, including `implicit_any`, `explicit_any` and `missing_return_type`.

**.config.luau, as alloy init writes it**

```luau
return {
    luau = {
        languagemode = "strict",
        aliases = {
            alloy = "./build/alloy",
        },
    },
}
```

If the folder already has a `.config.luau` or `.luaurc`, init adds only the mode and the alias it lacks, and keeps every other key.

Sometimes the error is the point, for example in a test of the contract itself. Mark the line with `--@alloy-expect-error` and a reason. It silences that line, and it reports once the line comes clean, so a stale expectation shows up. To debug the output, read the two artifacts side by side. **ship** is what Roblox runs. **check** keeps the types luau-lsp sees. Both keep your line numbers.

```alloy
enum Light
  Red
  Green
end

local function flip(l: Light) -> Light
  --@alloy-expect-error the test of ExhaustiveMatch
  return match l with
    case Red then Light.Green
  end
end
```

Without a reason, the directive draws the `missing_reason` lint. Use it for an error you mean, never to make a build pass.

**Key points**

- `match`, `new` and `impl` are compile-time contracts: `ExhaustiveMatch` (4.2), `ConstructorError` and `StructError`, and `TraitContract` (3.6).
- `@sealed` is checked at run time and by the type checker, and types come from luau-lsp under `alloy flux`.
- `alloy init` writes `languagemode = "strict"`. A `--!nonstrict` line wins for one file, and `[lint] strict` is the pedantic lint group.
- `--@alloy-expect-error <reason>` marks an error you mean, and it reports once the line comes clean.
- ship is what runs and check keeps the types. Both keep your lines.
