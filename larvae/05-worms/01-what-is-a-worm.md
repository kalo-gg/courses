# What a worm is

*A guest larvae hands a file to — in an embedded VM, in a wasm interpreter, or as a process of its own.*

Larvae · Worms & Extensions · 10 min

---

larvae understands Luau. A project may hold files that are not Luau — markup that compiles to it, a template language, a schema — and those files still want formatting, linting and a place in the require graph. A **worm** is how larvae reaches them.

**The head of a worm's manifest**

```toml
# worm.toml — what larvae reads to know what it is dealing with
name = "luaux"
api  = 1
form = "native"        # or "luau", or "wasm"

[frontend]
claims = [".luaux"]    # the extensions larvae hands to this worm
```

A worm is a **guest**, and its manifest says which of three kinds it is. That choice is the security decision of this whole module, so it comes first. For a native worm, larvae starts the executable once, keeps it alive, and speaks to it over stdin and stdout.

| `form` | How larvae runs it | Sandbox |
| --- | --- | --- |
| `luau` | Luau source, in an embedded VM | Yes |
| `wasm` | A `wasm32` module, in the interpreter | Yes — it cannot read a file the host did not give it |
| `native` | An ordinary executable, over a pipe | **None.** It runs with your full user access |

> **`native` is fast and unsandboxed, and that is the trade**
>
> The native form has native speed and no sandbox at all: a native worm runs with everything you can do, on every build and on every keystroke in the editor. The trade is a good one for a heavy worm a project trusts — `luaux-worm` is native, and compiling markup on each keypress is exactly the workload that needs it.
>
> But it is a downloaded binary you are agreeing to run. That is why native is **not the default** and why a project opts in per worm. Read the repo before you paste the two lines.

## Three jobs, and what comes back

| Job | larvae command | The worm returns |
| --- | --- | --- |
| Compile | `larvae process`, `larvae check` | Luau source text |
| Format | `larvae fmt` | a **layout document**, not text |
| Lint | `larvae lint` | findings **without** a severity |

The last two rows are the design. A worm that returned formatted text would need its own printer, its own column width and its own idea of indentation — and a `.luaux` file would then break lines differently from the `.luau` file beside it. Instead it returns a layout document and larvae applies the project's settings.

> **Findings come back without a level**
>
> The worm reports a finding and says nothing about how serious it is; larvae stamps the level from `[lint.rules]`, applies any `-- larvae: allow(...)` the author wrote, and decides the exit code. One severity scale, one suppression vocabulary, however many worms are installed.

**The worm family**

```bash
larvae worm add luaux          # writes [worms.luaux] at its newest release; downloads nothing
larvae worm install            # downloads every worm the config names (alias: i)
larvae worm run <dir> <file>   # passes one file through a worm directory; --fmt or --lint for those paths
larvae worm info <dir>         # what a worm declares
larvae worm claims             # the extensions this project's worms claim, as JSON
```

`add` and `install` are the next lesson; `run` and `info` are the last one. There is no `larvae worm develop`.

**Key points**

- A worm claims file extensions larvae does not understand and answers for them.
- Three forms: `luau` and `wasm` are sandboxed; `native` is a process with your full access, and is opt-in per worm.
- A worm compiles to Luau text, formats to a layout document, and lints to findings without a level — so a claimed file formats and reports like Luau.
- `larvae worm add` writes, `install` downloads, `run` and `info` need no install.
