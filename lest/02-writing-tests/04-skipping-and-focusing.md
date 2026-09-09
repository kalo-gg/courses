# Skipping and focusing

*Parking a test, recording one you have not written, and the modifier that silences a file.*

Lest · Writing Tests · 11 min

---

`it` and `describe` both carry modifiers, as ordinary fields on the same callable — so the destructuring line at the top of your spec keeps working.

**Skipping**

```luau
-- reported, never executed; the body is optional
it.skip('handles unicode escapes')

-- with a reason, which reporters surface
it.skip('handles surrogate pairs', function ()
	-- kept for when the encoder lands
end, 'waiting on the encoder')

-- a test you intend to write; a body here is an error
it.todo('rejects a negative quantity')
```

`describe.skip` skips a whole block, giving each test the reason `describe.skip` — the bodies are still collected, so names and nesting appear as they would have.

> **`xit` was removed in 0.6**
>
> `it.skip` replaces it with the same signature, so the migration is a rename.

Both modifiers have a block-level twin. `describe.skip` parks a whole group in one edit: the bodies are still collected, so the names and the nesting still appear in the report, and every test under it is reported skipped with the reason `describe.skip`. Nothing in the block runs, so by the rule from *Once-hooks and teardown* its `beforeAll` never runs either. `describe.only` is the same edit for focus, and it focuses every test in the block rather than one.

## Focusing

**`it.only`**

```luau
it.only('the one I am debugging', function ()
	expect(cart.total({ 3, 4 })).toBe(7)
end)
```

When any test in a spec file is focused, **only focused tests run** and every other test in that file is reported as skipped, with the reason `skipped by .only`. Focus is per spec file — each file gets its own VM on the native backend, so focusing in one never silences another. It also composes with `-t`: the filter narrows first, and focus narrows what remains.

> **Make it fatal on CI**
>
> A focused test committed by accident silently stops running everything around it, so every run reports how many tests were excluded — and `lest --forbid-only` turns that note into a failure. It exits **1**, alongside failing tests, because it is a test-suite defect rather than a tool error.

**Key points**

- `it.skip(name, fn?, reason?)` parks a test and can say why; `describe.skip` parks a block and gives each test the reason `describe.skip`.
- `it.todo(name)` records a test you have not written; a body is an error.
- `it.only` runs only focused tests in that file; the rest report `skipped by .only`.
- Focus is per file and composes with `-t`; `lest --forbid-only` makes a committed focus fail with exit 1.
