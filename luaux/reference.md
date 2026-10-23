# LuauX reference

*LuauX 0.2.0, released August 26, 2026. Checked on September 7, 2026.*

---

## Syntax

JSX, adapted for Luau and for Roblox class names.

### `Element`

```luau
<Frame Size={UDim2.fromScale(1, 1)}>…</Frame>
```

A Roblox class name, its attributes, and children between the tags.

### `Self-closing`

```luau
<UICorner CornerRadius={UDim.new(0, 8)} />
```

An element with no children.

### `Attributes`

```luau
Name="literal"
Attr={expression}
```

A quoted string, or any Luau expression in braces.

### `Boolean shorthand`

```luau
<TextLabel TextWrapped />
```

A bare attribute name is `TextWrapped={true}`.

### `Inferred name`

```luau
<Frame ={props.Size} ={props.Visible} />
```

When the expression already says which property it is, `={…}` says it once.

> The expression has to **be** a name — an identifier or a dotted path whose last segment is the property. `={f().Size}` or `={a or b}` is an error telling you to write the property out. It is spelled `=` rather than a bare `{Size}` because a bare hole in attribute position already means a spread.

### `Tag resolution`

```luau
<Frame />   -- class
<Card />    -- your component
<UI.Card /> -- always a component
```

A name resolves to a `luaux.toml` alias first, then a Roblox class, then anything bound in the file, and otherwise errors with a suggestion.

> A class beats a file binding, so a component named `Dialog` or `Frame` is unreachable. Only a `[elements]` alias outranks a class — and because renames are exclusive, declaring one retires the canonical spelling. Dotted names skip resolution entirely.

### `Text interpolation`

```luau
<TextLabel>Clicked {count} times</TextLabel>
```

Text mixed with expressions.

> **The one place reactivity is implicit**, because building a string from a reactive value would otherwise stringify the value itself. A *single* expression — `<TextLabel>{label}</TextLabel>` — is emitted bare in every target instead.

### `Expression child`

```luau
{items}
```

Any Luau expression in child position, handed to your library untouched.

### `Fragment`

```luau
<>…</>
```

Several siblings with no wrapper. The element backend needs `[factory] fragment` set, because a plain table is not an element.

### `Spread`

```luau
<Button {props} Color={…} />
```

Forwards a whole props table.

> Anything written **after** the spread wins, which is how a component supplies a default its caller can still override.

### `Comments`

```luau
<!-- between tags -->
{--[[ in child position ]]}
```

Markup comments do not reach the output.

> `--` does **not** start a comment inside markup, where it is ordinary text.

## Components

### `A component is a function`

```luau
local function Button(props: ButtonProps)
  return (<TextButton Text={props.Label} />)
end
```

Any function taking a single table. Under Vide and Fluid children arrive in that table under numeric keys, exactly as they do for a built-in class — so a component and an intrinsic are interchangeable at the call site. Fusion is the same arrangement but places them under the key `[factory] children` names.

### `Props are values or functions`

```luau
Label: string | vide.Source<string>
```

Neither case needs handling in the component: a backend that treats a function on a property key as reactive does the work. That is why the same component accepts a constant and a source.

> The type is a member of the module, so the prop is spelled `vide.Source<string>`; a bare `Source<string>` is unbound in a file that only requires `vide`.

### `Defaults`

```luau
props.Size or 14
<Frame Color={red} {props} />
```

Two ways to default a prop: `or` for a single value, spread order for a whole table.

> Written **before** the spread it is a default the caller overrides; **after** it, a value the component enforces. There is no third mechanism.

### `Naming a component`

```luau
<UI.Card />   vs   <Card />
```

A tag resolves to a Roblox class before it reaches anything bound in the file, so a component named after a class is unreachable.

> Keep shared components on a table: no Roblox class name contains a dot, so `<UI.Dialog />` can only be yours. `ConfirmDialog` over `Dialog` is the same habit without the table.

