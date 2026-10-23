# Formatting

*Every stylua option, and the three that are not stylua's, which are on by default.*

Larvae · Formatting & Linting · 11 min

---

`[fmt]` accepts every stylua option under its own name, so the *settings* in a `stylua.toml` can be pasted in wholesale. An existing one is read first and `[fmt]` layers over it, which means a project can move settings across gradually rather than all at once.

The **defaults** are a different question, and this is the part of the migration story that surprises people. Three options carry an opinion of larvae's rather than stylua's, and all three default to `true`. Run `larvae fmt` on an untouched repo and your tables change.

**Past stylua**

```toml
[fmt]
column_width = 120
indent_type = "tabs"

# options stylua does not have — and these three are ON by default
magic_trailing_comma = true    # a trailing comma means "keep this expanded"
space_inside_braces = true     # { a } rather than {a}
trailing_comma = true          # a table that broke keeps its last comma

block_newline_gaps = "never"   # or "preserve", a blank line at a block's edge
table_newline_gaps = "preserve" # or "never": the blank lines between a table's fields
call_chains = { style = "preserve", min_calls = 3, preserve_breaks = true }
```

They are on because the Luau community already writes that way — which is a good reason, and still a change to a repo that does not.

> **The first day: `--check`, then `recommended = false`**
>
> Run `larvae fmt --check` before you run `larvae fmt`. It writes nothing and exits non-zero if a file would change, so it tells you the size of the diff without making it. (It is also the CI invocation, and `larvae fmt --stdin --stdin-filepath <path>` is how an editor asks for one file.)
>
> If the diff is all braces and trailing commas, one line under `[fmt]` makes stylua parity literally true: **`recommended = false`**. It starts from a base that changes as little as possible — those three options go back to off, and every other default was stylua's already. Then add the extras back one at a time, on purpose. `[lint] recommended = false` is the same idea for the linter.
>
> Neither key is in `larvae.example.toml` or the README — but `larvae init` writes `recommended = true` into your config, so the key is there to flip.

`magic_trailing_comma` is the one worth understanding rather than just switching. It makes the author's trailing comma meaningful: leave a comma after the last entry and the table stays expanded even when it would fit on one line — a way to say "this list is going to grow" that survives formatting. The corollary is the bit that catches people out: a hand-aligned table with a trailing comma is not collapsed, it is **exploded** to one entry per line.

## Sorting, and a refusal

**Sorting the require block**

```toml
[fmt.sort_requires]
enabled = false      # off by default; it moves lines
grouping = "flat"    # one sorted run, as stylua does
# grouping = "by-kind"  # aliases, then absolute, then relative, blank line between
```

> **It would rather decline than delete a comment**
>
> The formatter checks every comment in the source against its own output at runtime. A comment sitting somewhere the emitter has no position for would vanish — so instead of dropping it, larvae leaves the file **byte for byte as it was** and reports it: *formatting would drop the comment "…", so the file was left alone*.
>
> This is a real difference from stylua, and it is the answer when one file in a repo mysteriously refuses to format.

## Unused imports

**A default chosen for a reason**

```toml
[fmt]
unused_imports = "ignore"   # or "underscore", "remove"
```

> **Why the default is `ignore`**
>
> `require` runs a module the first time anything asks for it. A module can therefore do its work simply by being required, and deleting the line stops that work — so removing an apparently unused import can silently change behavior. A name that only a type annotation reads still counts as used.

**Key points**

- `[fmt]` takes every stylua option under its own name; an existing `stylua.toml` is read first.
- Three defaults are larvae's own — `magic_trailing_comma`, `space_inside_braces`, `trailing_comma` — and `recommended = false` turns exactly those off.
- Run `larvae fmt --check` before `larvae fmt`; it is also the CI invocation.
- A file whose formatting would drop a comment is left byte for byte and reported.
- `unused_imports` is `ignore` because a require can do work by being required.
