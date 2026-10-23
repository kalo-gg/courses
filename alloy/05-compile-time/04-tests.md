# Inline tests

*`@test` functions beside the code, what each artifact does with them, and `alloy test` writing lest specs.*

Alloy · Compile-time Alloy · 11 min

---

A test in its own file has to reach the code it tests. So the helper you wanted private gets exported just for the spec, and the spec drifts as the module changes. Alloy puts the test beside the code: a function marked `@test`, in the same module, with the same locals in reach.

## @test beside the code

**clamp.aly**

```alloy
export function clamp01(x: number) -> number
  return math.clamp(x, 0, 1)
end

@test
function clamps_high()
  $assert_eq(clamp01(2), 1)
end
```

**The ship artifact**

```luau
local __alloy = require("@alloy") local function clamp01(x: number): number
  return math.clamp(x, 0, 1)
end

                            
                                                                   
                                             return { clamp01 = clamp01 }
```

The test's lines 5 to 8 go blank, so the game never carries it and no later line moves. `export` adds the module's table to the end of the last line.

`@test` on a `namespace` makes each public function in it a test named `Suite.name`, and a `private` member is a helper. Inside a test, `$assert` and `$assert_eq` raise with your source text. `$expect(hp):toBe(100)` gives lest's matchers, but only inside a `@test` (anywhere else is a `TestError`) and only with `[test] lest = true`, the default.

## alloy test

```bash
alloy test                 # build, then write the specs
alloy test --run           # and run lest on them
alloy test --filter hit    # run the tests whose name holds "hit"
alloy test --coverage      # run with line coverage
alloy test --check         # write nothing; fail if a spec would change
alloy test --watch         # write again after every change
```

`alloy test` builds the project, then writes one lest spec per source that holds a test: `src/inventory.aly` becomes `tests/inventory.spec.luau`, with the tests and only the top-level code they reach. The first run also writes `lest.toml` and adds the `lest = ".lest/core"` alias to your Luau config. `--filter` and `--coverage` run lest too, as `--run` does.

> **In this course's sandbox**
>
> Exercises here run your tests from the check artifact, without lest, so they use `$assert` and `$assert_eq`, not `$expect`. Code that should exist only in a test run can sit under `@cfg(test)`, which is true only then.

**Key points**

- `@test` marks a function, or every public function of a namespace, as a test beside the code.
- The ship artifact blanks tests line for line; the check artifact keeps them and registers each one.
- `$assert` and `$assert_eq` work in any test; `$expect` needs a `@test` and `[test] lest = true`.
- `alloy test` writes `tests/<path>.spec.luau`; `--run`, `--filter` and `--coverage` run lest on them.
