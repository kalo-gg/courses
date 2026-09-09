# Flag comments

*Talking to the tool from inside the file, and why the vocabulary is deliberately small.*

Larvae · Formatting & Linting · 9 min

---

A level is a project-wide decision. Most exceptions are not: one unused local kept for a debugger, one hand-aligned table, one require that reaches across a realm on purpose. Flag comments are how a file tells larvae about its exception without changing the config for everyone — and the realm allow from Rewriting Requires is one of them.

**Allowing one line**

```luau
local unused = 1 -- larvae: allow(unused_variable)

-- more than one, comma separated
local x = 1 -- larvae: allow(unused_variable, shadowing)
```

**selene's spelling works too.** A project switching over already has these comments in many files, and larvae reads `-- selene: allow(...)` rather than making anyone rewrite them by hand.

## Switching a tool off over a span

**fmt off, fmt on**

```luau
-- larvae: fmt off
local matrix = {
	1, 0, 0,
	0, 1, 0,
}
-- larvae: fmt on
```

`lint` reads the same way. stylua's `ignore start` and `ignore end` map onto these.

| Written | Holds for |
| --- | --- |
| `-- larvae: fmt off` at the top, with no `on` | The whole file |
| `-- larvae: fmt off` … `-- larvae: fmt on` | The region between the two |
| `-- larvae: fmt off(5)` | The five lines below the comment |
| `-- larvae: lint off` | The same three shapes, for the linter |

> **Why `-- larvae: this is load bearing` is not a flag**
>
> Recognition is narrow on purpose. If larvae treated any comment beginning `-- larvae:` as an instruction, `larvae process` — which strips tool instructions from the build — would delete that note without saying so. A comment it does not recognize stays a note for the next reader.

That single design decision explains why flag comments have one vocabulary read by two things: the linter reads them to learn what the author already accepts, and `larvae process` reads them to know which comments are instructions rather than prose. Keeping the recognition in one place stops the two from drifting apart.

**Key points**

- `-- larvae: allow(name, name)` on the line it applies to; `-- selene: allow(...)` is read too.
- `fmt off` / `lint off`: whole file with no `on`, a region between two markers, or a count with `off(n)`.
- Recognition is narrow so that `larvae process`, which strips instructions, never deletes a note.
- One vocabulary, two readers: the linter learns what the author accepts, the processor learns which comments are instructions.
