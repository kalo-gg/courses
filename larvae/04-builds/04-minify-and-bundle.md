# Minify and bundle

*Shrink the release build with the dense generator, and collapse the project into one file that loads on demand.*

Larvae · Builds & Profiles · 12 min

---

Two ways to ship less. Minifying keeps the file tree and makes each file smaller; bundling keeps the code and makes the tree one file. Both are settings on a build rather than changes to the source, and both are usually release-profile matters — a debug build wants its line numbers and its modules where you can find them.

**`[minify]`, the table behind `generator = "dense"`**

```toml
[profile.release.process]
generator = "dense"

[profile.release.minify]
column_span = 120       # break near this column, so line numbers still mean something
rename_variables = true # the [rules] rule of the same name, reachable from here
obfuscate = false
```

`dense` re-emits the output tokens with the least whitespace that still lexes the same. `[minify]` is inert under any other generator — except `obfuscate`, which turns the dense generator on by itself.

> **What `obfuscate` can and cannot do**
>
> Roblox removed `loadstring`, so what ships has to be Luau the compiler reads: no packer can hide it behind a decoder. What is left is what a reader actually uses — the names and the strings. So `obfuscate` drops every type annotation, gives every local a `_0x` name, writes every string literal as the `\xNN` form of its own bytes, and puts the file on one line.
>
> That last part is why `column_span` is an *option* rather than a number with a default: absent is not the same as 120. `obfuscate` reads an absent `column_span` as "this project does not care about line numbers" and goes unlimited, and a project that wrote a number keeps it.

`rename_variables` is reachable from two places on purpose: it is a `[rules]` rule, and `[minify]` exposes it so a release profile can shorten names without editing `[rules]`. `column_span` is the one to keep even when minifying: a crash report from a one-line file says nothing about where it happened.

## Bundling

**One file out**

```toml
[bundle]
entry = "src/init.luau"
output = "bundle.luau"
tree_shake = true    # drop the modules the entry never reaches
module_ids = "paths"   # or "opaque": what the registry calls each module
```

`larvae bundle --entry <file> --out <file>` overrides the table for one run. Modules in the bundle load on demand, so it cannot move a side effect.

> **Why `tree_shake` is a switch at all**
>
> The bundle is not a listing of the input directory — it is a **walk out from the entry**, following requires. The walk *is* the dead-code elimination, which is why there is no separate pass.
>
> The cost is dynamic requires. The walk cannot follow `require(someVariable)`, so a module reached only that way never enters the bundle and the call fails at run time. Turn `tree_shake` off and every module of the project goes in, unreachable ones included — which is exactly what a project with a runtime lookup keeps it off for.
>
> larvae does report a dynamic require inside a bundled module, so the problem does not wait for run time to find you. That is the same count `larvae check` prints.

> **Bundling cannot move a side effect**
>
> Modules in a bundle load on demand, exactly as `require` loads them normally. So a module whose work happens simply by being required still runs at the same moment relative to everything else — bundling changes how the code is delivered, not when it executes.

Which does Lantern need? The game ships through Rojo as a tree, so it minifies in the release profile and never bundles. Its Studio plugin is the opposite: one file is the format a plugin is installed as, so it bundles — tree-shaken, since nothing in it requires by variable.

**Key points**

- `[minify]` tunes `generator = "dense"` and is inert otherwise — except `obfuscate`, which switches dense on by itself.
- Keep `column_span` so a crash report still says something; `obfuscate` hides names and strings and nothing more, because what ships must still be Luau.
- `[bundle]` walks out from `entry` following requires — the walk is the tree-shaking, and a dynamic require is the one thing it cannot follow.
- Bundling changes how code is delivered, not when it runs: modules still load on demand.
