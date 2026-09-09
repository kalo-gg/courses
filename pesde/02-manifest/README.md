# 2. The Manifest

*Five ways to name a dependency, three tables to put it in, and the sections that tie them together.*

pesde · Easy

---

The manifest says who a project is, what it needs and where those things come from. Identity covers the top-level fields and the hyphen that silently means Wally; Targets covers lib, bin and the field a Roblox package cannot publish without; Five ways to name a dependency works through the registry, Wally, git, workspace and path specifiers; Dev, peer, and where to look separates the three dependency tables and names more than one registry.

## Lessons

1. [Identity: name, version, includes](01-identity.md) — The top-level fields that say who a package is, and the hyphen that silently turns a name into a Wally one.
2. [Targets: lib, bin and build_files](02-targets.md) — What a package exports for each environment, and the field a Roblox target cannot be published without.
3. [Five ways to name a dependency](03-specifiers.md) — Registry, Wally, git, workspace and path specifiers, and when each one is the right reach.
4. [Dev, peer, and where to look](04-dev-peer-and-indices.md) — Three dependency tables, what each means for a consumer, and naming more than one registry.
