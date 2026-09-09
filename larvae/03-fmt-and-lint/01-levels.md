# Levels and groups

*Four levels, six lints that deny by default, and groups that never wake a lint that is allowed on purpose.*

Larvae · Formatting & Linting · 10 min

---

larvae ships 64 lints, and every one has a level. Four levels exist, and the difference between two of them is subtler than it looks.

| Level | Reports? | Fails the run? | In an editor |
| --- | --- | --- | --- |
| `allow` | No | No | Nothing |
| `info` | Yes | No | A hint |
| `warn` | Yes | No | A squiggle |
| `deny` | Yes | **Yes** | A squiggle |

`info` and `warn` differ only in how loudly they are drawn. Neither changes the exit code.

## Why six lints deny by default

The project states the rule plainly: a lint denies only when the code cannot be what the author meant, and no reading of it makes the finding wrong. Everything softer warns, and a matter of taste stays `allow` until a project asks for it. That gives six denies, eleven allows, and forty-seven warns.

(Counted from the source. `larvae.example.toml` says sixty-three and the README says thirty-four — the set grows every release, and the docs lag it. Sixty-four is what `lint/lints/mod.rs` actually registers at 0.8.1.)

| The six denies | Why there is no other reading |
| --- | --- |
| `undefined_variable` | The name is nil, and the line throws. |
| `duplicate_keys` | The first entry is discarded. |
| `duplicate_local` | The first binding is dead on arrival. |
| `format_string` | `string.format` raises on this one. |
| `zero_step_loop` | The counter never moves, so it hangs. |
| `length_as_condition` | `if #t then` is true even for an empty table. |

> **Eleven start switched off**
>
> `prefer_const`, `high_cyclomatic_complexity`, `else_after_return`, `collapsible_if` and the rest are matters of taste, and the reasoning is given per lint — `and_or_conditional` is allowed because `x = c and a or b` is a style, and `implicit_any_parameter` because most Luau carries no annotations. Turn them on when the project wants them.

**Groups, and one lint at a time**

```toml
[lint.groups]
performance = "info"     # the six kinds: correctness, suspicious,
complexity = "allow"     # style, complexity, performance, roblox

[lint.rules]
unused_variable = "deny" # a named lint always beats its group
```

A group never wakes a lint that is `allow` on purpose, so `style = "info"` leaves `prefer_const` off.

**The six groups**

```toml
[lint.groups]
correctness = "warn"    # 29 lints: the code cannot do what it says
suspicious = "warn"     # 16: probably wrong, and a human decides
style = "allow"         # 12: how the code reads
performance = "info"    #  3: correct, and it costs more than it has to
roblox = "warn"         #  3: a Roblox data type used in a way that does not hold
complexity = "allow"    #  1: more shape than the job needs
```

Twenty-nine plus sixteen plus twelve plus three plus three plus one is sixty-four. A named lint always beats its group.

## Luau's own lints

Twenty-eight of the sixty-four are Luau's own compiler lints. larvae covers all of them and keeps the level Luau gives each one; where Luau would consult the type checker and larvae cannot, larvae reports the narrower case that needs no types — so a few type-dependent findings are lost by design, and no false ones are gained. That is why `larvae init` turns Luau's own lints off in `.luaurc` when it can: one reporter per finding, not two.

**Key points**

- `allow`, `info`, `warn`, `deny`: only `deny` fails the run, and `info` differs from `warn` only in how it is drawn.
- Six lints deny by default because no reading makes the finding wrong; eleven are `allow` because they are taste.
- `[lint.groups]` sets a level for a kind; `[lint.rules]` names one lint, and a name always beats its group.
- A group never wakes a lint that is `allow` on purpose.
- Luau's 28 compiler lints are all covered, at Luau's levels, so `.luaurc` should not report them too.
