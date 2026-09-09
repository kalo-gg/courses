# Composing trees

*Mixing $path with pure children, $className on a Folder path, optional paths, and nested project files.*

Rojo · Project Files in Depth · 10 min

---

A node described entirely in the project file, with no `$path`, is a **pure node**: `Workspace` with its properties, or a `Folder` you declare by hand. A node can also pull in a folder **and** list extra pure children. Rojo merges them: everything from disk, plus the children you wrote, under one instance. A child listed in the project sits **beside** anything `$path` brought in rather than overriding it. Rojo does not complain about two siblings called `Util`; it simply builds both.

```json
"ReplicatedStorage": {
  "Shared": {
    "$path": "src/shared",
    "Remotes": { "$className": "Folder" }
  }
}
```

`Shared` gets every file from `src/shared` plus an empty `Remotes` Folder that scripts fill at runtime.

`$className` and `$path` can meet on one node only when the path would be a Folder; then the explicit class replaces Folder and the files stay as children. `{ "$className": "ScreenGui", "$path": "src/gui" }` is the project-file way of doing what `init.meta.json` does on disk.

## Optional paths and nested projects

Wrap a path as `{ "optional": "Packages" }` and Rojo skips the node silently when the folder is missing, instead of failing. That is how a project references Wally's `Packages` folder before anyone has run `wally install`. A `$path` can also point at another `*.project.json`, or at a directory containing a `default.project.json`; the nested project's whole tree is inserted there.

**Optional and nested**

```json
"ReplicatedStorage": {
  "Packages": { "$path": { "optional": "Packages" } },
  "Promise":  { "$path": "vendor/promise.project.json" }
}
```

`Packages` vanishes silently while the folder is missing; `Promise` inserts the whole tree that `promise.project.json` describes, under a fresh settings context.

> **Never point a project at its own folder**
>
> A directory containing a `default.project.json` *is* that project, so `{ "tree": { "$path": "." } }` asks Rojo to build the project by building the project. There is no cycle check: it recurses until the process dies with `thread 'main' has overflowed its stack`, with nothing to say which file caused it. A nested project's `$path` values must point at the files inside it, never at the folder it lives in.

**Key points**

- A node can combine `$path` with pure children, which sit beside each other rather than overriding — so keep names distinct yourself, because Rojo will not.
- `$className` on a Folder path replaces the Folder class and keeps the children.
- `{ "optional": "…" }` paths are skipped when missing; a `$path` to a project file inserts that project's tree.
