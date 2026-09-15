# Validation and the trust boundary

*Blink checks every incoming field for you; WriteValidations checks the outgoing ones for you.*

Blink · Best Practices & Capstone · 10 min

---

There is an asymmetry in the generated code and it is the whole security story. Reads are always validated, because the sender is not trusted. Writes are validated only if you ask, because you are. Nothing you can set changes the first half.

**Server.luau (generated), reading a `string(1..30)`**

```luau
local Length = buffer.readu8(RecieveBuffer, BLOCK_START + 0)
local Value = buffer.readstring(RecieveBuffer, Read(Length), Length)
if #Value < 1 then error(`Expected length of "Value" to be larger than 1, got {#Value} instead.`) end
if #Value > 30 then error(`Expected length of "Value" to be smaller than 30, got {#Value} instead.`) end
```

Those two lines are in the module whether or not you asked for validation, because they are the type.

Which means your ranges are the validation. `string` and `u32` are not lazy sizing decisions, they are a trust boundary left open: an unbounded string is whatever the client felt like sending, and the listener behind it is the one that has to cope. Bound the type and the error happens in the deserialiser, before any of your code runs.

## Validating your own writes

**Client.luau with `option WriteValidations = true`**

```luau
function SerdesFunctions.WriteEVENT_Rename(Value: string): ()
	-- Allocate BLOCK: 2 bytes
	local BLOCK_START = Allocate(2)
	buffer.writeu8(SendBuffer, BLOCK_START + 0, 0)
	if typeof(Value) ~= "string" then
		error(`Expected Value to be of type string, got {type(Value)} instead.`)
	end
	if #Value < 1 then error(`Expected length of "Value" to be larger than 1, got {#Value} instead.`) end
	if #Value > 30 then error(`Expected length of "Value" to be smaller than 30, got {#Value} instead.`) end
	local Length = #Value
	buffer.writeu8(SendBuffer, BLOCK_START + 1, Length)
```

The same two range lines, plus a `typeof` check, now on the sending side. Without the option, none of these four lines exist.

It costs performance on every send and the documentation says to turn it off in production. The honest middle is on in development and off in a release build, because the bug it catches is yours: passing a `Vector3` where the description said `vector<i16>`, or a number where a string was declared.

> **The message says one thing and the code does another**
>
> The generated check for an upper bound of 10 reads `if Value > 10 then error("... to be smaller than 10 ...")`. It fires only when the value is strictly greater, so 10 is accepted and the word "smaller" is loose. Read the comparison, not the sentence.

| Validated on receipt | Not validated by anything |
| --- | --- |
| Number ranges, string, buffer and array lengths | Whether this player is allowed to fire this event |
| Enum tags and struct shape | Whether the value makes sense for the game state |
| A non-optional `Instance` actually arrived | The 1,000-byte unreliable ceiling |

## Your turn

**Key points**

- Reads are always validated. That code is generated from the type and cannot be turned off.
- Your ranges are the validation, so an unbounded type is a security decision.
- `WriteValidations` adds `typeof` and range checks to the sending side; on in development, off in a release.
- Ranges are inclusive at both ends, whatever the error message's wording suggests.
- Nothing checks entitlement or game sense. That is the listener's job, every time.
