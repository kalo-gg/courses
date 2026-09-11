# Why a type checker

*A mistake caught while you type costs seconds; the same mistake found at run time costs an hour.*

Luau Intermediate · Types in Luau · 9 min

---

A wrong Luau program runs anyway. It runs right up to the line that finally cannot cope, and that line is almost never the line that was wrong. The bad value was put into a table somewhere else, passed through two functions that did not mind, and only then handed to arithmetic that did.

The type checker exists to close that distance. It reads your program without running it and complains where the value of the wrong shape was first handed over.

**A bug that runs fine until it does not**

```luau
local prices = {
	lamp = 12,
	rope = "8", -- a string, by mistake
}

local function total(price, qty)
	return price * qty
end

print(total(prices.lamp, 2))
print(total(prices.rope, 2))
print(total(prices.nail, 1))
```

Run this and you get `24`, then `16`, then a crash: *attempt to perform arithmetic (mul) on nil and number*. The second line is the interesting one — `"8" * 2` is `16`, because Luau coerces numeric strings, so the mistake produces a plausible number and hides. The crash comes from a third, unrelated typo.

Two bugs, one traceback, and the traceback points at `total` — a function with nothing wrong with it. You would spend the next twenty minutes there.

A checker reads the table, sees `rope` is a `string`, sees `total` wants a `number`, and underlines the call site. It also knows the table has no `nail` key at all. Neither report needs the program to run.

## Gradual, not all or nothing

**Three annotations that pay for themselves**

```luau
--!strict
type Prices = { [string]: number }

local prices: Prices = { lamp = 12, rope = 8 }

local function total(price: number, qty: number): number
	return price * qty
end

-- The middle of a function needs no help: these two are obvious.
local base = total(prices.lamp, 2)
local tax = base * 0.1
print(base + tax)
```

Prints `26.4`. Three annotations — the table, the two parameters and the return — and the rest is inferred. `base` and `tax` are `number` without being told.

Luau's checker is **gradual**: a file with three annotations is genuinely better than a file with none, and you never have to finish. Annotate the places where values arrive from somewhere else — a module's exported functions, a table you build by hand, a parameter — and let inference do the interior.

That is the habit this module is teaching. The next three lessons are the syntax, the three modes that decide how hard the checker looks, and how to read what it says back.

> **Where the errors appear on Kalo**
>
> The editor underlines type errors as you type — that is a language server reading your file. The **Run** button does something different: it executes your code in a Luau sandbox, and Luau erases every annotation before executing anything. So Run will happily print the output of a file the editor is covered in red about. Both halves matter, and every exercise in the first three modules is graded on both.

**Key points**

- A type checker reads the program **without running it** and reports where a value of the wrong shape crossed an annotated boundary.
- Run time finds bugs late and in the wrong file; coercion like `"8" * 2` hides them completely.
- Luau's checking is **gradual** — three annotations beat none, and inference fills in the rest.
- Types describe shape, not intent: an off-by-one is still yours to find.
- On Kalo the editor checks types and the **Run** button does not, because Luau erases annotations before executing.
