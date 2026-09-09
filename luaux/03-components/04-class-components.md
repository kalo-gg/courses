# Class components

*A class works as a tag because a tag is a call; put call on the metatable and keep sources, not fields.*

LuauX · Components · 11 min

---

Luau has no classes, and LuauX needs none. A tag compiles to an ordinary call, so **anything callable works as a component** — including a table whose metatable has `__call`.

**A class used as a tag**

```luaux
local Profile = {}
Profile.__index = Profile

setmetatable(Profile, {
	__call = function(class, props: ProfileProps)
		return class.new(props):Render()
	end,
})

function Profile.new(props: ProfileProps): Profile
	local name = source(props.Name)

	return setmetatable({
		Name = name,
		Age = source(props.Age),
		Greeting = derive(function() return "Hi, " .. name() end),
	}, Profile)
end
```

`<Profile Name="Ada" Age={36} />` then constructs and renders in one step.

> **`__call` goes on the class's metatable, not on the class**
>
> `__index` is read off `Profile` itself, because that is what instances carry as their metatable. `__call` is different: Luau looks for it on **Profile's own** metatable. `function Profile.__call()` therefore defines a field nothing ever consults, and `<Profile />` fails with *attempt to call a table value*.

## Hold sources, not fields

The instance above stores `source(props.Name)` rather than `props.Name`. That is not a style choice. A reactive library tracks **reads of sources**, so a plain field registers no dependency and assigning to it later updates nothing — the difference between a class that renders once and one that stays live. When a component needs a *second* value computed from the first — a label reading `On` or `Off` from a boolean — do not store it: `derive(fn)` returns a read-only source that recomputes when anything it read changes, and it is the one Vide name besides `source` this course's projects use.

**Update through the source**

```luaux
local function changeName()
	-- Call the source. Anything derived from it, Greeting included,
	-- recomputes on its own.
	self.Name(NameField.Text)

	-- self.Name = NameField.Text would replace the source with a plain
	-- string and silently break every reader.
end
```

> **An element really is an Instance**
>
> `NameField.Text` above reads a property off an element the markup produced, because under Vide, Fluid and Fusion `<TextBox/>` **is** an Instance — assignable `.Parent`, indexable `.Size`. Under React it is an inert description and that line would not work. It is the clearest example of something LuauX cannot make portable, and the lesson "Two arrangements" explains why it does not try.

Both mistakes below are silent in the markup: a class with `__call` in the wrong place still looks callable, and a plain field still looks like state. The compiler cannot see either, because both are ordinary Luau. Fix the first, then make the second live.

**What you can do now**

- Write components as plain functions, and know that a Roblox class name always beats a local one.
- Reach for a dotted name when a collision is likely.
- Recognize `static_conditional_child` on sight, and reach for your library's `show` and `indexes` instead.
- Use a class as a component by putting `__call` on its metatable — and store sources in it, not fields.
