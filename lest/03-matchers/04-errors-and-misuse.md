# Errors and matcher misuse

*Why toThrow takes a function, matching on the message, and what a misuse says about your test.*

Lest · Matchers · 11 min

---

Code that is supposed to fail needs testing too — the guard that rejects an overdraft is as much a behavior as the withdrawal. `toThrow` calls a function in protected mode and passes if it raised; given a substring, the raised message must contain it. `toNotThrow` is the other half: the call completes, or the test fails with the error it raised.

**`toThrow`**

```luau
expect(function ()
	parse('{')
end).toThrow('unexpected end of input')

expect(function ()
	parse('{}')
end).toNotThrow()
```

With no argument, `toThrow` passes if the call raises at all; with a substring, the message must contain it.

> **The value must be a function**
>
> `expect(parse('{'))` would have raised **while building the expectation**, before any matcher ran — so the test would fail with a raw error rather than a clean assertion. Passing a non-function is reported as a matcher misuse for exactly that reason.

## Matcher misuse

Handing a matcher the wrong kind of thing is reported as **misuse** rather than as a raw runtime error — a non-string to `toMatch`, a non-number to `toBeCloseTo`, something that is neither string nor table to `toContain`. The distinction matters: a misuse means your *test* is wrong, not the code under test.

**Two kinds of red**

```text
● withdraw rejects an overdraft

    Thrown message does not contain the substring
    Expected: "overdrawn"
    Received: insufficient funds

● account misuses toMatch

    Matcher misuse: toMatch expects a string value, got number
```

The first block is your code disagreeing with your claim. The second is your claim being malformed.

For a message mismatch the `Received` side is the raw message, position prefix dropped, so the comparison reads clean. A misuse block has no pair at all: there was nothing to compare. That leaves one matcher unmentioned, `toMatchSnapshot`, which is odd enough to get its own lesson — *Snapshots*, in *The Loop and CI*.

**Key points**

- `expect(function () … end).toThrow('fragment')` — always a function, optionally a substring.
- `toNotThrow` fails with the raised message.
- Misuse means the test is malformed: no Expected/Received pair, just the reason.
- The Received line for a message mismatch is the raw message.
