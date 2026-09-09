# describe and it

*Grouping tests under a label, how a full name is built, and the phase each body runs in.*

Lest · Writing Tests · 11 min

---

`it(name, fn)` registers one test. `describe(name, body)` groups tests under a label — and its body runs **immediately, at collection time**. Its only job is to register things.

**Nesting**

```luau
describe('parser', function ()
	describe('errors', function ()
		it('reports the line number', function ()
			-- full name: "parser errors reports the line number"
		end)
	end)
end)
```

> **The full name is joined by single spaces**
>
> That is what `-t` filters against. The pretty reporter shows describe levels as indented headers, and the ` › ` glyph appears only in the "Slowest Tests" block and the JUnit `classname` — both are display decoration, so filtering on `›` matches nothing.

> **Two tests may never share a full name**
>
> Including tests under same-named sibling `describe` blocks, since their labels join identically. A duplicate is an error **at collection time**, because two tests with one name would collide on snapshot keys and be indistinguishable to reports and to `-t`.

The reason duplicates are refused rather than merged is everything downstream of the name. A snapshot key is built from it, and `-t` selects by it, so two tests called the same thing would share one stored value and could never be run apart. Lest raises at collection time and prints the name both claimed, which is usually enough to see which one you meant to rename.

## Keep bodies to registration

Because a `describe` body runs during collection, an error thrown there fails collection — Lest re-raises it as `error while collecting describe('name')`. Your `describe` calls sit at the top level of a module, so that propagates out of *loading the spec*: the whole file fails to load and nothing in it runs, including the blocks that collected cleanly. Put real work inside an `it` or a lifecycle hook. Calling `describe`, `it` or a hook *after* the run has started, from inside a test body, is a hard error rather than a registration that silently vanishes.

**Work belongs in a test or a hook**

```luau
describe('parser', function ()
	local fixtures = loadFixtures()      -- runs at collection: if it throws, the file fails to load

	it('parses every fixture', function ()
		-- work belongs here, or in a hook
	end)
end)
```

Move `loadFixtures()` into a `beforeAll` and the same failure becomes one block's failure, reported as such, instead of a spec that never loaded.

**Key points**

- `describe` groups and its body runs at collection time; `it` registers and its body runs later.
- The full name is the describe path plus the test name, joined by single spaces — it is what `-t` sees.
- Two tests never share a full name; that is a collection-time error.
- An error in a `describe` body fails the whole file; put work in tests and hooks.
