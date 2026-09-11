# When it goes wrong on purpose

*error stops the script, pcall catches it, and assert is the short way to demand a condition.*

Luau Fundamentals · Modules & Mistakes · 10 min

---

So far every error you have seen was an accident. Sometimes you want one on purpose: a function given an argument it cannot possibly work with should say so loudly rather than return something wrong and let the damage surface three files away.

`error(message)` raises one. It stops the current function, unwinds through every caller, and — unless somebody catches it — stops the script with a traceback.

**error, and the message it prints**

```luau
local function openDoor(locked)
	if locked then
		error("the door is locked")
	end
	return "opened"
end

print(openDoor(false)) -- opened
print(openDoor(true))  -- stops here
```

The second call raises `the door is locked`, prefixed with the file and line where `error` was called. `error(msg, 0)` suppresses that prefix, which is what you want when the message is for a player rather than for you.

## Catching it

**pcall returns ok first**

```luau
local ok, err = pcall(function()
	error("the door is locked")
end)
print(ok, err) -- false  <line>: the door is locked

local ok2, a, b = pcall(function()
	return 1, 2
end)
print(ok2, a, b) -- true  1  2
```

`pcall` — protected call — runs a function and catches anything it throws. It returns `false` plus the message, or `true` plus **everything the function returned**. `ok` is always first, which is why `local ok, value = pcall(f)` reads so naturally.

`pcall(f, ...)` takes the arguments to pass on, so `pcall(parse, text)` is the usual spelling — no wrapper function needed. `xpcall(f, handler, ...)` is the same thing with a handler that runs *before* the stack unwinds, which is the only way to capture a traceback of where the error actually happened.

The rule for where to put a `pcall` is worth stating plainly: **catch at a boundary you can do something about.** Wrapping every call in the program makes every failure silent. Wrapping the one dispatcher that runs a player's command means a bad command prints a message and the game keeps going, which is exactly what the project at the end of this module does.

| Function | Does | Use it when |
| --- | --- | --- |
| `error(msg, level?)` | Raises `msg`; `level` of `0` drops the file:line prefix | The caller broke a rule you can state |
| `assert(v, msg?)` | Returns its arguments when `v` is truthy, raises `msg` when it is not | A one-line demand at the top of a function |
| `pcall(f, …)` | Runs `f`, returns `ok` plus results or `ok` plus the message | You are at a boundary and can recover |
| `xpcall(f, handler, …)` | The same, with a handler that runs before the stack unwinds | You need the traceback, not just the message |

`assert` is worth a second look because it is easy to misread. It **returns** its arguments when the condition holds, so `local n = assert(tonumber(text), "not a number")` both checks and assigns in one line. And its message is not lazy: the string is built every time the line runs, whether or not the assertion fails, so an expensive message inside a hot loop costs you on every pass.

**Key points**

- `error(msg)` raises and unwinds; `error(msg, 0)` drops the file:line prefix.
- `pcall(f, …)` returns `ok` first, then the results or the message.
- `assert(v, msg)` returns its arguments when `v` holds, so it checks and assigns in one line.
- Catch at a boundary you can act on, not around every call.
