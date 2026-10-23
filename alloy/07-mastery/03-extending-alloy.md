# Docs, agents and ingots

*Read the docs from the terminal, hand them to a coding agent, and add native extensions to the compiler.*

Alloy · The toolchain and mastery · 11 min

---

A build fails with `error(4.2)`, and you're offline, or a coding agent is writing your Alloy. Neither situation needs the website. The docs ship inside the `alloy` binary, and a diagnostic's code is the book section that explains it.

## Docs in the terminal

```bash
alloy doc 4.2              # the book section behind error(4.2)
alloy doc ExhaustiveMatch  # the same, by the error's kind
alloy doc prefer_new       # a lint
alloy doc HashMap:get      # a std method
alloy doc style            # the style guide
alloy doc --json           # everything, for a tool to read
```

With no topic, `alloy doc` lists every topic. The editor shows the same code as `Alloy(4.2)`, linked to its section.

`alloy mcp` serves the same docs to a coding agent over MCP: JSON-RPC 2.0 on stdin and stdout. It offers four tools: `search_docs`, `read_doc`, `list_topics` and `style_guide`. Register it once with `claude mcp add alloy -- alloy mcp`. The AGENTS.md that `alloy init` writes holds the style guide word for word and gives the same line.

The editor runs `alloy-lsp`, a proxy in front of luau-lsp. It desugars each open buffer in memory, hands the emitted Luau to luau-lsp, and maps every answer back to your positions. It shows the lints as warnings with quick fixes, and writes the `end` of a block after Enter. It starts the `luau-lsp` on the PATH, or the one `ALLOY_LUAU_LSP` names. `alloy self install` puts `alloy` and `alloy-lsp` in `~/.alloy/bin`, and `alloy self uninstall` removes them.

## Ingots

**alloy.toml**

```toml
[ingots]
tailwind = "ingots/tailwind"                                   # a folder in the project
logger = { repo = "someone/logger-ingot" }                     # the latest release
sorter = { repo = "someone/sorter-ingot", version = "0.2.0" }  # this release, for good

[lint.rules]
"tailwind/unknown_class" = "warn"  # an ingot's lint is <ingot>/<lint>
```

There is no ingot registry. An ingot comes from a path or from a GitHub release. The key with the slash takes quotes because TOML requires them.

> **What an ingot is**
>
> An ingot is a native executable beside an `ingot.toml`. The compiler and the language server start it once and exchange length-framed JSON with it over stdin and stdout. Its hooks include `transform` (edit the source before the desugar), `output`, `lint`, `format`, `hover` and `complete`. An edit must keep the line count, because every position maps back. Write one in Rust: `alloy ingot new <name>`.

```bash
alloy ingot install          # fetch every declared ingot the project lacks
alloy ingot update           # re-resolve each unpinned ingot to its latest release
alloy ingot info ingots/tailwind
alloy ingot run ingots/tailwind src/ui.alx --lint
```

`.alloy/ingots.lock` records what was installed. A build never fetches: a missing ingot is a report that names `alloy ingot install`.

**Key points**

- `alloy doc` takes a section (`4.2`), a kind, a lint, a std name or `style`. `--json` prints everything.
- `alloy mcp` serves the docs to agents with four tools. Register it with `claude mcp add alloy -- alloy mcp`.
- `alloy-lsp` desugars each buffer in memory and proxies luau-lsp, mapping every answer back to your lines.
- An ingot comes from a path or a GitHub release, its lints are `<ingot>/<lint>`, and its edits keep the line count.
- A build never fetches. `alloy ingot install` does, and only a pinned version gives every machine the same release.
