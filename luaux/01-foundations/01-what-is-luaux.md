# What is LuauX?

*Markup in, plain Luau out, nothing left behind: what LuauX is and what it leaves to your library.*

LuauX · LuauX Foundations · 9 min

---

Building a Roblox interface in code means a lot of punctuation. Nesting shows up as indentation and closing braces rather than as structure, and it takes a moment to see the tree. **LuauX** lets you write the tree as a tree.

**The same panel, twice**

```luau
-- in Luau
create "Frame" {
	Size = UDim2.fromOffset(320, 0),
	create "TextLabel" { Text = "Shop" },
	create "TextButton" { Text = "Buy" },
}
```

**In LuauX**

```luaux
<Frame Size={UDim2.fromOffset(320, 0)}>
	<TextLabel>Shop</TextLabel>
	<TextButton>Buy</TextButton>
</Frame>
```

The second compiles into the equivalent of the first — the real output is `create("Frame")({ … })`, which the lesson "What ports, and what does not" shows in full. What ships is the Luau.

## Zero runtime

This matters more than it sounds. LuauX has no library to install into your game, no module every file must require, and nothing to appear in a stack trace. The compiler runs on your machine and the output is code you could have written by hand.

> **"No runtime" is literally true**
>
> Two constructs need a helper — reading a reactive value inside interpolated text, and merging a spread. LuauX **inlines** those one-liners into the files that use them, as `__luaux_read` and `__luaux_merge`, and emits no `require` at all. There is no module to install, because a require is location-dependent and no configured path would be right for every file.

> **Line numbers survive the compile**
>
> Line N of the generated Luau is line N of the `.luaux` source. A Studio error therefore points at the line you actually wrote, which is the difference between a compiled language you can debug and one you cannot.

So three things share the work, and keeping them apart is most of this course. **LuauX** turns a tag into a constructor call and checks the class and property names while it does. **Your library** — Vide in this course's examples — builds the Instances and decides what updates. **Rojo** syncs the compiled files into Studio, exactly as it would sync files you typed.

**Key points**

- LuauX compiles `.luaux` markup into ordinary Luau that calls your library.
- There is no runtime: two one-line helpers are inlined where used and no `require` is emitted.
- Line N of the output is line N of the source, so Studio errors point at the line you wrote.
- Reactivity, Instances and syncing belong to your library and to Rojo, not to LuauX.
