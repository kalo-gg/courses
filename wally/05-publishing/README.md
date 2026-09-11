# 5. Publishing and Registries

*Claiming a scope, publishing an archive, and what it would take to run a registry of your own.*

Wally · Hard

---

Publishing is the half of a package manager most people never touch, and it is where the surprises are: what actually goes in the archive, who owns a name, what a failed publish looks like, and what it would take to run a registry of your own. It is one command, and everything interesting happens around it.

## Lessons

1. [Scopes and signing in](01-scopes-and-auth.md) — The registry checks a token and a scope, and owners.json is the file that decides who may publish.
2. [Running your own registry](02-self-hosting.md) — A registry is a git index plus an HTTP backend, and the index holds more of the decisions.
3. [wally publish](03-publishing.md) — Publish is one command, permanent, capped at 2 MiB, and it exits 0 even when it fails.
4. [What actually goes in the archive](04-what-ships.md) — include is not a whitelist, whatever the manifest says, so check before you publish.
