# Equality and truthiness

*Two kinds of equal, and two values that are truthy here and falsy elsewhere.*

Lest · Matchers · 11 min

---

`toBe` is **identity** — plain Luau `==`. Two tables are `toBe` only if they are the same table. `toEqual` is **deep structural equality**: tables compare by contents, recursively, which is what you want for most table assertions.

**The difference**

```luau
expect(1 + 1).toBe(2)
expect({}).toNotBe({})                  -- different tables

expect(parse('a=1')).toEqual({ a = 1 })
expect({ 1, { 2 } }).toEqual({ 1, { 2 } })
```

> **Negations are spelled `toNot*`**
>
> There is no `.not.toBe`, because `not` is a Luau keyword and Jest's spelling simply is not available. `toNotBe`, `toNotEqual`, `toNotBeNil` and `toNotThrow` are the four that exist.

## Two surprises

**NaN, and Luau truthiness**

```luau
-- toEqual treats NaN as equal to NaN, matching Jest.
-- toBe keeps plain Luau ==, where NaN ~= NaN.
expect(0 / 0).toEqual(0 / 0)
expect(0 / 0).toNotBe(0 / 0)

-- In Luau, everything except false and nil is truthy.
expect(0).toBeTruthy()
expect('').toBeTruthy()
expect(nil).toBeFalsy()
```

`0` and `''` are **truthy** here, unlike in JavaScript. Both matchers agree that `-0 == 0`.

Both are the right tool somewhere. `toBe` is for numbers, strings, booleans and *the same table*; `toEqual` is for a table you built to compare against. Reaching for `toEqual` on two numbers is harmless; reaching for `toBe` on two tables is a test that can never pass, and the report will say `Values are not identical` while showing you two tables that look the same.

When you mean `nil` specifically rather than merely falsy, say so: `toBeNil` and `toNotBeNil` test for `nil` and nothing else.

**Key points**

- `toBe` is `==`; `toEqual` is deep structural equality.
- Negations are `toNotBe`, `toNotEqual`, `toNotBeNil`, `toNotThrow` — there is no `.not`.
- `toEqual` treats NaN as equal to NaN; `toBe` does not.
- `0` and `''` are truthy in Luau; use `toBeNil` when you mean nil.
