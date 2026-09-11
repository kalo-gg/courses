# What is Wally?

*A package manager for Roblox: a manifest names your dependencies and a registry serves them.*

Wally · Wally Foundations · 9 min

---

Here is how a Roblox project usually acquires a library. Someone finds a model on the toolbox, or opens an uncopylocked place, drags a ModuleScript into `ReplicatedStorage`, and commits it. Six months later nobody can say which version it was, whether it was edited on the way in, or what breaks if it is replaced.

A **package manager** replaces that with a written declaration. You say which packages you want and which versions you will accept; the tool works out the rest of the graph, downloads it, and writes the result into your project. The declaration is a file called `wally.toml`, and it is the only part you edit by hand.

**wally.toml**

```toml
[package]
name = "biff/dungeon"
version = "0.1.0"
registry = "https://github.com/UpliftGames/wally-index"
realm = "shared"

[dependencies]
Signal = "sleitnick/signal@^2.0.0"
```

The smallest manifest that does something: who you are, where packages come from, and one dependency.

Three things follow from that file. Your dependencies are **reproducible** — the same manifest and lockfile produce the same tree on every machine. They are **transitive** — a package's own dependencies come along without your having to know they exist. And they are **shared** — the version you name is the version everyone else names, so bug reports mean something.

## What Wally is not

Roblox Studio has a feature called **Packages**, and it is not this. Studio Packages are instance reuse inside Roblox: you publish a model to the cloud and update copies of it across your places. Wally is a source package manager that fetches files at build time, before Studio is involved. The name collision is unfortunate and it confuses everyone once.

|  | Wally | Studio "Packages" |
| --- | --- | --- |
| What it moves | Source files, fetched from a registry | Instances, stored on the Roblox cloud |
| When | Before you open Studio | Inside Studio, on demand |
| Declared in | `wally.toml`, in your repository | Nowhere you can read as text |
| Versioned by | SemVer requirements you write | A Roblox-managed update prompt |

Two features with one name. They solve different problems and neither is built on the other.

**The division of labour**

```bash
wally install     # writes Packages/ from the manifest
rojo serve        # maps Packages/ into ReplicatedStorage
```

Wally produces a tree of files. Something else has to put that tree into a place — for almost everyone, that is Rojo.

> **Roblox does not ship one of these**
>
> There is no first-party source package manager. Roblox's own documentation covers Wally and Rokit under a disclaimer saying neither is maintained by Roblox. The community built this, and the community maintains it.

**Key points**

- A package manager replaces dragged-in ModuleScripts with a written, versioned declaration.
- `wally.toml` is the file you edit; everything else Wally touches is generated.
- You get reproducibility, transitive dependencies and a shared vocabulary of versions.
- Studio's "Packages" feature is instance reuse in the cloud and is unrelated despite the name.
- Wally writes files; Rojo puts them in the datamodel.
