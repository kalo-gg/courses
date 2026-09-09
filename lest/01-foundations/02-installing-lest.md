# Installing and scaffolding

*One binary, then lest init writes a config, an example spec and an alias, and what not to commit.*

Lest · Lest Foundations · 9 min

---

Lest ships as one static binary, so installing it is one line — and because the framework lives inside it, there is nothing to add to `wally.toml` or `pesde.toml`. Use whatever your project already pins its tools with; `lest self install` is the route for a build you made yourself, and `lest --version` tells you which one you ended up with.

**Installing**

```bash
lpm tool add lest && lpm i    # LPM pins the version per project
rokit add lest-luau/lest      # or rokit

# or from source, which is also how you get an unreleased build
git clone https://github.com/lest-luau/lest
cd lest
cargo build --release
./target/release/lest self install

lest --version                # check what you ended up with
```

`lest self install` copies the binary into `~/.lest/bin` and puts it on your `PATH`. `lest self uninstall` reverses both.

## lest init

When you do want a config, `lest init` writes one. It detects what it can — a Rojo project file, `lune` or `lute` on your `PATH`, existing spec files — and asks only about what it cannot infer. It writes a commented `lest.toml`, an example spec and — if you accept — a `.luaurc` alias so specs can `require('@lest')`. It also offers to add `/.lest` to your `.gitignore`.

**What init produces**

```bash
lest init          # prompts
lest init --yes    # take every default
```

`--yes` takes every default, which is what a script wants.

Accept the `.luaurc` alias when it is offered: it is what makes `require('@lest')` resolve for Luau LSP as well as for the runner, so a spec type-checks in the editor instead of showing an unresolved module. A project already using `.config.luau` never gains one — the Luau config RFC makes the pair an error — and `init` prints the snippet for you to paste instead.

> **`.lest/` is where the framework lands**
>
> Remember the headline claim — the framework is compiled into the binary, so runner and framework can never disagree. `.lest/core` is the mechanism: the framework is **written out of the binary on demand** into that scratch directory, alongside the generated harness scripts the Lune and Lute backends run. It is entirely reproducible — delete it and the next run puts it back — so **don't commit it, and don't edit anything in it**.

> **Re-running init is safe**
>
> An existing `lest.toml` prompts before being overwritten, and an alias already bound to `lest` in `.luaurc` is left alone. `.luaurc` is only rewritten when it parses as plain JSON with no comments; otherwise init prints the snippet for you to paste — and a project using `.config.luau` never gains a `.luaurc` at all, since the Luau config RFC makes the pair an error. That prompt is the interactive path. `lest init --yes` answers every prompt with its default and asks nothing, so an existing `lest.toml` is not overwritten either — `init` refuses and exits **2**, a tool error rather than a silent replacement of a file you meant to keep.

**Key points**

- One binary, installed with the tool you already use; `lest self install` for your own build.
- `lest init` writes a config, an example spec and the `@lest` alias, and only asks what it cannot detect.
- `.lest/` is written out of the binary on demand: never commit it, never edit it.
- Snapshots are committed; the scratch directory is not.
