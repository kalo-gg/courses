# Installing Blink

*Rokit pins the compiler per project, and the version you get is the one this course uses.*

Blink · Blink Foundations · 9 min

---

A toolchain manager installs a tool per project rather than per machine, and records the version in a file you commit. That matters more than usual for Blink, because the compiler writes source code into your repository: two people on two versions produce two different `Server.luau` files from the same description, and the difference lands in the pull request rather than in the error log.

**rokit.toml**

```toml
[tools]
rojo = "rojo-rbx/rojo@7.7.0"
blink = "1Axen/blink@0.18.8"
```

One line per tool. `rokit install` reads this, so a teammate who clones the repository gets exactly these.

| Route | Command | Notes |
| --- | --- | --- |
| Rokit (recommended) | `rokit add 1Axen/blink` | `rokit update 1Axen/blink` to bump |
| GitHub Releases | download `blink-windows-x86_64.tar.gz` | one file, `blink.exe` |
| pesde (0.18.5+) | `pesde add --dev 1axen/blink` then `pesde install` | published as `1axen/blink`, a Lune binary target |
| Studio plugin | Creator Store asset `77231976488966` | the same compiler, inside Studio |

`rokit add` does two things. It appends the pin to `rokit.toml` in the current project, and it downloads that release into Rokit's own store so the alias works in your shell. Only the first of those is in the repository, and only the first one matters to anybody else. Bumping is the same move in reverse: `rokit update` rewrites the line, and the line is what you review.

## Checking it works

**terminal**

```bash
$ blink --version
Blink 0.18.8

$ blink --help
Blink 0.18.8
USAGE:
    blink.exe [CONFIG] [OPTIONS]
OPTIONS:
    -c, --compact   Compacts error output, the full message is still printed out after a compacted version.
    -h, --help      Print help information
    -q, --quiet     Silence program output
    -v, --version   Print version information
    -w, --watch     Watch [CONFIG] for changes, automatically recompile upon one occuring
    -y, --yes       Accept all prompts
```

Verbatim, misspelling included. The whole CLI fits on a screen.

Read it as one positional argument and six flags, and nothing else. `[CONFIG]` is the definition file; every flag is a switch with no value. There is no output flag, no target flag and no format flag, because where the modules go is decided inside the definition file rather than on the command line.

> **Two flags the help does not print**
>
> `src/CLI/init.luau` marks two flags `Hidden = true`, so they work but are never listed: `--ast` dumps the parsed options table and declaration list, and `-S` / `--stats` prints compile timings. Both are useful when a definition is not producing what you expected, and neither belongs in a script, since an undocumented flag is an unsupported one.

## Pinning it

**Key points**

- Rokit pins the compiler per project, and the pin is the line your teammates get.
- Blink emits source into your repository, so a version drift shows up as a diff rather than an error.
- The CLI is one positional argument and six flags; the output paths live in the definition file.
- `--ast` and `-S` are real and hidden, and `-q` silences progress output but not everything.
