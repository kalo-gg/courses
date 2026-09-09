# Async and isolation

*Two properties that genuinely differ between backends, and neither is a detail.*

Lest · Backends · 10 min

---

Test bodies are called synchronously, so what "async" means depends entirely on whether the backend running them **has a scheduler**.

| Backend | Scheduler | How a timeout is enforced |
| --- | --- | --- |
| `native` | None — a bare Luau VM, no `task` library | The VM's interrupt callback, per test |
| `lune` / `lute` | The real runtime's own | Killing the process — a whole-suite budget |
| `cloud` | The engine's task scheduler | A per-spec deadline from `timeout_ms` |

**The same wait, two backends**

```luau
-- native: no task library and nothing to yield to; this is an error, not a wait
task.wait(0.1)

-- lune: the runtime's own scheduler drives it
local task = require('@lune/task')
task.wait(0.1)
```

Not runnable here. The sandbox is a bare VM: `task` is nil, exactly as on native.

> **`task.wait` does not exist on the native backend**
>
> There is no task library and nothing to yield to. Full task-library semantics exist only where a task library does — so if a spec needs one, give its suite a backend that has one. That is a configuration decision, not something to work around in the test.

## Isolation

The test registry is reset before each spec file on **every** backend, so no spec ever sees another's tests. How far isolation goes beyond that — whether the *modules* a spec requires were already required by the spec before it — is where they part company.

**The module both spec files require**

```luau
-- counter.luau — an ordinary helper that happens to hold state
local counter = { n = 0 }
function counter.bump()
	counter.n += 1
	return counter.n
end
return counter

-- a.spec.luau and b.spec.luau both require it. On native and cloud each
-- spec file gets its own copy; on lune and lute they share one.
```

Same source, two answers, depending on where the suite runs — which is why the reset belongs in a hook you wrote rather than in an assumption.

| Backend | Per spec file |
| --- | --- |
| `native` | Its own VM — module state is fresh |
| `cloud` | Its own task — module state is fresh |
| `lune` / `lute` | **One process, one module cache** for the whole suite |

> **The one that will bite you**
>
> Under `lune` and `lute`, a helper module's state **does persist across spec files**. A counter, a cache, a registry initialized at require time — all of it carries over. Do not rely on a shared module resetting itself between specs; reset it in a `beforeEach` you wrote.

Write the reset yourself and put it where the dependency is visible: a `beforeEach` (or `beforeAll`) that calls the module's own `reset`, or reassigns what the spec relies on. A spec that resets what it uses passes on every backend; a spec that assumes freshness passes on two of them.

**Key points**

- Test bodies are synchronous; "async" means whatever the backend's scheduler means.
- `native` has no `task` library and interrupts a test per `timeout_ms`; `lune`/`lute` kill the process on a whole-suite budget.
- The registry resets before every spec file on every backend.
- Under `lune`/`lute` a helper module's state persists across spec files — reset it in a hook you wrote.
