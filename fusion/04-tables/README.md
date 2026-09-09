# 4. Tables & Lists

*Turning a table of data into a table of something else, and into a list of instances that comes and goes.*

Fusion · Medium

---

An inventory, a leaderboard, a row of buttons: all of them are a table of data that has to become a table of instances and stay right as the data changes. Fusion's answer is the For family. This module covers all three, the scope each processed item gets, what happens to an instance when its item leaves the table, and the other half of a real screen: parts that appear and disappear, built from a Computed that returns an instance or nothing.

## Lessons

1. [ForValues](01-forvalues.md) — Process every value in a table, keep the keys, and give each entry a scope of its own.
2. [ForKeys and ForPairs](02-forkeys-and-forpairs.md) — The other two thirds of the family: rename the keys, or rewrite both halves at once.
3. [A list that changes](03-dynamic-children.md) — Drop a For object into Children and the interface follows its data, one instance per entry.
4. [Showing and hiding](04-showing-and-hiding.md) — Two ways to make a panel come and go: rebuild it from a Computed, or leave it there and bind Visible.
