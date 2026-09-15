# Getting the modules into your game

*Two generated modules, two realms, and one folder that both sides can actually see.*

Blink · Blink in a Real Project · 10 min

---

The two generated modules police their own realm: require the server one from a LocalScript and it errors, and the client one from the server errors with `Client network module can only be required from the client.` So each side has to be able to reach its own file. That is not the same as putting them in two places: the *client* module has to exist on the client, which means it has to be under something that replicates.

**default.project.json**

```json
{
  "name": "crosswire",
  "tree": {
    "$className": "DataModel",
    "ReplicatedStorage": {
      "Network": { "$path": "src/shared/Network" }
    },
    "ServerScriptService": {
      "Main": { "$path": "src/server" }
    }
  }
}
```

One folder holding both generated modules, under a service that replicates. The realm check does the rest.

## Build output, not source

> **These files are build output**
>
> Every generated module opens with `-- This file is not meant to be edited` and is overwritten on the next compile, so an edit survives exactly until someone runs Blink. Pick one policy: commit them, or generate them in CI and gitignore them. Doing both gives you a diff that changes depending on who pushed last.

Which to pick depends on who has to build the place. Committing them keeps it buildable by an artist with no toolchain installed, at the cost of a generated diff in every networking pull request. Generating them in CI keeps the diff honest, at the cost of a Rokit install in everyone's path. A team already running Rojo almost always commits them.

## The day-to-day loop

**terminal**

```luau
$ blink net --watch
Blink 0.18.8
Blink is watching for changes:
	Entry: net.blink
	Imports: 2
```

`--watch` takes over from the ordinary compile, so the four progress lines never print. Recompiles are silent; only diagnostics and a change in the number of imports print anything.

It watches the entry file and every file it imports, transitively: the watcher walks the `import` lines of each file it finds and adds them to its list, then compares modification times once a second.

**Key points**

- Each generated module refuses to load on the wrong side, so both can live in one folder.
- That folder has to replicate, which in practice means `ReplicatedStorage`.
- The generated files are build output: commit them or generate them in CI, never both.
- `--watch` replaces the ordinary compile, recompiles silently, and follows imports transitively.