### `A class component`

```luau
setmetatable(Profile, {
  __call = function(class, props)
    return class.new(props):Render()
  end,
})
```

A tag compiles to a plain call, so anything callable works — including a table with `__call`.

> `__call` goes on the class's **own metatable**, not on the class. `function Profile.__call()` defines a field nothing consults, and `<Profile />` fails with *attempt to call a table value*.

### `Forwarding children`

```luau
React   {props.children}
Vide    {props}
Fluid   {props}
Fusion  {props[Children]}
```

LuauX generates the code that passes children, never the code that reads them back out.

> So this is a line to type, not a setting to choose. Porting it wrong is loud rather than subtle — React tries to render the props table itself and fails on the first render.

## Backends

A backend owns the arrangement; [factory] owns the contents.

### `backend`

```luau
backend = "table"   # or "element"
```

`table` emits `F(class)(props)` with children in the props table — Vide, Fusion and Fluid. `element` emits `F(class, props, children)` with children as a third argument — React.

> **Required in every `[factory]` block.** With no block at all LuauX targets React; the moment you write one, nothing is assumed. Otherwise a Vide project setting only `create` would inherit React's arrangement and emit the wrong shape in silence.

### `The four libraries`

```luau
Vide/Fluid  F(class)(props)          numeric keys   string key
Fusion      F(class)(props)          [Children]     [OnEvent(name)]
React       F(class, props, children) 3rd argument   [Event.Name]
```

How each library takes an arrangement, its children and its events. Two things vary between them and they are not the same kind of thing, which is why one is a backend and the other is configuration.

> The reactive value differs too: a function for Vide and Fluid, a `StateObject` for Fusion, and nothing at all for React.

### `create`

```luau
create = "create"
```

The constructor to call. Every `[factory]` name has to be in scope in each file that uses it — LuauX never emits a `require`.

### `children`

```luau
children = "Children"
```

The key children are placed under, written without brackets. Omit it for Vide and Fluid; the element backend rejects it outright.

### `event`

```luau
event = "React.Event."   # [React.Event.Activated]
event = "OnEvent"        # [OnEvent("Activated")]
```

How an event attribute is written — a trailing dot indexes, no trailing dot calls.

> Applies only to intrinsics, because that is the only place LuauX knows an attribute *is* an event — a component's props are arbitrary.

### `compute`

```luau
compute = "scope:Computed"
```

Wraps interpolated text as `E(function(use) return … end)`.

### `use`

```luau
use = "use"
```

The reader called inside `compute`'s callback.

### `fragment`

```luau
fragment = "React.Fragment"
```

Required by the element backend, where a bare table is not an element and there is nothing sensible to guess.

### `merge`

```luau
merge = "mergeProps"
```

Replaces the inlined spread helper with your library's own.

### `interpolate`

```luau
interpolate = "plain"   # or "wrap"
```

Follows the arrangement — `plain` under `element`, `wrap` under `table`. There to override a library that does not match its shape.

> Naming a `compute` wrapper while setting `interpolate = "plain"` is a contradiction the config loader rejects at load rather than resolving.

### `__luaux_read`

```luau
__luaux_read(x)   -- inlined
```

Calls `x` if it is a function and returns it otherwise; appears inside a wrapped interpolation under the table arrangement.

> One of the two helpers that make "no runtime" true: it is pasted into the files that use it, never required. Seeing it in the output tells you the text was interpolated rather than built with `..`.

### `__luaux_merge`

```luau
__luaux_merge(a, b)   -- inlined
```

The spread helper: merges two props tables, later keys winning, and concatenating the numeric keys that are children.

> Emitted wherever you wrote a spread **unless** `[factory] merge` names your library's own (Fusion ships none, so Fusion projects keep the helper).

## luaux.toml

### `[build]`

```luau
[build]
in = "src"
out = "build"
include = ["*"]
exclude = ["**/*.spec.luaux"]
clean = true
```

