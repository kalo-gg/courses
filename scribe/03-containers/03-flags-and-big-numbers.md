# Flags and big numbers

*Flags hold up to 32 named booleans in one write; Big carries a currency past 2^53 and past 1.8e308.*

Scribe · Containers & Big Numbers · 10 min

---

When a fixed set of booleans belongs together, `Scribe.Flags(members)` holds them in one field addressed by name. `Enable`, `Disable`, `Toggle` (which returns the new state) and `Has` take a member name; `Get()` is the list of enabled names; `Clear()` turns every flag off. A name that is not a member is an error, and setting a flag to what it already is writes nothing. Flags start off; a starting set is written in `OnPlayerInit`. The cap is 32 members.

**Emberfall's Settings**

```luau
Settings = Scribe.Flags({ "Music", "Sfx", "TutorialDone" }),

data.Settings.Enable("Music")
data.Settings.Enable("Sfx")
data.Settings.Has("Music") --> true
data.Settings.Toggle("Sfx") --> false, the new state
data.Settings.Disable("Music")
data.Settings.Enable("TutorialDone")
data.Settings.Get() --> { "TutorialDone" }
data.Settings.Clear() -- every flag off
```

## Numbers past 2^53

A Luau number stops being exact past `2^53` and stops existing past `1.8e308`. `Scribe.Big(default, { Min, Max })` carries about 15 significant digits at any magnitude: the right trade for a currency whose magnitude is the point (Emberfall's prestige `Essence`), the wrong one for anything audited (`Gems` stay an `Int`). `Set`, `Increment`, `Decrement`, `Multiply` and `Divide` accept a number, a numeric string like `"1.5e100"`, or another big.

**Essence**

```luau
Essence = Scribe.Big(0, { Min = 0 }),

data.Essence.Set("1.5e100") -- a numeric string, because 1e400 is not a number
data.Essence.Increment("2e99") --> 1.7e100
data.Essence.Multiply(1.15) --> 1.955e100, the VIP bonus
data.Essence.Divide(2) --> 9.775e99, the prestige reset
print(data.Essence.Get():Short()) --> 9.775e99
```

| Method | Second argument | Returns |
| --- | --- | --- |
| `Set(value, replicate?)` | replicate flag | the new value |
| `Increment(delta, tags?)` | economy tags | the new value |
| `Decrement(delta, tags?)` | economy tags | the new value |
| `Multiply(factor, replicate?)` | replicate flag | the new value |
| `Divide(factor, replicate?)` | replicate flag; zero errors naming the field | the new value |
| `Update(fn, replicate?)` | replicate flag | the new value |

`Get()` returns a `BigValue`, not a number. It supports `+ - * /` and `tostring`, exposes `.M` and `.E`, and has a few methods: `:ToNumber()` is lossy and saturates past `1.8e308`, `:Pow(n)` and `:Log10()` are reads, and `:Short(decimals?)` formats for UI: `1500` shows `1.50K`, `1e40` shows `10.00DDc`, `1.5e100` stays `1.5e100`. `Scribe.Short(value)` is the same renderer for a plain number or a Big.

> **Comparisons need a big on both sides**
>
> `data.Essence.Get() < 2000` throws (attempt to compare table < number) and `data.Essence.Get() == 5` is silently false. Compare two bigs directly, or convert first: `data.Essence.Get():ToNumber() >= price`.

**Affordability, the right way**

```luau
if data.Essence.Get():ToNumber() >= price then
	data.Essence.Decrement(price, {
		TransactionType = Enum.AnalyticsEconomyTransactionType.Shop,
		ItemSku = itemId,
	})
end

if data.Essence.Get() < data.Banked.Get() then -- fine: both are bigs
end
```

**Key points**

- `Scribe.Flags(members)`: one field, addressed by member name; a non-member is an error; 32 members at most.
- `Scribe.Big` trades precision for range: about 15 digits at any magnitude.
- `Multiply` and `Divide` live on big fields only; `Increment` and `Decrement` take economy tags.
- `Get()` is a `BigValue`: `Short`, `ToNumber`, `Pow`, `Log10`, `.M`, `.E`; compare bigs to bigs or convert first.
