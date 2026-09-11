# 1. How Luau Runs

*Bytecode, the VM, and the difference between a real measurement and a guess.*

Luau Advanced · Easy

---

Performance advice is worth exactly as much as the model behind it. This module builds that model: what the compiler does before your code runs, what the VM does while it runs, why a local is cheaper than a global, and how to compare two versions of a function honestly. It ends on native code generation, which is the one switch that changes the rules.

## Lessons

1. [What happens before it runs](01-compiler-and-vm.md) — Source becomes bytecode, bytecode runs on a register machine, and that split explains most of the rules.
2. [What costs what](02-what-costs-what.md) — A rough price list for the operations in a loop body, and why allocation is the one that bites.
3. [Measuring instead of guessing](03-measuring.md) — A change is not an optimization until something says so, and a count is a fair thing to say it with.
4. [Native code generation](04-native-codegen.md) — What --!native and @native ask for, which functions actually gain from it, and which cannot.