What is compiled and where it lands. With no `out`, each `X.luaux` is written beside itself as `X.luau` — which is how a project migrates one file at a time.

> Every non-source file is copied into the output unless `include` or `exclude` say otherwise, so Rojo can point at the build directory and still find your models and JSON. `include` defaults to `["**"]` and narrows the compile and the copy together.

### `[elements] and [properties]`

```luau
[elements]
all = "flatcase"
TextLabel = "text"

[properties.Frame]
BackgroundColor3 = "bgColor"
```

Renaming: wholesale with `all` (`PascalCase`, `camelCase`, `snake_case`, `flatcase`), one at a time, or per class. An explicit entry always beats `all`.

> Renames are **exclusive** — once `TextLabel = "text"` is declared, `<TextLabel>` is an error. These keys use **canonical Roblox spelling**, because the names are borrowed from the Roblox API; everything LuauX defines itself is snake_case.

### `[lints]`

```luau
[lints]
static_conditional_child = "warn"
```

LuauX's own lints, by level: `off`, `warn` or `error`.

> `static_conditional_child` is the compiler's only lint. It defaults to `warn` under the table backend, where such a child really is built once, and `off` under the element backend, where the component body re-runs. `allow` and `deny` are larvae's levels, for the worm's lints in `larvae.toml`.

### `Where the config is read`

```luau
cd ports/fusion && luaux build
```

One `luaux.toml` describes one build; a second target is a second build root with its own config and its own `in`/`out`.

> The compiler walks **up** from the source root — or from the working directory when no path is given — and takes the first `luaux.toml` it finds, falling back to built-in defaults if there is none. There is no `--config` flag: the CLI is `build`, `check`, `watch`, `init [--library]` and `scan`.

## Tooling

### `luaux build`

```luau
luaux build src build
```

Compiles `.luaux` to `.luau`. Point Rojo at the output directory.

### `luaux check`

```luau
luaux check src
```

Compiles without writing — the CI form.

### `luaux watch`

```luau
luaux watch
```

Rebuilds as files change, beside `rojo serve`.

### `luaux init`

```luau
luaux init --library vide
```

Scaffolds a `luaux.toml`. `--library` takes `react`, `vide`, `fluid` or `fusion` and writes the matching `[factory]` block.

### `luaux scan`

```luau
luaux scan src
```

Reports where LuauX is detected — the report you want halfway through a migration.

### `Project layout`

```luau
src/ → luaux build → build/ → Rojo $path
```

Markup is compiled, everything else is copied, and Rojo's tree points at the output directory only.

> Never edit or commit the output; `clean = true` deletes a `.luau` whose `.luaux` is gone.

### `LuauX in CI`

```luau
rokit install
luaux check src
rojo build -o place.rbxl
```

The gate: `check` compiles everything, writes nothing, and exits non-zero on the first bad class name or `error`-level lint.

> `watch` never exits and `build` writes files, so neither belongs in CI.

## Editor & the larvae worm

Lints and formatting are larvae's, through the worm, and they make a `.luaux` file behave like the `.luau` beside it. Types come from LuauX's own editor extension, the last entry.

### `luaux-worm`

```luau
[worms]
luaux = { repo = "larvae-luau/luaux-worm", version = "^" }
```

A larvae worm giving `.luaux` files a compiler, a formatter and a linter inside the larvae toolchain.

> The compile preserves line counts: line N of the output is line N of the source, so a Studio stack trace points at the line you wrote.

### `larvae worm install`

```luau
larvae worm add luaux
larvae worm install
```

`add` writes the table (the short name `luaux` expands to `larvae-luau/luaux-worm`); `install` downloads it.

> Naming a worm does not fetch it, and the editor never fetches at all — it quietly serves `.luaux` as nothing until someone runs `install`.

### `[lint.rules.luaux]`

```luau
[lint.rules.luaux]
useless_fragment = "deny"
```

