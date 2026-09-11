# nocheck, nonstrict and strict

*Three comments at the top of a file decide how hard the checker looks, and the default is nonstrict.*

Luau Intermediate · Types in Luau · 9 min

---

How hard the checker looks at a file is decided by a comment on its first line. There are three, they are per file, and they never spread to a file's neighbours.

The one people get wrong is the default. A file with no directive is **not** unchecked and it is **not** strict: it is nonstrict, which is why a file you never annotated still occasionally reports something.

| Directive | What it does | Use it for |
| --- | --- | --- |
| `--!nocheck` | No inference and no reports at all for this file | Generated code, a vendored file you will not fix |
| `--!nonstrict` | The default. Infers, but only reports what will certainly misbehave; unannotated parameters are `any` | Existing code you have not annotated yet |
| `--!strict` | Reports what *might* misbehave: missing fields, unnarrowed optionals, wrong returns | Every new file you write |

There is a fourth directive, `--!nolint`, which silences the linter rather than the type checker; the two are separate tools.

**The same file under two modes**

```luau
--!nonstrict
local function area(w, h)
	return w * h
end

-- nonstrict gives `w` and `h` the type `any`, so nothing is reported here.
print(area("3", 4))
```

Prints `12`: Luau coerces `"3"` at run time. Change the first line to `--!strict` and the unannotated parameters become an error of their own — strict mode will not infer `any` for you.

That is the difference in one sentence. Nonstrict quietly hands an unannotated parameter the type `any` and moves on. Strict refuses to, which is what makes it useful: it turns "I did not say" into a report instead of a silent hole.

Everything else follows from that. Under strict a record type must have the fields it claims, an optional must be checked before it is used, and a function must actually return what it says it returns.

## Turning it up gradually

Do not put `--!strict` at the top of forty files at once. The report you get is one enormous list, most of it in code you were not going to touch, and the usual outcome is that the directive comes back off again.

Add it to the **next new module** you write. Then to the module you are already editing this week. The errors arrive in the file you have in your head, and they stay a handful at a time.

**One new module, strict from line one**

```luau
--!strict
local Shipping = {}

function Shipping.cost(weight: number, express: boolean): number
	return weight * 2 + (if express then 5 else 0)
end

print(Shipping.cost(3, true))
```

Prints `11`. Nothing else in the project changed, and this file's callers are checked against its signature the moment they mention it.

**Key points**

- The mode is a comment on the file's first line and applies to that file only.
- **nonstrict is the default**, so a file with no directive is still checked — just leniently.
- `--!strict` is the one worth reaching for: it refuses to infer `any` for an unannotated parameter.
- `--!nocheck` is for generated or vendored code; `--!nolint` is a different tool and silences the linter.
- Adopt strict one new file at a time, never project-wide in one commit.
