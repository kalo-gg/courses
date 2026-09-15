# Your first description

*Two output options, one event, and the compiler writes both sides for you.*

Blink · Blink Foundations · 10 min

---

A `.blink` file has two regions and one punctuation mark worth knowing. `option` lines come first and configure the compiler. Declarations come below them: types, events, functions, and the containers a later module covers. A `--` comment runs to the end of the line and is legal anywhere.

**crosswire.blink**

```blink
option ClientOutput = "./Client.luau"
option ServerOutput = "./Server.luau"

-- A percentage, bounded where it is declared rather than at each call site.
type Health = u8(0..100)

event Damage {
    From: Server,
    Type: Reliable,
    Call: SingleAsync,
    Data: Health
}
```

`ClientOutput` and `ServerOutput` are the two options the CLI insists on. Leave either out and the compiler stops with `A client output path must be defined.` Both paths are relative to the `.blink` file, not to your shell, and `.luau` is appended for you if you left it off. Give them different names: the same path twice means one module overwrites the other.

> **Kalo's editor does not need them**
>
> The exercises and the playground on this site compile the file the way the Studio plugin does, which chooses the output destination itself, so a `.blink` file here works with or without the two `option` lines. The CLI needs them; write them anyway, out of habit.

## Compiling it

**terminal**

```bash
$ blink crosswire
Blink 0.18.8
Reading source from crosswire.blink...
Parsing source into AST...
Generating output files...
Network files generated!
```

Four lines, four stages. They are the four `Message` calls in `src/CLI/Utility/Compile.luau`.

Notice that the command said `crosswire`, not `crosswire.blink`. `blink <name>` looks for `<name>`, then `<name>.txt`, then `<name>.blink` in the current directory, in that order, which is the `EXTENSIONS` list in `src/CLI/Utility/GetDefinitionFilePath.luau`. The extension is optional on the command line, and a stray `crosswire.txt` beside your definition would win.

## Your turn

**Key points**

- Options at the top, declarations below, `--` comments anywhere.
- `ClientOutput` and `ServerOutput` are required by the CLI and are relative to the definition file.
- `blink <name>` tries `<name>`, `<name>.txt` and `<name>.blink`, in that order.
- A compile prints read, parse, generate, done, and a failure names the stage it stopped in.
