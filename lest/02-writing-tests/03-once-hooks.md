# Once-hooks and teardown

*beforeAll and afterAll wrap a block once, run LIFO on the way out, and decide what happens when setup fails.*

Lest · Writing Tests · 10 min

---

Some setup is too expensive to repeat: opening a connection, reading a fixture file, building a large table. `beforeAll` runs once before the first test in its block and `afterAll` once after the last — and only if something in the block will actually run, so a block whose tests are all skipped sets nothing up and tears nothing down.

**Setup and teardown**

```luau
describe('database', function ()
	local db

	beforeAll(function ()
		db = openInMemory()           -- once for the block
	end)

	afterAll(function ()
		db:close()                    -- once, after the last test
	end)

	it('starts empty', function ()
		expect(db:count()).toBe(0)
	end)
end)
```

## When setup fails

> **A failing `beforeAll` fails the whole block**
>
> There is no point running tests against setup that did not happen, so Lest says exactly that rather than reporting a cascade of unrelated assertion failures. Descendant blocks of a failed `beforeAll` do not run their own `beforeAll` either. Every test in the block is reported `✗` with `A beforeAll hook failed: <message>`.

Teardown is more forgiving than setup. A block that was *entered* runs its `afterAll` hooks even when its own `beforeAll` failed, because partial setup still has to be undone; only an ancestor's failure keeps the once-hooks from running at all. And a failing `afterAll` is not swallowed: it is reported as a failure named `(afterAll)` under the block, since teardown that breaks leaves state behind for whatever runs next.

**What the report shows**

```text
  broken setup
    ✗ never runs
  broken teardown
    ✓ passes
    ✗ (afterAll)

● broken setup never runs

    A beforeAll hook failed: no database
```

Blocks are identified by position, not by name: two sibling `describe('db')` blocks get independent once-hooks and independent state. Their *tests* still cannot share a full name, which is the other side of that coin.

**Key points**

- `beforeAll`/`afterAll` run once per block, and only if something in it runs.
- Several `afterAll`s run in reverse order: teardown is LIFO.
- A failing `beforeAll` fails every test in the block; descendants skip their own `beforeAll`.
- An entered block still runs `afterAll` after a failed `beforeAll`; a failing `afterAll` is reported as `(afterAll)`.
