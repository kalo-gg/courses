# Reading the errors

*Eight error codes, one exit code, and the exit code is always zero.*

Blink · Blink in a Real Project · 10 min

---

Blink's diagnostics are good ones: a code, a one-line message, the offending source line and a caret under the exact span that caused it. Learning the eight codes is faster than reading the messages, because the code tells you which half of the compiler complained: `E2###` came from the parser, `E3###` from the analyser that runs after it.

| Code | Message | Cause |
| --- | --- | --- |
| `E2002` | Unexpected token | a syntax slip: `export struct X = {` and a quoted `as "Alias"` are both this |
| `E2003` | Unknown option "X" | a misspelled `option`, or a lower-case field *value* |
| `E2005` | Field "From" is missing | an event without all three required fields |
| `E3005` | Reserved identifier | you named something `StepReplication` |
| `E3007` | Unknown reference | a type that does not exist: `u64`, `String`, a typo |
| `E3009` | Range outside bounds | `u8(0..300)` |
| `E3010` | Duplicate declaration | two declarations with one name; both spans are shown |
| `E3014` | Unknown require | an `import` pointing at a file that is not there |

**net.blink**

```blink
option ClientOutput = "./Client.luau"
option ServerOutput = "./Server.luau"
type Foo = u8
type Foo = u16
```

**terminal**

```luau
[E3010] Error: Duplicate declaration
    ╭─[net.blink:1:5]
    │
003 │ type Foo = u8
    ┆      ─┬─
    ┆       │
    ┆       ╰── Previously declared here
004 │ type Foo = u16
    ┆      ─┬─
    ┆       │
    ┆       ╰── Duplicate declared here
    │
────╯
```

Read it from the bottom up. The carets are the spans, and a duplicate shows you both of them with different labels, so you never have to go hunting for the first declaration. The `[net.blink:1:5]` after the file name is the window the compiler printed, not the position of the error: the same mistake in a three-line file says `1:4`.

> **Every one of these exits 0**
>
> A missing output path, an `[E3007]`, an unknown casing and a clean compile all return exit code 0. A CI step that trusts `blink net` to fail will pass on a broken description and ship yesterday's generated module, silently, for as long as nobody notices. The fix is a job that reads the output, and it is the next module's first lesson.

## The two that are not diagnostics

Two failures come out as bare Luau errors with a stack trace instead: a bad `Casing` value, and a `Future` or `Promise` yield with no library option. You get no code and no caret, so recognise them by the `[Stack Begin]` block. The casing one even carries upstream's typo, which makes it easy to search for: `Unknown casing "camel", expectd one of "Pascal" or "Camel" or "Snake"`. Note what it is telling you as well as how: casing values are capitalised.

## Your turn

**Key points**

- `E2###` is the parser, `E3###` the analyser; eight codes cover everything this course meets.
- The caret marks the span, and a duplicate declaration shows you both spans at once.
- The `file:N:M` trailer is the printed source window, not the error's position.
- A bad `Casing` and a library-less `Future` are bare Luau errors: look for `[Stack Begin]`.
- Every failure exits 0, so anything automated must read the output.
