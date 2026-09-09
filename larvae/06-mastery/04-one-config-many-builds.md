# One config, many builds

*A base, a leaf, overrides and profiles: the layout that gives every build of Lantern one source of truth.*

Larvae · Larvae for a Team · 12 min

---

Everything Lantern needs is now in the course; the question is where each line lives. The config has three layers and each has a job. The base holds what the studio decides for every repo: formatting, lint levels, the rules every build gets. The leaf — the repo's own `larvae.toml` — holds what is true of this tree: input and output, aliases, the require shape, its worms. Profiles hold what differs between builds of the same tree.

**tooling/larvae.base.toml**

```toml
# what the studio decides once, for every repo
[fmt]
recommended = false
column_width = 100

[lint]
std = "roblox"

[lint.rules]
global_usage = "deny"

[rules]
use_get_service = true
add_luau_directive = "strict"
```

**larvae.toml, the leaf**

```toml
extends = "./tooling/larvae.base.toml"
input = "src"
output = "dist"
target = "roblox-string"
exclude = ["Packages"]

[aliases]
pkg = "@game/ReplicatedStorage/Packages"

[requires.overrides]
"tools/**" = "path"

[worms.luaux]
repo = "larvae-luau/luaux-worm"
version = "^"

[profile.release]
defines = { DEBUG = false }

[profile.release.process]
generator = "dense"
```

## What goes where

| Line | Layer | Why |
| --- | --- | --- |
| `[fmt]`, `[lint.rules]`, `[rules]` | Base | A style is a studio decision: one file, every repo |
| `input`, `output`, `[aliases]` | Leaf | True of this tree and no other |
| `[worms.*]` | Leaf | A worm serves the files this repo has |
| `[requires.overrides]` | Leaf | A corner of this tree |
| `defines`, `generator`, `[minify]` | Profile | Differs between builds of the same tree |
| `[check]` | Base for policy, leaf for `entries` | `cycles = "deny"` is everyone's; an entry point is this repo's |

Two rules keep the layout honest. A profile changes scalars and adds tables; it never repeats a list from the root, because an array in a profile replaces the root's rather than extending it — so a profile that needs one more glob writes the whole list. And the leaf never restates the base: if Lantern needs a different `column_width`, that is one key in the leaf, not a pasted `[fmt]` table. The diff between the two files is the whole story of what makes this repo different.

The final project assembles exactly this: Lantern's `larvae.toml` from Project 1, grown into a leaf over a base, with a release profile, the gate, the worm, the editor's settings, and the workflow that runs them. Every task is a line in one of the three layers, and the table above says which.

**Key points**

- Base: what the studio decides for every repo. Leaf: what is true of this tree. Profile: what differs between builds of it.
- Tables merge key by key across the layers; arrays and scalars are replaced whole.
- A profile never repeats a list from the root without meaning to replace it; a leaf never restates the base.
- The diff between base and leaf is the story of the repo; the diff between root and profile is the story of the build.
