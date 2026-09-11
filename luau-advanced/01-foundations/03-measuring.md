# Measuring instead of guessing

*A change is not an optimization until something says so, and a count is a fair thing to say it with.*

Luau Advanced · How Luau Runs · 9 min

---

Every rule in the last lesson is a heuristic, and heuristics are for deciding **where to look**. They are not evidence. A change is not an optimization until a measurement says so, and until then it is a change that made the code harder to read.

> **There is no clock in this sandbox**
>
> `os`, `debug` and `io` are not exposed by the Kalo runtime, so nothing in this course times anything, and no number you see here came from a stopwatch. Every measurement in these lessons is an **operation count** — a counter incremented where the work happens — or a reading from `gcinfo()`, which reports live heap in whole kilobytes. On a real machine you would use `os.clock` in standalone Luau (the `luau` CLI, Lune) or the microprofiler in Roblox Studio. Both are named in the reference and neither is used in an exercise.

**Counting work instead of time**

```luau
local reads = 0
local function readField(t: { n: number }): number
	reads += 1 -- an operation count: deterministic, and honest about what it is
	return t.n
end

local box = { n = 7 }
local sum = 0
for _ = 1, 4 do
	sum += readField(box)
end

print("sum", sum, "reads", reads)
```

Prints `sum	28	reads	4`. Run it a thousand times and it says `4` a thousand times — which is the whole advantage.

A counter is **deterministic**: it gives the same answer on a loaded machine, on a fast one, and in a browser tab. That makes it a better teaching instrument than a noisy timer, and a worse tuning instrument, because it counts operations you chose to count rather than the ones that actually cost. Knowing which of the two you are doing is most of the skill.

The one real measurement available here is memory. `gcinfo()` returns the live heap in whole kilobytes, so reading it either side of a loop tells you what that loop kept.

**The one number this sandbox does report**

```luau
local before = gcinfo() -- live heap, in whole kilobytes

local hold = {}
for i = 1, 20000 do
	hold[i] = { i } -- 20,000 tables, all still reachable
end

local after = gcinfo()
print("held onto memory:", after - before > 0)
```

Prints `held onto memory:	true`. The difference is deliberately compared rather than printed: the exact kilobyte figure depends on the host, the boolean does not.

## The measurement that lies

Most bad performance numbers are not fabricated, they are misread. Three ways it happens: measuring at the wrong **scale**, so the shape that wins on ten items loses on ten thousand; measuring work the optimizer can **delete**, because nothing reads the result; and measuring **once**, so what you learned was the state of the machine that second.

**Key points**

- A change is not an optimization until a measurement says so.
- This sandbox has no clock: every number here is an operation count or a `gcinfo()` reading, and the prose says which.
- Counters are deterministic and therefore good for teaching; a real timer is what you tune with.
- Wrong scale, deletable work and a single run are the three ways a confident number turns out to be meaningless.
