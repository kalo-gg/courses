# Fakes you write yourself

*Lest ships no mocking API: inject the clock, the stock and the mailer, and hand a spec a fake it can inspect.*

Lest · Suite Design & Capstone · 11 min

---

The no-emulation position has a corollary for your own code: Lest ships **no mocking API** — no `fn()`, no spies, no module patching. What it expects instead is that the code under test takes its collaborators as arguments, so a spec can pass a version it controls. A **fake** is a working stand-in (`clock = function () return 1700000000 end`); a **recording fake** also remembers what it was asked, in a plain table — which is all a spy ever was.

**Inject the boundary, then fake it**

```luau
-- checkout.luau takes its collaborators; nothing inside reaches for os.time or a service
function checkout.new(deps: { clock: () -> number, stock: { reserve: (string, number) -> () } })

-- checkout.spec.luau hands in what it controls
local calls = {}
local co = checkout.new({
	clock = function () return 1700000000 end,
	stock = { reserve = function (name, quantity) table.insert(calls, { name, quantity }) end },
})

co.place(LINES)
expect(calls).toEqual({ { 'widget', 2 }, { 'gadget', 1 } })
```

## Where to draw the line

Fake the things that make a test slow, flaky or impossible on native — time, randomness, the network, storage, the engine — and only at a boundary you own. Fake a function the code *calls*, never the function you are *testing*. If the boundary is the engine itself, the test belongs in an engine suite, not behind a fake.

**A recording fake as a helper**

```luau
local function recorder()
	local calls = {}
	return calls, function (...)
		table.insert(calls, { ... })
	end
end

local sent, send = recorder()
notify({ send = send }, { email = 'ada@example.com' })
expect(sent).toEqual({ { 'ada@example.com', 'Welcome' } })
```

Eight lines, no library. The assertion is the interesting part: what was the collaborator asked to do?

The sandbox works the same way: every exercise's subject arrives as a local defined for you, a stand-in for the disk the browser does not have. When you see a test that is hard to write because the code reaches for something global, that is the code asking for an argument.

**Key points**

- Lest has no mocking API; code takes its collaborators as arguments.
- A fake stands in; a recording fake also remembers, in a table you assert on with `toEqual`.
- Fake time, randomness, network and storage at a boundary you own — never the thing under test, never the engine.
- Engine behavior is tested in an engine suite.
