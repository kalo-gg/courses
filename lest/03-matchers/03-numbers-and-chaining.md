# Numbers and chaining

*Ordering matchers, toBeCloseTo and its precision, and reading several claims about one value as one chain.*

Lest · Matchers · 9 min

---

Floating point makes `toBe` the wrong tool for arithmetic: `0.1 + 0.2` is not `0.3`. The ordering matchers compare two numbers (or two strings), and `toBeCloseTo` compares within a tolerance you choose: `|value − expected| < 10^-precision / 2`, with `precision` defaulting to 2.

**Ordering and closeness**

```luau
expect(elapsed).toBeLessThan(100)
expect(score).toBeGreaterThanOrEqual(0)

-- |value - expected| < 10^-precision / 2, precision defaults to 2
expect(0.1 + 0.2).toBeCloseTo(0.3)
expect(math.pi).toBeCloseTo(3.14159, 5)
```

Ordering matchers take two numbers or two strings; a mismatch fails rather than comparing something meaningless.

> **`toBeCloseTo` and NaN**
>
> NaN is never close to anything, including itself. That is deliberate, and consistent with `toBe` — only `toEqual` makes NaN equal to NaN.

## Chaining

Every matcher returns the expectation, so several assertions about one value read as one line, in the order you wrote them:

**A range, as one line**

```luau
expect(score).toBeGreaterThanOrEqual(0).toBeLessThanOrEqual(100)
expect(name).toBeA('string').toMatch('^user_')
```

The first failing link stops the chain and reports; the links after it never run.

Ordering matchers take two numbers or two strings; anything else — a number against a numeric string, a table — is reported as a matcher misuse rather than compared meaninglessly. The next lesson is about exactly those reports.

**Key points**

- Ordering matchers need two numbers or two strings.
- `toBeCloseTo(x, precision?)` passes within `10^-precision / 2`; the default is 2.
- NaN is never close to anything.
- Every matcher returns the expectation, so one value's claims read as one chain.
