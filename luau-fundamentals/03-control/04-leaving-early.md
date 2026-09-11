# Leaving a loop early

*break stops the loop, continue skips the rest of one pass, and nesting decides which one.*

Luau Fundamentals · Control Flow · 9 min

---

Two keywords change a loop's flow from inside it.

- **`break`** abandons the loop entirely and carries on after its `end`.
- **`continue`** abandons the rest of *this pass* and goes straight to the next one.

Both work in every loop shape: `while`, `repeat` and both kinds of `for`.

**break**

```luau
for i = 1, 10 do
	if i > 3 then
		break
	end
	print("kept", i)
end

print("after the loop")
```

Prints `kept 1`, `kept 2`, `kept 3`, then `after the loop`. The loop was written to go to 10 and stopped at 4 — `break` is how you say "I have found what I came for".

**continue**

```luau
for i = 1, 6 do
	if i % 2 == 0 then
		continue
	end
	print("odd", i)
end
```

Prints `odd 1`, `odd 3`, `odd 5`. The `print` is skipped for even numbers, but the loop itself carries on to the end. Without `continue` you would wrap the body in an `if i % 2 ~= 0 then … end`, which works and indents everything by one more level.

## Which loop does it leave?

Both keywords apply to the **innermost** loop containing them, and there is no labelled `break` in Luau to reach any further out.

When you genuinely need to leave two loops at once there are two honest options: set a flag that the outer loop's condition reads, or — once Module 5 has given you functions — put the loops in a function and `return`, which leaves as many loops as it likes in one word. The second is almost always the clearer of the two.

> **`continue` is Luau's, not Lua's**
>
> Lua 5.1 has no `continue` at all, so a Lua tutorial will show you a flag variable or a nested `if` instead. Both still work in Luau and you will meet them in older Roblox code. Lua 5.2 solved the same problem with `goto continue` and a label, and Luau has never adopted `goto` — if you find one in a snippet, it is not Luau.

One placement rule: `continue` must be the last statement in its block. `if bad then continue end` is fine because the `continue` ends that block; writing a line *after* a `continue` inside the same block is a syntax error rather than dead code, which is a small kindness.

**Key points**

- `break` leaves the loop; `continue` skips the rest of one pass.
- Both apply to the innermost loop only, and Luau has no labelled break.
- `continue` is Luau's; Lua 5.1 uses a flag or a nested `if`, and Lua 5.2 uses `goto`, which Luau does not have.
- `continue` must be the last statement in its block.
