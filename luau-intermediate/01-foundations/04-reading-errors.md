# Reading a type error

*Expected against got, where each side came from, and the three errors you will see most.*

Luau Intermediate · Types in Luau · 10 min

---

Almost every Luau type error is the same sentence with different nouns in it: *this type could not be converted into that one*. The first type is what arrived, the second is what was expected, and the fix is in whichever of the two you got wrong.

Read it backwards. Find the expected type first — that is the annotation you wrote — then ask where the arriving value came from.

**Type 'string' could not be converted into 'number'**

```luau
--!strict
local prices = { lamp = 12, rope = "8" }

local function total(price: number, qty: number): number
	return price * qty
end

print(total(prices.rope, 2))
```

The report lands on `prices.rope`: **Type 'string' could not be converted into 'number'**. Expected `number`, because `total`'s first parameter says so. Got `string`, because the table literal put one there. Two candidate fixes, and only you know which is right.

That is the whole method: **expected**, **got**, and the two places they came from. The expected side is nearly always an annotation, so it is easy to find. The got side is a value, so you follow it back — usually one or two hops.

Three shapes cover most of what you will see, and they have distinct fixes.

| The error says | It means | The usual fix |
| --- | --- | --- |
| Type 'A' could not be converted into 'B' | A value of the wrong type reached an annotated position | Convert it, or change the annotation if the annotation was the lie |
| Key 'x' not found in table '{ … }' | You read or wrote a field the record type does not list | Fix the spelling, or add the field to the type |
| Value of type 'T?' could not be converted into 'T' | An optional was used without ruling out `nil` | Check it with `if`, or supply a default |

The third is the one that feels unfair and is not: the checker is telling you the value might be missing and you have not said what to do about that.

## The error is not always where the fix is

The checker reports where two types met. That is often not where either of them was decided. A `string` handed to a `number` parameter may have been put in a table fifty lines earlier by a function whose own return type was already wrong.

So when the reported line looks innocent, do not annotate it into silence. Follow the **got** side one hop back and check whether the value or the signature is the thing that is lying.

**Reported here, wrong over there**

```luau
--!strict
-- The lie is on this line: it returns a string, not a number.
local function parseQty(text: string): number
	return text :: any
end

local function restock(qty: number): number
	return qty + 1
end

-- ...but the report arrives here, if it arrives at all.
print(restock(parseQty("4")))
```

Prints `5`, because Luau coerces. The `:: any` inside `parseQty` is what makes the return type a lie, and it means the checker has nothing to complain about at the call. Remove the cast and the error appears where it belongs.

**Key points**

- Every conversion error names the type that **arrived** and the type that was **expected**; read the expected side first.
- Three shapes cover most reports: not convertible, key not found, and an optional used without narrowing.
- The report lands where two types met, which is often not where either was decided — follow the value one hop back.
- Decide each time whether the annotation or the code is the thing that is wrong; the message cannot tell you.
- Silencing a report with `:: any` removes the warning and leaves the bug.
