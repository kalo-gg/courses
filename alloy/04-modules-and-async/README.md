# 4. Modules, Results and async

*Imports and exports, failure as a value, work that yields, and cleanup that knows what it is cleaning.*

Alloy · Medium

---

How Alloy files talk to each other, fail, wait and clean up. The module starts with `import` and `export`, service and std imports, and namespaces. Then come Results: `Ok`, `Err`, `try` and `Result.pcall` at the edge where Roblox throws. Next is async code: `async function`, `await` and where it may stand, `try await`, and the `Future` combinators. It ends with `delete`, `destroy`, `after`, and the std `Scope` and `Signal`. The first project then builds the satchel's core inventory module from these pieces.

## Lessons

1. [Modules and imports](01-modules.md) — `export` on what leaves a file and `import` on what arrives, with services, the std and namespaces.
2. [Results and try](02-results.md) — Return failure as a value, pass it up with `try`, and catch Roblox's throws once, at the edge.
3. [Async and Futures](03-async.md) — `async` functions return Futures, `await` waits for them, and the combinators wait for several at once.
4. [Cleanup](04-cleanup.md) — `delete` cleans up whatever it is given, `destroy` and `after` wait, and a `Scope` ends a round in one line.
