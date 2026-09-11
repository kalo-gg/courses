# wally publish

*Publish is one command, permanent, capped at 2 MiB, and it exits 0 even when it fails.*

Wally · Publishing and Registries · 11 min

---

The flow is two commands, and only one of them is per-project. `wally login` once on the machine, then `wally publish` from the directory holding your `wally.toml`. There is no dry-run flag, no version prompt and no confirmation: the version in the manifest is the version that gets published.

**The whole flow**

```bash
$ wally login                 # once per machine
$ wally publish               # from the directory holding wally.toml
```

Bump `version` in the manifest first. Publishing the same version twice is not a thing that can happen.

What `publish` actually does: it packs the project directory into a zip, applying the `include` and `exclude` filter the next lesson pulls apart; it rewrites the `name` in any `default.project.json` it packs to the package's short name; and it POSTs the archive to the registry's publish endpoint with your token in the header. The registry validates the manifest, checks that you own the scope, and appends a line to the index.

Two limits sit on that upload. The archive may be at most **2 MiB** — generous for source, reachable the moment a `.rbxm` or a folder of art is in the tree. And the result is **immutable**: republishing a version fails with `package already exists in index`, there is no delete, no overwrite, and no `wally yank` in 0.3.2 even though the registry's policy page discusses yanking. A mistake is fixed forwards, by publishing a new version.

**A publish that failed**

```bash
$ wally publish
Error: <status>

$ echo $?
0
```

Read that twice. The command reported an error and then told the shell it had succeeded.

> **A failed publish exits 0**
>
> `wally publish` prints `Error: <status>` when the registry rejects the upload, and then **returns success anyway**. Any CI step that trusts the exit code — which is every CI step — goes green on a publish that did not happen. Check the *output*, not the code: capture stdout and stderr and fail the job yourself if the word `Error` appears in it.

## What stops a publish

| The situation | What happens | The fix |
| --- | --- | --- |
| `private = true` in the manifest | The client refuses before uploading anything | Nothing — this is the setting working. Games should have it |
| That version already exists | `package already exists in index` | Bump `version` and publish again |
| The scope's `owners.json` does not list you | The registry rejects the upload | Get an owner to add your numeric GitHub id |
| The archive is over 2 MiB | The registry rejects it for size | `exclude` what does not belong in a package |

Only the first of these is caught locally. The other three cost you a round trip — and, thanks to the exit code, only announce themselves in the output.

> **`--token` is not in this release**
>
> Wally's README on the default branch documents `wally publish --token <token>`, which would be the obvious way to publish from CI. It is absent from 0.3.2. In the release the token comes from `wally login` or from the auth file the login wrote, so CI has to place that file itself.

**Key points**

- `wally login` once per machine, then `wally publish` from the project directory. There is no dry run and no confirmation.
- Publish packs the directory, rewrites a packed `default.project.json`'s name, and POSTs the archive with your token.
- The archive cap is **2 MiB**, and a published version is immutable: no republish, no delete, no `yank` in 0.3.2.
- **A failed publish exits 0.** CI must check the output for `Error`, because the exit code will lie to it.
- `--token` is on the default branch only; in 0.3.2 the token comes from the auth file.
