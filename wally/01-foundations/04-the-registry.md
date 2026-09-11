# The registry behind it

*One git repository holds every published manifest, and knowing its shape explains a lot.*

Wally · Wally Foundations · 10 min

---

The `registry` key in your manifest is a git URL, and it means exactly what it says: the index is a **git repository**. You can clone it, read it, and `git log` it. There is an HTTP service in front for uploads and downloads, but the record of what exists is a directory of text files.

One file per package, holding newline-delimited JSON — one serialised manifest per published version, appended in publication order. Nothing is ever rewritten, so the file is also the package's history.

**An index entry, three versions of one package**

```json
{"package":{"name":"biff/minimal","version":"0.1.0","registry":"https://github.com/UpliftGames/wally-index","realm":"shared"},"dependencies":{},"server-dependencies":{},"dev-dependencies":{}}
{"package":{"name":"biff/minimal","version":"0.2.0","registry":"https://github.com/UpliftGames/wally-index","realm":"shared"},"dependencies":{},"server-dependencies":{},"dev-dependencies":{}}
{"package":{"name":"biff/minimal","version":"1.0.0","registry":"https://github.com/UpliftGames/wally-index","realm":"shared","description":"A minimal package","license":"MIT"},"dependencies":{"Signal":"sleitnick/signal@>=2.0.0, <3.0.0"},"server-dependencies":{},"dev-dependencies":{}}
```

Three lines, three versions. Note the requirement on the last line: whoever wrote that manifest typed `^2.0.0`, and the index stores the expanded form.

It is a busy repository. Round numbers, because precision here is false precision: roughly **1,500 scopes** and several thousand packages, taking somewhere around **26 publishes a day**. Two careful counts made on the same day will disagree, because a package and a published version are not the same thing and the index counts versions.

## Who owns a name

Scopes are claimed, not assigned. The first time you publish to an unclaimed scope whose name matches your GitHub login, the registry writes an `owners.json` for it and puts you in it. After that, only the ids in that file may publish under the scope, and adding a collaborator means adding their numeric GitHub user id.

**owners.json**

```json
[
  1234567,
  7654321
]
```

Numeric ids, not usernames — a rename does not cost anyone their packages. There is no web UI for this file.

> **Versions are immutable**
>
> Republishing a version fails with `package already exists in index`. There is no delete, no overwrite, and no `wally yank` command in 0.3.2 even though the registry's policy page mentions yanking. Publish carefully; Module 5 covers how to see the archive before you send it.

**Key points**

- The registry index is a plain git repository: one file per package, newline-delimited JSON, one line per version.
- Entries are appended and never rewritten, so the file doubles as the package's history.
- The index stores the *expanded* requirement, so `^2.0.0` appears there as `>=2.0.0, <3.0.0`.
- A scope is claimed by its first publish and owned through `owners.json`, which holds numeric GitHub ids.
- Published versions are immutable: no republish, no delete, and no `yank` command in this release.
