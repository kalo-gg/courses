# In your editor

*The larvae worm that lints and formats .luaux files, the LuauX extension that types them, and what each configures where.*

LuauX · Tooling & Adoption · 10 min

---

`.luaux` files get a formatter and a linter through **`luaux-worm`**, an extension for the larvae toolchain, and completions and hover through **LuauX LSP**, LuauX's own editor extension. The worm is one line.

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

> **Types come from the LuauX extension**
>
> Install **LuauX LSP** (`code --install-extension RyanCundiff.luaux-lsp`, also on OpenVSX) beside the luau-lsp extension. It owns `.luaux`, compiles each file in memory and asks a luau-lsp child about the Luau inside, so markup gets tag and property completions, hover, and types across requires. It reads your `luau-lsp.*` settings, so definition files are set once.

> **Recognising older code**
>
> Guides from before larvae 0.10 send `.luaux` completions and hover through `larvae-lsp`, via the worm's `[lsp]` hooks. 0.10 deleted that binary and the hooks. The worm's newest release, 0.2.2, still declares them, so larvae prints one note that they do nothing. Nothing to change in your project: install the LuauX extension for types.

## What the editor gets

Naming the worm is not fetching it: `larvae worm install` downloads what the config names, and until it runs `larvae lsp` quietly serves `.luaux` as nothing. Once it is there, `larvae lsp` hands every `.luaux` buffer to the worm, so the editor shows its findings, offers its fixes and lays the markup out with your project's width and indentation. Types are not larvae's job: since 0.10 it serves none, and the LuauX extension answers them.

Kalo's own editors split the work the same way: each `.luaux` exercise is compiled in memory and luau-lsp answers for the Luau inside, which is where its completions and hover come from. Diagnostics are off here on purpose — an exercise is often a fragment, and a fragment is an expression, not a file, so the compiler would reject perfectly good answers on their first character.

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
- `larvae lsp` gives `.luaux` the worm's findings, layout and fixes; completions, hover and types come from the LuauX LSP extension, which runs luau-lsp.
