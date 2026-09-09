# Per-test hooks

*beforeEach and afterEach give every test a fresh start, nesting outermost-first and unwinding innermost-first.*

Lest · Writing Tests · 10 min

---

Most tests need the same setup: a fresh table, an open handle, a counter at zero. Doing it inside every `it` repeats the noise and — worse — invites one test to reuse what the previous one left behind. `beforeEach` runs before every test in its block and `afterEach` after every one. Both are scoped to the `describe` that registers them, or to the whole file at the top level.

**A fresh basket for every test**

```luau
describe('basket', function ()
	local basket

	beforeEach(function ()
		basket = {}                   -- every test starts from nothing
	end)

	afterEach(function ()
		basket = nil
	end)

	it('holds one item', function ()
		table.insert(basket, 'apple')
		expect(basket).toHaveLength(1)
	end)
end)
```

## Nesting

| Hook | When |
| --- | --- |
| `Lest.beforeEach(fn)` | Before every test in the block, **outermost first** |
| `Lest.afterEach(fn)` | After every test in the block, **innermost first** |

Each is scoped to the `describe` that registers it, or to the whole file at the top level.

Setup nests and teardown unwinds: the outer block prepares the ground, the inner block refines it, and each block's teardown runs while its own setup is still in effect. Registering a hook is passing a function, so `beforeEach(reset)` works as well as an inline body; assign a fresh table rather than emptying the old one, because a test that kept a reference to the old table would otherwise keep seeing it.

**See the order for yourself**

```luau
describe('outer', function ()
	beforeEach(function () print('outer before') end)
	afterEach(function () print('outer after') end)

	describe('inner', function ()
		beforeEach(function () print('inner before') end)
		afterEach(function () print('inner after') end)

		it('runs in the middle', function () print('the test') end)
	end)
end)
-- outer before, inner before, the test, inner after, outer after
```

Unprefixed output lands in the report where it ran, so a `print` is a fair way to watch the order.

> **Same-named siblings are still separate**
>
> Two sibling `describe` blocks with the same label get independent hooks and independent state — blocks are identified by position, not by name. (Their *tests* still cannot share a full name, which is the other side of that coin.)

A hook is not always the answer. When one test needs the setup, a local inside that test says where its subject came from; `beforeEach` earns its place once two tests share it, or once forgetting the reset would couple them silently. It buys that at the cost of moving the arrangement out of the test, so register it in the narrowest block that needs it.

**Key points**

- `beforeEach`/`afterEach` wrap every test in their block.
- Outermost `beforeEach` first; innermost `afterEach` first.
- A `describe` body runs once, so its locals are shared unless a hook reassigns them.
- Blocks are identified by position: same-named siblings do not share hooks.
