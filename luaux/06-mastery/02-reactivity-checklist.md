# What updates, and what never will

*Where a source may be read, which reads survive the first frame, and the checklist for when nothing moves.*

LuauX · Best Practices & Capstone · 10 min

---

Every "it renders once and never changes" bug in a LuauX screen is the same bug: a source was **read** somewhere nothing will run again. The compiler wraps exactly one construct, your library re-runs exactly what you handed it as a function, and everything else runs once, while the tree is being built. So the question is never "is this reactive?" but "where does this read happen, and who will run that code again?"

**Four reads, one line apart**

```luaux
local function Panel(props)
	local title = props.Title()                   -- read once, when Panel is called

	return (
		<Frame>
			<TextLabel Text={title} />            -- a finished string: frozen
			<TextLabel Text={props.Title} />      -- the source itself: the library re-reads it
			<TextLabel>Now: {props.Title}</TextLabel>  -- LuauX wraps the template: live
			<TextButton Activated={function() print(props.Title()) end} />
		</Frame>
	)
end
```

Only the first line is a mistake, and it is the one that looks most like ordinary Luau.

## Pass the source, read at the edge

The rule that follows is short: **hand the source down, and read it at the last possible place.** A component takes `vide.Source<T>` and passes it to an attribute, into interpolated text, into `show`'s condition or into `indexes`' transform — every one of those is code the library or the compiler will run again. A read into a local, a string built with `..`, or a comparison in the component body is a value, and a value cannot change.

**The same panel, read at the edge**

```luaux
local function Panel(props)
	return (
		<Frame>
			<TextLabel>Now: {props.Title}</TextLabel>
			{show(function()
				return props.Title() ~= ""
			end, function()
				return (<TextLabel TextTransparency={0.5}>Tap to rename</TextLabel>)
			end)}
		</Frame>
	)
end
```

`props.Title()` appears here too — inside a function `show` calls again whenever the source updates, which is what makes the same read live.

The table below is the same rule read backwards. You do not usually notice a read in the wrong place; you notice a label that will not change. So start from what is on screen, and let the symptom name the read to go and find — it is one of five, every time.

| Symptom | First thing to check | The fix |
| --- | --- | --- |
| A label never changes | Was the text built with `..` in an attribute? | Interpolate between the tags, or pass the source whole |
| A child never appears | `and`/`or` in a hole | `show(condition, component)` |
| A list never updates | A `for` loop in a hole | `indexes(source, fn)` |
| The screen freezes after one frame | Did a component read `props.X()` into a local? | Pass the source down; read at the edge |
| A React label reads `RoactBinding(…)` | A binding inside interpolated text | Pass it whole, or `Text={binding:map(f)}` |

Five symptoms, five reads in the wrong place. Nothing on this list is a compiler bug, and nothing on it is reported.

**Key points**

- A read is live only inside something that runs again: a wrapper, a condition, a transform, a handler.
- LuauX wraps exactly one construct — interpolated text; every other wrapper is one you passed.
- Pass the source down and read it at the edge; a read into a local freezes the value.
- Five symptoms, one cause: run the checklist before suspecting the compiler.
