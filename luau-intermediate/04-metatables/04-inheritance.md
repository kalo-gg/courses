# Inheritance, and when not to

*A metatable on the class table itself gives you a parent, and composition usually beats it anyway.*

Luau Intermediate · Metatables · 11 min

---

If an instance reaches its methods through `__index`, then a class can reach *its* methods the same way. Give the child class table a metatable whose `__index` is the parent, and a method missing on the child is found one level up.

That is all inheritance is here. Two `__index` links: one from instance to class, one from class to parent.

**`Savings` extending `Account`**

```luau
local Savings = setmetatable({}, { __index = Account })
Savings.__index = Savings

function Savings.new(owner: string)
	local self = Account.new(owner)
	self.rate = 0.05
	return setmetatable(self, Savings)
end

function Savings.addInterest(self)
	self.balance += self.balance * self.rate
end

local s = Savings.new("mira")
s:deposit(100)   -- inherited from Account
s:addInterest()
print(s.owner, s.balance)
```

With the `Account` from the last lesson, this prints `mira  105`. Two lines do the linking: the child *table* gets a metatable pointing at the parent, and the child's own `__index` still points at itself.

Read the two lines carefully, because they look almost the same and mean different things.

`Savings = setmetatable({}, { __index = Account })` is about the **class table**: a name missing on `Savings` is looked up on `Account`. `Savings.__index = Savings` is about **instances**: a name missing on an instance is looked up on `Savings`, which then falls through to `Account` by the first rule. Both are needed, and swapping them silently gives you an object with no methods at all.

## Two levels is usually one too many

Inheritance is easy to reach for and hard to unpick later. A child that overrides half its parent's methods is a class you can no longer read on its own, and a three-level chain is one where nobody knows which level a method came from.

The alternative is **composition**: a class that *has* the other thing instead of *being* it. A `Slot` that holds an item is easier to reason about than a `Slot` that inherits from `Item`, and a `Bag` that owns a list of slots is easier than a `Bag` that extends one.

**A `Bag` that has slots rather than being one**

```luau
local Bag = {}
Bag.__index = Bag

function Bag.new()
	return setmetatable({ slots = {} }, Bag)
end

function Bag.add(self, slot)
	table.insert(self.slots, slot)
	return #self.slots
end

local bag = Bag.new()
print(bag:add({ item = "lamp" }), bag:add({ item = "rope" }))
```

Prints `1  2`. Nothing is inherited: `Bag` keeps a list and forwards to it. When you later need a bag that behaves differently, you change one method instead of rearranging a hierarchy.

> **Every level is another lookup on a miss**
>
> Correctness first: use inheritance where it genuinely models the problem. But be aware that a lookup which fails at every level walks the whole chain before it answers `nil`, and a method resolved three levels up is resolved three levels up on **every call**. In hot code — a per-frame update, a loop over thousands of entities — that shows. Luau Advanced measures it; here it is one more reason to keep chains at one level.

**Key points**

- Inheritance is two `__index` links: instance to class, and class to parent.
- `Child = setmetatable({}, { __index = Parent })` links the class tables; `Child.__index = Child` links the instances.
- A miss walks the whole chain, so every level costs a lookup on every failed step.
- Composition — a class that *has* the other thing — is usually the better default.
- Keep chains at one level unless the hierarchy is genuinely what you are modelling.
