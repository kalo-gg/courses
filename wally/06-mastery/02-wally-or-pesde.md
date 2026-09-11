# Wally or pesde

*Two real answers: adoption is Wally's, development is pesde's, and each carries a named risk.*

Wally · Best Practices & Capstone · 10 min

---

There are two package managers for Luau, and every Roblox project started this year has to pick one. Kalo teaches both, so this lesson is not going to tell you which. What it will do is give you the numbers, name the risk on each side, and be clear about which of them is a real risk and which is a preference.

**wally.toml**

```toml
[package]
name = "biff/dungeon"
version = "1.4.0"
registry = "https://github.com/UpliftGames/wally-index"
realm = "shared"

[dependencies]
Signal = "sleitnick/signal@^2.0.0"
```

Wally's manifest is four required keys and a table of `alias = "scope/name@requirement"` lines. pesde's dependency is a table-valued specifier instead, and its package declares a `[target]` naming the runtime it is for — a distinction Wally does not have, because Wally is Roblox-only.

**Adoption is Wally's, decisively.** Its index holds roughly **1,500 scopes** against pesde's **180**. Take twelve libraries a Roblox project actually reaches for — jecs, Fusion, Matter, Knit, Promise, Roact, React, TestEZ, Jest, Charm, Ripple, Vide — and **twelve are on Wally and one is on pesde**; that one is Vide, and its pesde copy is a version behind its Wally copy. Ratios rather than package counts, deliberately: two careful counts of an index disagree, because a package and a published version are not the same thing.

|  | Wally | pesde |
| --- | --- | --- |
| Last release | 0.3.2, June 2023 | 0.7.3, March 2026 |
| Index scale | Roughly 1,500 scopes | Roughly 180 scopes |
| Runtime targets | Roblox only | Roblox, Lune, standalone Luau |
| `add` / `remove` | Neither — you edit the manifest | Both |
| `yank` | No such command | `pesde yank`, and `--undo` to reverse it |
| Reads the other's registry | No | Yes, once `[wally_indices]` is declared |

Six rows, and they do not all point the same way. The first two rows are one argument; the last four are the other.

**Development is pesde's, just as decisively.** Wally's client has had no release since **June 2023**, and three features its own README documents are missing from the release you install: `wally update` panics, `install --locked` does not exist, and `publish --token` is not there. pesde ships regularly, and it has runtime targets, git, path and workspace dependencies, patches, overrides, real `add` and `remove`, and `yank`.

## Two risks, and both are real

> **Name the risk on each side**
>
> **Wally's is a frozen client.** An unfixed bug stays unfixed: there is nowhere for a patch to land, and an open build issue from August 2026 has no maintainer answer. Note carefully that this is not a deprecation — no maintainer has said anything either way — but it is a fact you inherit.
>
> **pesde's is concentration.** One person has written essentially all of it, and the manifest is pre-1.0 with a breaking change already in progress on the main branch. That is not a criticism of the work, which is good; it is a description of what you would be depending on.

## They are not exclusive

This is the practical point, and it is the one most arguments skip. pesde ships Wally support **compiled in**: a Wally package is a first-class dependency kind, not a plugin. It is not on by default, though — a pesde project must declare the Wally index before a Wally specifier resolves at all, and Kalo's pesde course grades exactly that step.

Roughly **one pesde package in seven** declares a Wally dependency, counted by walking the pesde index. Read that the other way round and it says something plain: a pesde project reaches the Roblox ecosystem *through* Wally's registry. There is no migration tool in either direction, and none is planned.

**pesde.toml**

```toml
[wally_indices]
default = "https://github.com/UpliftGames/wally-index"

[dependencies]
signal = { wally = "sleitnick/signal", version = "^2.0.0" }

[scripts]
sourcemap_generator = ".pesde/scripts/sourcemap_generator.luau"
```

The same dependency as the first block, installed by pesde. The index has to be named, and without a `sourcemap_generator` the Wally package has no entry point either — a Wally package carries no `pesde.toml` to say which file is the library.

> **The complaint is real, and it is not going where you would expect**
>
> A 2026 developer-forum thread has someone calling Wally abandoned, on three charges that are all true: no release since 2023, a README recommending an installer that is archived, and an open build issue. Another poster answers that the silence is stability rather than neglect. What is striking is the negative space — **pesde is not mentioned once in that entire thread**. The frustration is producing *new registries* instead, and each of them mirrors Wally's index rather than pesde's, which is a fair picture of where the ecosystem's centre of gravity actually sits.

And nobody is coming to settle it. Roblox ships no package manager of its own; its documentation covers Wally and Rokit under a disclaimer saying neither is maintained by Roblox, and does not mention pesde at all. Roblox's own Lute README says its libraries will be split out "once a good dependency management solution is in place" — the platform saying out loud that it has left this to the community.

So: no verdict here, because both answers are real. If what you need is the **ecosystem** — the packages, the tutorials, the person who can answer your question — that is Wally. If what you need is the **tooling** — runtime targets, workspaces, patches, an `update` that works — that is pesde, and Kalo's **pesde** course teaches it end to end. Choose deliberately, because there is no tool that will move you afterwards.

**Key points**

- Adoption is Wally's: roughly 1,500 scopes against pesde's 180, and twelve of twelve libraries a Roblox project reaches for are on Wally against one on pesde.
- Development is pesde's: Wally has had no release since June 2023, and three features its own README documents are missing from it.
- Each carries a real risk — Wally a frozen client, pesde a single author and a pre-1.0 manifest with a breaking change already in progress.
- They are not exclusive: pesde installs Wally packages once `[wally_indices]` is declared, so a pesde project still reaches Roblox through Wally's registry.
- Roblox ships no package manager and names neither as maintained by it, so the choice is yours: the ecosystem, or the tooling.
