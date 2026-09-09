# Planning and testing the HUD

*Split a screen into a state module, components and an entry script, then test a component without Roblox.*

Vide · Shipping a Vide App · 10 min

---

The final project is a HUD: a health bar with a warning, the inventory panel from Project 1, a feed of notifications that fade out, and a Tab key that hides the inventory. Before writing any of it, decide three things: which sources exist (`health`, `coins`, `items`, `feed`, `inventoryOpen`), who writes them (the game, through functions in one state module), and which component reads which. Components never write state they do not own; the entry script mounts once and connects input.

**The tree you will build**

```text
src/
├── main.client.luau
├── state.luau
└── components/
    ├── HealthBar.luau
    ├── Inventory.luau
    └── Feed.luau
```

`state.luau` owns every source and every function that writes one: `damage` and `heal` for `health`, `pickup` and `earn` for `items` and `coins`, `notify` and `clearFeed` for `feed`. The components only read — `HealthBar.luau` `health` and `MAX_HEALTH`, `Inventory.luau` `items`, `coins` and `CAPACITY`, `Feed.luau` `feed`. `main.client.luau` mounts the tree and is the one file that writes `inventoryOpen`, from the Tab key.

## Testing a component without Roblox

A component is a function, so it can be tested the way Vide tests itself: against a mock, with no engine running. Vide's repository ships `test/mock.luau` for exactly this, and the Kalo sandbox is the same idea grown up. Mount the component into a fake `PlayerGui`, drive it — `click` a button, write a source, `step` some frames — and assert on the instance's properties. Turn strict mode on first so the test runs the checks Studio runs. `vide.step` is the public function the sandbox's `step()` wraps; in a test harness stepping by hand is exactly what you want.

**A headless test**

```luau
vide.strict = true -- tests should run the checks Studio runs

local unmount = mount(Counter, PlayerGui)
local button = PlayerGui.Counter

assert(button.Text == "count: 0", "starts at zero")
click(button)
assert(button.Text == "count: 1", "one press counts one")

unmount()
click(button)
assert(button.Text == "count: 1", "nothing updates after unmount")
```

> **Name your instances**
>
> Every component in the project sets `Name`. Tests, `FindFirstChild` and Studio's explorer all find things by name; `gui.Panel.HealthBar.Label` reads like the tree it is.

**Key points**

- Plan a screen as sources, the functions that write them, and the components that read them.
- One state module, one entry script that mounts and connects input, one file per component.
- Test components headlessly: strict mode on, mount, drive with click and step, assert, unmount.
- Name every instance so tests and the explorer can find it.
