# Repeating while something is true

*while tests before the body and repeat tests after, so repeat always runs at least once.*

Luau Fundamentals · Control Flow · 9 min

---

A `while` loop asks its question, runs the body if the answer is yes, and then asks again. It keeps going until the answer is no.

That means **the body may never run at all**. If the condition is already false when the loop is reached, the whole thing is skipped and the script carries on.

**while**

```luau
local n = 3
while n > 0 do
	print("countdown", n)
	n -= 1
end
print("liftoff")

-- Already false at the top, so this body never runs.
local k = 0
while k > 0 do
	print("never printed")
	k -= 1
end
```

Prints the three countdown lines and then `liftoff`. Something inside the body has to change what the condition reads, or the loop never ends — here it is `n -= 1`.

## repeat runs first, asks later

**repeat … until**

```luau
local tries = 0
repeat
	tries += 1
	print("attempt", tries)
until tries >= 3

-- The condition is already true, and the body still runs once.
local done = true
repeat
	print("runs once anyway")
until done
```

Note that `until` is the *stopping* condition, the opposite of `while`'s. `until tries >= 3` means "keep going until this becomes true", where `while tries < 3` means "keep going while this stays true".

|  | `while cond do … end` | `repeat … until cond` |
| --- | --- | --- |
| When the test runs | Before the body | After the body |
| Minimum runs | Zero | One |
| The condition means | Keep going while true | Stop once true |
| Typical use | Draining a list, waiting for a state | Asking for input, retrying an attempt |

> **The loop that never ends**
>
> If nothing in the body changes what the condition reads, the loop runs forever. This sandbox has an instruction limit and will cut a runaway loop off, so you will get an error rather than a hung page — but in Studio, and in a real game, an infinite loop freezes the script that runs it. The fix is never to add a `break` on the way out; it is to make sure the variable the condition reads actually moves.

There is one small scoping surprise in `repeat`: a local declared inside the body is still visible to the `until` condition. `repeat local line = read() until line == "quit"` is legal, and the same trick written with a `while` is not, because a `while`'s condition sits outside the block.

Which of the two to reach for is usually decided by one question: **is it possible that the work should not happen even once?** If yes, `while`. If the first pass always has to happen — you have to roll the dice before you can check the number — `repeat`.

**Key points**

- `while` tests before the body, so it can run zero times.
- `repeat … until` tests after, so it always runs at least once, and `until` is the *stopping* condition.
- Something in the body must change what the condition reads, or the loop never ends.
- A local declared in a `repeat` body is still visible to its `until`.
