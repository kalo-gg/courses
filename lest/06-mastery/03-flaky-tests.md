# Flaky tests

*Where nondeterminism creeps in, and how to quarantine a test honestly when you cannot fix it today.*

Lest · Suite Design & Capstone · 10 min

---

A flaky test passes and fails without a code change. It is worse than a failing test: it trains everyone to re-run, and a re-run is how a real regression walks in. Nearly every flake has one of four causes: **order** (a test that depends on what another left behind — the lune module cache is the classic), **time** (`os.clock`, `os.time`, a `task.wait`), **randomness** (`math.random` with a seed the test does not control), or a **focus** modifier that quietly changed what ran.

**Randomness as an argument**

```luau
-- the code takes its randomness as an argument
local function pickWinner(names, random)
	return names[random(1, #names)]
end

-- the spec hands it a random it controls
it('picks by index', function ()
	expect(pickWinner({ 'ada', 'bob', 'cy' }, function () return 2 end)).toBe('bob')
end)
```

## Diagnose, then quarantine

| Symptom | Likely cause | First move |
| --- | --- | --- |
| Passes alone, fails in the run | Order | Find the shared state; reset it in a `beforeEach` |
| Fails on a slow CI runner or around midnight | Time | Inject the clock; raise `timeout_ms` only for a suite that is genuinely slow |
| Fails one run in ten | Randomness | Inject the random or seed it in the test |
| The number of tests changed between runs | Focus | `lest --forbid-only` |

Time is the cause that hides longest, because a clock-dependent test is usually right for months before a date rolls over or a CI runner is slow enough to matter. The move is the same as for randomness: the clock becomes an argument, and the spec passes one that never moves. Reaching for a longer `timeout_ms` first is how a flake becomes a slow flake.

**The clock, as an argument**

```luau
-- before: the code reads the clock, so the test can only hope
local function isExpired(token)
	return token.expiresAt < os.time()
end

-- after: the clock is a collaborator the spec chooses
local function isExpired(token, now)
	return token.expiresAt < now
end

it('is expired a second past its deadline', function ()
	expect(isExpired({ expiresAt = 100 }, 101)).toBeTruthy()
end)
```

The sandbox has no `os`, so the top half raises the moment it runs — which is the argument, made for you.

When you cannot fix it today, quarantine it with `it.skip(name, fn, reason)`: the reason is the paper trail every report prints, and the body stays so the test is one edit from running again. Never delete a flaky test and never wrap it in a retry; both convert "sometimes wrong" into "never checked". `--forbid-only` and an injected random are cheap; a fixed clock is a design change. Do the cheap ones today.

**Key points**

- Four causes: order, time, randomness, focus.
- Alone versus together tells order from the rest.
- Inject the clock and the random; reset shared state in a hook; `--forbid-only` on CI.
- Quarantine with `it.skip` and a reason; never delete, never retry.
