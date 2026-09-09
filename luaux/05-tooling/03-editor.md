# In your editor

*The larvae worm that gives .luaux files a formatter, a linter, completions and hover, and what it configures where.*

LuauX · Tooling & Adoption · 10 min

---

`.luaux` files get a formatter, a linter and completions and hover through **`luaux-worm`**, an extension for the larvae toolchain. Installing it is one line.

**larvae.toml**

```toml
[worms]
luaux = { repo = "larvae-luau/luaux-worm", version = "^" }

[lint.rules.luaux]
useless_fragment = "deny"

[fmt.luaux]
attribute_per_line = true
```

The key is the repository name with `-worm` removed. Everything the worm adds is namespaced under it — and the lints levelled here are the worm's own, never the compiler's `static_conditional_child`.

> **Why a worm rather than a plugin**
>
> A worm returns a **layout document** rather than formatted text, and findings **without** a severity, so larvae applies your project's width, indentation, lint levels and `-- larvae: allow(…)` suppressions. A `.luaux` file therefore formats and reports exactly like the `.luau` file beside it. Kalo's **larvae** course covers the mechanism in full.

> **Editor support comes from larvae**
>
> Kalo's larvae course explains how worms work; here it is enough to know one line in `[worms]` gives your editor everything it needs.

## What the editor gets

Naming the worm is not fetching it: `larvae worm install` downloads what the config names, and until it runs the editor quietly serves `.luaux` as nothing. Once it is there, `larvae-lsp` routes every `.luaux` file to the worm, which compiles it before the analyzer sees it — so you get completions and hover for the Luau in and around the markup, and `larvae fmt` lays the markup out with your project's width and indentation.

Kalo's own editors work the same way: the `.luaux` exercises on this site are served by the same worm, which is where their completions and hover come from. Diagnostics are off here on purpose — an exercise is often a fragment, and a fragment is an expression, not a file, so the compiler would reject perfectly good answers on their first character.

**A finding, and how to silence one**

```luaux
-- larvae: allow(luaux.useless_fragment)
local function Solo()
	return (<><TextLabel Text="alone" /></>)
end
```

Everywhere outside `[lint.rules.luaux]` the lint carries its namespace: `luaux.useless_fragment` in a finding, in `--explain`, and in an allow comment.

**Key points**

- `[worms.luaux]` (or the inline `[worms] luaux = { repo, version }`) names the worm; `larvae worm install` fetches it.
- The worm's lints live under `[lint.rules.luaux]`, its formatting under `[fmt.luaux]`; elsewhere the name is `luaux.<lint>`.
- It returns a layout document and unsevered findings, so `.luaux` formats and reports like `.luau`.
- The editor gets completions and hover for `.luaux` through `larvae-lsp` — including on Kalo.
