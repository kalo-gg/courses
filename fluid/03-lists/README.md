# 3. Lists & Control Flow

*Showing one thing or another, and lists whose entries know when they are leaving.*

Fluid · Medium

---

`show` and `switch` pick between branches, and their delayed variants hold the outgoing one long enough to animate it away. `for_values` turns a table of data into a table of anything, and Fluid's version carries an idea the other libraries do not have: an entry is told when it has been removed, and can ask for time before it goes. The module closes by building the scoreboard.

## Lessons

1. [show and switch](01-show-and-switch.md) — Pick one branch from state with show, or one of many with switch — both are derives underneath.
2. [Branches that linger](02-delayed-branches.md) — Hold the outgoing branch long enough to animate it away, and read the array that comes back.
3. [for_values and for_keys](03-for-values.md) — Map a table's values, and learn what the transform is handed and what the output is not.
4. [Entries that leave slowly](04-leaving-entries.md) — An entry is told when it has been removed, and can ask for time before it goes.
