# The class pattern

*A table of methods, an index pointing at itself, a new that sets the metatable, and self.*

Luau Intermediate · Metatables · 10 min

---

Luau has no `class` keyword. What it has is one table holding the methods, `__index` pointing at that table, and a constructor that attaches it — and because every instance shares the same method table, the methods are stored once no matter how many objects exist.

The pattern is four lines of boilerplate you will write hundreds of times. Learn the shape rather than the reasoning and you will still get it right; learn both and you will know why a method is on the class and the data is on the instance.

**The pattern, in full**

```luau
local Account = {}
Account.__index = Account

function Account.new(owner: string)
	return setmetatable({ owner = owner, balance = 0 }, Account)
end

function Account.deposit(self, amount: number)
	self.balance += amount
end

local a = Account.new("mira")
a:deposit(5)
a:deposit(7)
print(a.owner, a.balance)
```

Prints `mira  12`. `Account.__index = Account` is the line that does the work: an instance that misses on `deposit` falls through to the class table and finds it there.

`a:deposit(5)` is exactly `Account.deposit(a, 5)`. The colon is one piece of syntax with one job: it passes the thing on its left as the first argument. That first parameter is conventionally called `self`, and it is an ordinary parameter — nothing declares it for you.

The same colon works at the declaration site: `function Account:deposit(amount)` adds an implicit `self`. This course spells `self` out, because under strict mode it has to be annotated anyway.

## Typing it

**`typeof(setmetatable(…))`**

```luau
--!strict
type AccountData = { owner: string, balance: number }

local Account = {}
Account.__index = Account

export type Acc = typeof(setmetatable({} :: AccountData, Account))

function Account.new(owner: string): Acc
	return setmetatable({ owner = owner, balance = 0 }, Account)
end

function Account.deposit(self: Acc, amount: number)
	self.balance += amount
end

local a = Account.new("mira")
a:deposit(5)
print(a.balance)
```

Prints `5`. `typeof(setmetatable({} :: AccountData, Account))` is the documented way to type the pattern: the instance's own fields, plus everything reachable through the class table.

Two details in that block are not optional under strict mode. The `export type` has to come **after** `Account.__index = Account`, because `typeof` reads the class table as it is at that point. And `self` needs its annotation: Luau considered a shared self type and decided against it, so every method spells the receiver out.

It is more typing than a language with a `class` keyword. What you get for it is that there is no keyword — the pattern is made of pieces you already understand, and you can bend it when you need to.

**Key points**

- The class pattern is a method table, `Class.__index = Class`, and a `new` that calls `setmetatable`.
- Methods live once on the class; data lives on the instance.
- `a:method(x)` is `Class.method(a, x)` — the colon passes the receiver, and forgetting it is the classic bug.
- Type it with `typeof(setmetatable({} :: Data, Class))`, declared after `__index` is set.
- Strict mode requires the `self` parameter to be annotated; Luau deliberately has no shared self type.
