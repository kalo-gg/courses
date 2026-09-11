# Version requirements

*A bare version means caret, and two of Wally's semver rules differ from the Cargo it copies.*

Wally · The Manifest · 11 min

---

A requirement is not a version. It is a **set** of versions you are willing to accept, and Wally picks one from it at install time. The syntax is Cargo's, because Wally is a Rust program using the same `semver` crate — so if you have written a `Cargo.toml` this will look familiar. Two of the behaviours behind it will not.

**Every operator, once**

```toml
[dependencies]
Any     = "biff/minimal@*"
Caret   = "biff/minimal@^1.2.3"
Bare    = "biff/minimal@1.2.3"
Tilde   = "biff/minimal@~1.2.3"
Exact   = "biff/minimal@=1.2.3"
AtLeast = "biff/minimal@>=1.2.0"
Window  = "biff/minimal@>=1.2.0, <1.5.0"
```

Seven requirements against one package. Only the last needs a comma, and the comma means **and**.

| Requirement | Accepts | Reach for it when |
| --- | --- | --- |
| `^1.2.3` | `>=1.2.3, <2.0.0` | You trust the publisher's SemVer — the usual choice |
| `1.2.3` | `>=1.2.3, <2.0.0` | Never on purpose: it is a caret wearing a disguise |
| `~1.2.3` | `>=1.2.3, <1.3.0` | You want fixes but no new features |
| `=1.2.3` | That version and nothing else | A break would be expensive and you want no surprises |
| `>=1.2.0, <1.5.0` | Both halves must hold | A known-bad version sits above your range |
| `*` | Anything published | Almost never in a manifest you intend to keep |

Comma-separated requirements are ANDed, so a conjunction is always narrower than either half alone.

Look again at the second row. **A bare version is a caret requirement.** `"biff/minimal@1.2.0"` does not mean version 1.2.0; it means `^1.2.0`, and an install will happily choose 1.9.4. This is the single most common misreading of a Wally manifest, and it reads as a pin to everyone who has not been told. If you meant exactly one version, write `=1.2.0`.

The caret has one wrinkle worth memorising: it protects the **leftmost non-zero** part, not the major. So `^1.2.3` is `>=1.2.3, <2.0.0` as you would expect, but `^0.2.3` is `>=0.2.3, <0.3.0` — on a 0.x package the minor is treated as the breaking number, which is what the ecosystem's pre-1.0 libraries actually mean by it.

## Two surprises

| What you expect | What actually happens |
| --- | --- |
| The requirement you wrote is the requirement that gets recorded. | **It is not preserved.** `^1.2.3` is stored in the index as `>=1.2.3, <2.0.0`, because Wally normalises on display. An index entry never shows you a caret. |
| `@4.0.0` will pick up `4.0.0-rc.2` if that is all there is. | **It will not.** A prerelease only matches a requirement that itself mentions a prerelease. You have to ask for `@4.0.0-rc.2` by name. |

Neither is a bug. Both are places where reading a manifest at face value gives you the wrong answer.

**The same requirement, as the index records it**

```json
{"dependencies":{"Signal":"sleitnick/signal@>=2.0.0, <3.0.0"}}
```

The publisher typed `^2.0.0`. When you read someone's index entry, expect the expanded form and translate it back yourself.

The prerelease rule is the one that wastes an afternoon. A library ships `4.0.0-rc.2`, you write `"evaera/promise@^4.0.0"`, and the install fails as though nothing were published at all — because from the resolver's point of view nothing matching was. Prereleases are opt-in, one requirement at a time.

**Key points**

- A requirement is a set of acceptable versions; Wally chooses one from it at install time.
- A bare `"1.2.0"` means `^1.2.0`, not `=1.2.0` — write `=` when you mean a pin.
- The caret protects the leftmost non-zero part, so `^0.2.3` is `>=0.2.3, <0.3.0`.
- What you wrote is not what is stored: the index records `^1.2.3` as `>=1.2.3, <2.0.0`.
- Prereleases are opt-in — `@4.0.0` never matches `4.0.0-rc.2`, and neither does `@*`.
