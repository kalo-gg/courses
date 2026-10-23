# The editor server

*Two servers per Luau file: what larvae lsp answers, the [lsp] table that is left, and where the types live now.*

Larvae · Larvae for a Team · 10 min

---

Everything so far was the CLI. In the editor, two servers share each Luau file. `larvae lsp` publishes the lint findings — a worm's findings and the cross-realm requires included — formats the file, and offers the fixes, through the same code the commands run, so the terminal and the editor cannot disagree. **luau-lsp** serves the types: hover, completion, inlay hints, type diagnostics and go to definition. larvae advertises nothing luau-lsp answers, so the two never compete.

**What is left of [lsp]**

```toml
[lsp]
enabled = true       # false answers every request with nothing: another server owns the files
claim_only = false   # true lints and formats only worm-claimed files; other tools keep plain Luau

[lsp.markup]         # one worm's editor settings, checked against the [options] it declares
strict = false
```

That is the whole table. The Larvae extension takes no server settings of its own, so a checkout behaves the same for everyone who opens it.

## Where the types went

| Before 0.10, in larvae | Now, in luau-lsp |
| --- | --- |
| `[lsp] definitions` | `luau-lsp.types.definitionFiles`, a map: `{ "@lantern": "types/lantern.d.luau" }` |
| `sourcemap_autogenerate`, `rojo_project_file` | `luau-lsp.sourcemap.autogenerate` (on by default) and `luau-lsp.sourcemap.rojoProjectFile` |
| `[lsp.inlay_hints] parameter_names` | `luau-lsp.inlayHints.parameterNames` |
| `[lsp.completion.imports] use_const` | `luau-lsp.completion.imports.useConst` |
| `larvae analyze --definitions <file>` | `luau-lsp analyze --definitions:@lantern=types/lantern.d.luau` |

Nothing about types moved into a new larvae setting. Each key went to the luau-lsp setting that already did the job, so a team that ran both servers before 0.10 deletes lines and adds none. luau-lsp also regenerates the sourcemap by itself, running `rojo sourcemap --watch`, and larvae spawns no rojo process at all. The file below is the whole editor setup in VS Code.

**.vscode/settings.json**

```json
{
  "[luau]": {
    "editor.defaultFormatter": "AndrewBordis.larvae",
    "editor.formatOnSave": true
  },
  "luau-lsp.types.definitionFiles": { "@lantern": "types/lantern.d.luau" }
}
```

An editor formats a file with one server, so name larvae for `[luau]`; luau-lsp then owns everything about types. In Zed, the same choice is `languages.Luau.formatter` naming the `larvae` language server.

> **Recognising older code**
>
> A 0.8 or 0.9 `larvae.toml` may set `analyzer`, `roblox_security_level`, `definitions` or `sourcemap` under `[lsp]`, or tables such as `[lsp.inlay_hints]` and `[lsp.fflags]`. They still load — the editor shows one warning per key, naming luau-lsp — and do nothing. Move what you need into luau-lsp's settings with the table above, and delete the rest.

> **A .luaux file gets no types**
>
> `larvae lsp` still hands a `.luaux` buffer to its worm, so markup gets its findings, its layout and its fixes from the first keystroke. But nothing types it any more: no hover, no completion, and a require of a `.luaux` module has no type in luau-lsp. In Zed, which sends a server files by language, a `.luaux` file gets no larvae diagnostics yet either.

`larvae self code` is the last editor job, and it is about `larvae.toml` itself: it wires the config's JSON schema into the editor — through the TOML extension's settings when one is installed, or a `#:schema` line at the top of the file when not — so every key and every lint name, worm lints included, completes as you type it.

**Key points**

- `larvae lsp` lints, formats and offers fixes; luau-lsp, beside it, serves every type.
- `[lsp]` keeps `enabled`, `claim_only` and `[lsp.<worm>]`; the analyzer's old keys load with a warning and do nothing.
- Definitions, the sourcemap, inlay hints and type checks in CI are luau-lsp settings and `luau-lsp analyze` now.
- Name larvae as the Luau formatter, and `larvae self code` makes `larvae.toml` itself complete as you type.
