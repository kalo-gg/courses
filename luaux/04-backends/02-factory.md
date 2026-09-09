# The [factory] block

*Five or six lines that retarget the whole project, and the rule that writing them turns every default off.*

LuauX · Backends · 12 min

---

Everything a target needs lives in one block of `luaux.toml`. Here is React's:

**The element arrangement**

```toml
[factory]
backend = "element"           # F(class, props, children)
create = "React.createElement"
fragment = "React.Fragment"   # required here; a plain table is not an element
event = "React.Event."        # trailing dot indexes: [React.Event.Activated]
```

**The one-table arrangement, with everything it can move**

```toml
[factory]
backend = "table"             # F(class)(propsAndChildren)
create = "scope:New"
children = "Children"         # [Children] = { … }; omit for Vide and Fluid
event = "OnEvent"             # no trailing dot, so it calls: [OnEvent("Activated")]
compute = "scope:Computed"    # wrapper for interpolated text
use = "use"                   # the reader inside compute's callback
merge = "mergeProps"          # replaces the inlined spread helper
```

Everything the one-table arrangement can move. Fusion uses all of these **except `merge`** — it ships no `mergeProps`, and every `[factory]` name has to be in scope, so copying this block verbatim fails on your first spread. Vide and Fluid need only `backend` and `create`.

## Reading the keys

`event` reads its own punctuation: a **trailing dot** means index (`[React.Event.Activated]`), no trailing dot means call (`[OnEvent("Activated")]`).

`interpolate` follows the arrangement — `plain` under `element`, `wrap` under `table` — and exists only to override a library that does not match its shape. It is also the one pair of keys the loader refuses to reconcile: naming a `compute` wrapper while setting `interpolate = "plain"` says both wrap and do not wrap, so the config is rejected rather than quietly resolved. Everything else in the block is a name, and the rule for every one of them is the same: it has to be in scope in each file that uses markup.

> **Writing a `[factory]` block turns every default off**
>
> With no block at all, LuauX targets **React** — the one place it names a library. The moment you write one, `backend` becomes required and nothing else is assumed. Without that rule the flip would be silent: a Vide project setting only `create` would inherit React's arrangement, pass the in-scope check because its own name really is in scope, and emit the wrong shape without a word.

> **Every name has to be in scope where it is used**
>
> LuauX never emits a `require` — a require is location-dependent, and your own import always works. So `create = "scope:New"` means each file using markup must already have a `scope` in scope. The compiler checks this rather than letting it fail at runtime.

> **You do not have to write it by hand**
>
> `luaux init --library vide` scaffolds the block for you, and takes `react`, `vide`, `fluid` or `fusion`. Presets are scaffolding, not a config key — so a library that is not on that list needs no compiler change, only a block describing its shape.

**Key points**

- `backend` is required in every block; with no block at all, LuauX targets React.
- `create` names the constructor; `children`, `event`, `compute`, `use`, `merge`, `fragment`, `interpolate` describe the table's contents.
- A trailing dot on `event` indexes; no dot calls.
- Every `[factory]` name has to be in scope where it is used — LuauX emits no `require`.
- `luaux init --library` scaffolds the block; presets are scaffolding, not a config key.