Levels for the lints the worm adds — `self_closing_element` and `useless_fragment` today.

> Larvae's four levels, not the compiler's three: these are stamped by larvae, which is why the same pattern can be loud in your editor and quiet in `luaux check`. Whether the worm declares anything beyond those two could not be checked — its repository 404s — so this course names only `self_closing_element` and `useless_fragment`, and never puts `static_conditional_child` here: that one is the compiler's.

### `[fmt.luaux]`

```luau
[fmt.luaux]
attribute_per_line = true
```

Formatting options the worm adds, applied by `larvae fmt`.

> The worm returns a layout document rather than text, so your project's width and indentation still decide the rest.

### `inherit_lints / inherit`

```luau
inherit_lints = true
inherit = { lints_except = ["unused_variable"] }
```

Whether larvae's own lints run inside the files this worm claims, and which of them.

> `lints_only` and `lints_except` are exclusive.

### `Suppressing a finding`

```luau
-- larvae: allow(luaux.useless_fragment)
```

One allow comment, the same vocabulary as every other larvae lint.

> Everywhere outside `[lint.rules.luaux]` the lint carries its namespace: in a finding, in `--explain`, and in the comment.

### `What the editor gets`

```luau
larvae lsp: findings · formatting · fixes
LuauX LSP: completions · hover · types
```

`larvae lsp` hands a `.luaux` buffer to the worm for its findings, layout and fixes. The LuauX LSP extension (`RyanCundiff.luaux-lsp`) answers everything typed: it compiles the file in memory and asks a luau-lsp child about the Luau inside.

> **No diagnostics on Kalo**, on purpose: several exercises are markup fragments, and a fragment is an expression rather than a file. The editor completes class and property names; it does not underline a typo.

## The library names the examples use

None of these is LuauX's. Seven are Vide 0.4.1, the library this course's examples compile into; the last two are Fusion 0.3, which the final project ports one file to. They are the names a lesson's code calls.

### `create`

```luau
vide.create(className) -> (props) -> Instance
```

The constructor `[factory] create` names. Every `.luaux` file that uses markup must bring it into scope itself.

> LuauX emits no `require`, so the missing line is a compile error, not a runtime one.

### `source`

```luau
vide.source<T>(value: T) -> Source<T>
```

A function that returns its value when called with no arguments and sets it with one.

> The prop type the course writes is `vide.Source<T>` — the type is a member of the module, so a bare `Source<T>` is unbound; state lives in the file that assembles the screen.

### `derive`

```luau
vide.derive<T>(fn: () -> T) -> () -> T
```

A cached read-only source computed from other sources.

> What `Toggle` uses to turn a boolean into "On"/"Off".

### `show`

```luau
vide.show(source, component, fallback?)
```

Runs `component` in a stable scope while the source is truthy and destroys it when it is not.

> The answer to `static_conditional_child`: both arguments are functions, so the read happens again.

### `indexes`

```luau
vide.indexes(source, fn(value, index, present))
```

One stable scope per table index; a changed value updates that scope rather than rebuilding the row.

> `value` is a **function** — read it (`bind().Key`) inside the markup, not above it.

### `read`

```luau
vide.read(value: T | () -> T) -> T
```

Returns `value()` for a function and `value` otherwise.

> For the rare component that must inspect a prop rather than pass it on.

### `mount`

```luau
vide.mount(component, target?) -> () -> ()
```

Runs a component in a stable scope and parents what it returns.

> The last line of `main.client.luau`, and the only Vide call outside the markup files.

### `Fusion.scoped / innerScope`

```luau
Fusion.scoped(…: Constructors) -> Scope<Constructors>
Fusion.innerScope<T>(existing: Scope<T>, …) -> Scope<T & …>
```

A scope whose methods are the tables passed in, so `scope:New(…)` works; `innerScope` makes a child of one that dies with it.

