# The require gate

*larvae check asks the questions only a whole require graph can answer, and two of them no level can silence.*

Larvae · Rewriting Requires · 10 min

---

`larvae check` asks questions that only make sense across the require graph, and writes nothing. That makes it the CI gate: every key in `[check]` is a level, and as with lints only `deny` fails the run. It also counts the **dynamic requires** it deliberately did not touch — a number worth watching, for a reason the Minify and bundle lesson makes concrete.

**The gate**

```toml
[check]
cycles = "warn"           # modules that require each other; the report names the cycle
unused_modules = "allow"  # modules nothing requires
early_require = "warn"    # roblox-instance only: client requires that do not wait
# entries = ["src/boot.luau"]  # extra entry points, never reported as unused
```

> **Two things no level can silence**
>
> Unresolvable requires and realm violations are always reported by `larvae check`, whatever the levels say. Those are not opinions about style — they are requires that cannot work.

`unused_modules` defaults to `allow` for a good reason: Scripts and LocalScripts are never counted as unused, but a module reached only by a tool, a test harness or a runtime lookup would be. `entries` is how you tell larvae about those.

> **`early_require` is warning about a default, not about you**
>
> `early_require` only applies to `roblox-instance` targets, and it warns out of the box because the default `indexing_style` is `find_first_child` — which does not wait. So on an untouched `roblox-instance` project this warning is larvae reporting its own default back at you.
>
> The fix is in Aliases and targets: set `indexing_style = "wait_for_child"` and the warnings go, or set `early_require = "allow"` if you have decided the requires are late enough to be safe.

## Strict, and what the editor sees

**Warnings that should be errors**

```toml
[requires]
strict = true     # a warn-level problem, such as a case mismatch, becomes an error
```

**The gate, and the file beside it**

```bash
larvae check                       # validates requires and syntax; writes nothing
larvae self sync-luaurc            # writes [aliases] into .luaurc as paths an editor can follow
larvae self sync-luaurc --check    # writes nothing; exits non-zero if it would change
```

`larvae check` also counts the dynamic requires it deliberately left alone: `require(someVariable)` cannot be rewritten, so it passes through as written. Watch that number — every one of them is a module the bundler cannot follow either. And `sync-luaurc` closes the loop with your editor: the aliases you wrote in `larvae.toml` become paths in `.luaurc`, so luau-lsp follows them too, with `--check` as the CI line that says the two files agree.

**Key points**

- `larvae check` validates requires and syntax across the whole graph and writes nothing: the CI gate.
- Unresolvable requires and realm violations always report; `cycles`, `unused_modules` and `early_require` take a level, and only `deny` fails.
- `entries` names the modules only a tool or a runtime lookup reaches, so they are never unused.
- `strict = true` upgrades warn-level require problems to errors; `larvae self sync-luaurc --check` keeps `.luaurc` honest.
