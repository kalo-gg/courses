# 2. The Manifest

*Package specs, version requirements, three dependency tables and the realm rule that catches everyone.*

Wally · Easy

---

The manifest is the whole interface: a package spec says what you want, a dependency table says where it lands, and a realm says who is allowed to see it. This module reads each of those in turn, including the version syntax, which is Cargo's with two differences that will surprise you.

## Lessons

1. [Naming a package](01-package-specs.md) — The spec is scope/name@requirement, the alias is yours to choose, and the version half is required.
2. [Version requirements](02-version-requirements.md) — A bare version means caret, and two of Wally's semver rules differ from the Cargo it copies.
3. [Three tables](03-dependency-tables.md) — Three tables, three directories, and a dependency's own dev-dependencies never reach you.
4. [Realms](04-realms.md) — shared, server and dev, and the rule that catches people is not the one the docs state.
