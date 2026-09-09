# Installing pesde

*Download, self-install, verify and upgrade the binary, then scaffold your first manifest with pesde init.*

pesde · pesde Foundations · 9 min

---

pesde is one binary. Download the archive for your platform from the GitHub releases page, extract it, and run its `self-install` subcommand from wherever it landed — that copies it into `~/.pesde/bin` (or `$PESDE_HOME/bin`) and, on Windows, adds that directory to your `PATH`. On Linux and macOS you add the export yourself and may need `chmod +x` first. `pesde -v` is the smoke test.

Everything pesde owns lives under that one directory: the binary itself, the store every project's packages are linked out of, and the config `pesde config` reads and writes.

`PESDE_HOME` moves the lot somewhere else, which is what a CI runner sets so the store survives between jobs. Nothing else on your machine is touched, and uninstalling is deleting the directory and the `PATH` line.

**Installing**

```bash
chmod +x ~/Downloads/pesde          # Linux and macOS only
~/Downloads/pesde self-install
export PATH="$PATH:$HOME/.pesde/bin" # in your shell profile
pesde -v
```

`PESDE_HOME` overrides the whole directory; it defaults to `$HOME/.pesde`.

> **Do not install pesde with Rokit or Aftman**
>
> The docs recommend against it explicitly. Use `pesde self-upgrade` to update, and if a team needs to agree on a version, pin it with `[engines.pesde]` in the manifest rather than through a toolchain manager.

## Your first manifest

**Scaffolding**

```bash
mkdir hello-pesde && cd hello-pesde
pesde init

# what is the name of the project? you/hello_pesde
# what is the description of the project?
# who are the authors of this project?
# what is the repository URL of this project?
# what is the license of this project? MIT
# what environment are you targeting for your package? luau
# would you like to setup Roblox compatibility scripts? No
```

A name is `scope/name`, and may only contain lowercase letters, numbers and underscores. There is no version question — `init` always writes `0.1.0`.

**pesde.toml**

```toml
name = "you/hello_pesde"
version = "0.1.0"
license = "MIT"

[target]
environment = "luau"
bin = "main.luau"

[indices]
default = "https://github.com/pesde-pkg/index"
```

`init` writes the name, the hardcoded `version = "0.1.0"`, the license, `[target]` and `[indices]`. You add `bin` yourself: it names the file `pesde run` executes when you give it no argument.

**Key points**

- Download a release, run `pesde self-install`, put `~/.pesde/bin` on your `PATH`, check with `pesde -v`.
- Never through Rokit or Aftman; `pesde self-upgrade` updates it.
- `pesde init` asks six questions — name, description, authors, repository, license, environment — then one Roblox question that depends on the answer to the last: a non-Roblox target is asked whether to set up compatibility scripts, a Roblox target is taken straight to the scripts-package picker and never sees that question.
- It writes `name`, `version` (always `0.1.0` — there is no version prompt), `license`, `[target]` and `[indices]`; `bin` is yours to add, and `pesde run` executes it.
