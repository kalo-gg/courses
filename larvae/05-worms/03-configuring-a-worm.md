# Configuring a worm

*Everything a worm adds lives under its key: lints, formatter options, settings, and its share of larvae's rules.*

Larvae · Worms & Extensions · 10 min

---

Once a worm is installed it brings things with it: lints of its own, formatter options of its own, settings only it reads. larvae files all of them under the worm's key, so a worm can never collide with a builtin name — and so a project configures a worm's lint exactly the way it configures `shadowing`.

**Configuring what a worm added**

```toml
[lint.rules.luaux]
self_closing_element = "deny"
useless_fragment = "allow"

[fmt.luaux]
attribute_per_line = true
self_closing_space = true
```

The names are the worm's own, declared in its `worm.toml`. larvae files them under the worm's key for you.

> **Everywhere else, the name is dotted**
>
> `[lint.rules.luaux]` is the only place the lint is written bare. In a finding, in `larvae lint --explain`, and in an allow comment, larvae joins the namespace to the name with a dot — so the lint you configured above is **`luaux.self_closing_element`** everywhere you meet it:
>
> `-- larvae: allow(luaux.useless_fragment)`
>
> Without that, a learner who sees `luaux.useless_fragment` in the output has no way to look it up or silence it.

## Which of larvae's own rules run inside a worm's files

**`inherit_lints`, and taking one back out**

```toml
[worms.luaux]
repo = "larvae-luau/luaux-worm"
version = "^"
inherit_lints = true

# keep everything except one
inherit = { lints_except = ["unused_variable"] }

# or the other way round: only these
# inherit = { lints_only = ["undefined_variable"] }

# and the same for formatter options
# inherit = { fmt_except = ["magic_trailing_comma"] }
```

`inherit_lints = true` gets you everything, because that is the answer almost every time. `lints_only` and `lints_except` are exclusive — a list of what to keep and a list of what to drop cannot both be the answer, and larvae refuses a config that gives both. An option named in `fmt_except` keeps its own default inside that worm's files.

`inherit_lints = true` is the answer almost every time: a project-wide rule should hold inside markup too. The exceptions are the lints that misread markup — a variable used only by an attribute can look unused — and those are what `lints_except` is for. `fmt_except` is the same idea for formatter options: an option named there keeps its own default inside the worm's files.

The long form of a worm's table carries the rest. `run_order` says whether the worm's rules run before or after larvae's own, or in a numbered slot; `rules` switches the worm's own rules on by their names; and `[worms.<name>.config]` is handed to the worm untouched — larvae never reads inside it, which is why it is a table of its own.

**The long form**

```toml
[worms.markup]
repo = "someone/markup-worm"
version = "2.1.0"
run_order = "before"        # or "after", or a slot number
rules = { tidy = true }     # this worm's rules, by its own names

[worms.markup.config]       # the worm's own settings, handed over untouched
pretty = true
indent = 2
```

**Key points**

- A worm's lints go under `[lint.rules.<key>]`, its formatter options under `[fmt.<key>]`, its own settings under `[worms.<key>.config]`.
- Everywhere else the lint is `<key>.<name>` — in a finding, in `--explain`, in an allow comment.
- `inherit_lints = true`, then `lints_except` or `lints_only` (never both) and `fmt_except`.
- `run_order` places the worm's rules among larvae's; `rules` switches its own on by name.