> `create = "scope:New"` means the name `scope` must be in scope in each file. Take it from props with `innerScope` — a file-level `scoped(Fusion)` is one scope shared by every instance and destroyed by nothing.

### `Fusion.Children`

```luau
[Fusion.Children] = …
```

The key Fusion parents instances under, and the value of `[factory] children` in a Fusion block.

> It is why a Fusion wrapper forwards children explicitly (`{props[Children]}`): a spread carries children only where they are numeric keys.

## Ecosystem

| Package | Role | Version | Install |
| --- | --- | --- | --- |
| [luaux](https://github.com/luau-xml/luaux) | The compiler and CLI | 0.2.0 | `lpm tool add luaux` |
| [luaux-worm](https://github.com/larvae-luau/luaux-worm) | Lints and formatting for `.luaux` inside larvae | 0.2.2 | `[worms] luaux = { repo = "larvae-luau/luaux-worm", version = "^" }` |
| [LuauX LSP](https://github.com/luau-xml/lsp) | Editor extension: completions, hover and types for `.luaux` | 0.2.2 | `code --install-extension RyanCundiff.luaux-lsp` |
| [Vide](https://github.com/centau/vide) | The UI library this course's examples target | 0.4.1 | `centau/vide (wally) · build.rbxm on GitHub releases` |
| [Rojo](https://github.com/rojo-rbx/rojo) | Syncs the compiled output into Studio | 7.7.0 | `rokit add rojo-rbx/rojo` |
| [larvae](https://github.com/larvae-luau/larvae) | The toolchain that hosts the worm | 0.10.0 | `cargo install larvae larvae self install` |
| [Rokit](https://github.com/rojo-rbx/rokit) | Toolchain manager: pins luaux and rojo for a repository and for CI | 1.2.0 | `curl -sSf https://raw.githubusercontent.com/rojo-rbx/rokit/main/scripts/install.sh \| bash` |

## Out of date elsewhere

Shapes you may still meet in older tutorials:

- **Anything describing a runtime adapter.** An early proposal emitted one shape everywhere and translated per library at runtime; it was rejected because it reintroduces a runtime, and because `<Frame/>` genuinely *is* an Instance under Vide, Fusion and Fluid while being an inert description under React. No adapter can hide that difference.
- **Assuming `table` is the default backend.** With no `[factory]` block LuauX targets React — the one place it names a library. Writing a block turns every default off and makes `backend` required.
- **snake_case element or property keys.** `[elements]` and `[properties]` use Roblox's own spelling — `TextLabel`, not `text_label`.
- **Assuming every interpolation is reactive.** Only interpolated *text* is wrapped for you. An attribute takes the function you write.
- **`[worms] luaux = "larvae-luau/luaux-worm@1.0"`.** The string form of a worm entry was removed in larvae v0.5.0 and is now refused outright; a worm is a table with `repo` and `version`. It still appears in older configs and in `luaux-worm`'s own README.
- **"`--` starts a comment in markup."** It does not. Inside markup `--` is ordinary text, so `-- TODO` between a label's tags ends up on screen. The comment spellings are `<!-- … -->` between tags and `{--[[ … ]]}` in child position.
- **Lint levels spelled `allow` / `deny`.** Those are larvae's, for the worm's lints in `larvae.toml`. The compiler's own `[lints]` in `luaux.toml` take `off`, `warn` and `error`, and `static_conditional_child` — its only lint — defaults to `warn` under the table backend and `off` under the element backend.
- **`.luaux` completions and hover from `larvae-lsp`.** Before larvae 0.10 the worm's `[lsp]` hooks fed `larvae-lsp`'s analyzer. 0.10 deleted that binary and the hooks: `larvae lsp` serves the worm's findings, formatting and fixes only. Completions, hover and types come from the LuauX LSP extension (`luau-xml/lsp`), which runs luau-lsp. Older guides that say LuauX has no language server of its own are wrong too. The compiler itself is still a terminal binary that runs beside `rojo serve`.
