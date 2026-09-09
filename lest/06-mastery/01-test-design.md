# What a test should say

*One behavior per test, a name that reads as a sentence, and asserting on the result rather than the route.*

Lest · Suite Design & Capstone · 10 min

---

A test is a claim about behavior, and its full name is the sentence you will read in a red report at two in the morning. `it('works')` tells you nothing; `it('rejects a discount over 100')` tells you what broke. One behavior per `it` keeps the report specific: three assertions about three behaviors in one test stop at the first failure and hide the other two.

**Before and after**

```luau
-- before: one vague test, three behaviors
it('works', function ()
	expect(cart.total({})).toBe(0)
	expect(cart.discount(200, 25)).toBe(150)
	expect(cart.format(7)).toBe('$7.00')
end)

-- after: three claims, three names
it('is 0 for an empty list', function () expect(cart.total({})).toBe(0) end)
it('takes a percentage off', function () expect(cart.discount(200, 25)).toBe(150) end)
it('formats to two decimals', function () expect(cart.format(7)).toBe('$7.00') end)
```

## Assert the result, not the route

Prefer one `toEqual` against the whole value you expect over a field-by-field walk: it fails with both tables in full, and it documents the shape. Prefer asserting what a function *returned* or *changed* over asserting which helpers it called — a test that knows the route breaks on every refactor and passes on every bug that keeps the route. And keep arrange, act and assert visible: set up, call once, check.

**Arrange, act, assert**

```luau
it('builds a receipt for two lines', function ()
	-- arrange
	local co = checkout.new({ clock = function () return 0 end, stock = quietStock })
	-- act
	local receipt = co.place(LINES)
	-- assert: the whole value, once
	expect(receipt).toEqual({ lines = 2, subtotal = 180, total = 180, display = '$180.00', placedAt = 0 })
end)
```

When this fails you see the receipt you got and the receipt you meant, side by side.

Names are also what `-t` filters on, so a consistent vocabulary — `rejects`, `returns`, `is`, `keeps` — turns `lest -t rejects` into a meaningful slice of the suite. Put the condition in the name, not in a comment: the report prints names, never comments.

**Key points**

- A test name is a claim with a subject and a verb; the report prints names.
- One behavior per `it`, so a red report lists everything that broke.
- Assert the whole value with `toEqual`; assert results, not routes.
- Arrange, act, assert — visibly.
