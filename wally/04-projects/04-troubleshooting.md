# Reading the errors

*Four errors you will actually meet, what each one really means, and the one that lies about failing.*

Wally · Wally in a Real Project · 11 min

---

Wally's failures are terse, and two of the four you will meet most often describe something other than what went wrong. That is not carelessness — in both cases the error is an honest report from the layer that noticed, and the cause is a layer above. Learning to translate is worth more than memorising the text.

| What you see | What it actually is | What to do |
| --- | --- | --- |
| `No packages were found that matched (Shared) …`, then `Are you sure this is a Shared dependency?` | A realm mistake. The package exists; it was filtered out of the candidate pool before matching ever started. | Move the entry to `[server-dependencies]`. |
| Install stops on two requirements for one package that cannot both hold | The resolver is greedy and first-fit. A version was activated early, a later requirement contradicts it, and it will not go back and try again. | Delete `wally.lock` and install again, or loosen one of the two. |
| Install fails writing links for a package in `ServerPackages/` that depends on one in `Packages/` | A cross-realm link, and no `[place]` in the manifest to say where the two directories live. | Add `[place]` with both keys. |
| `Error: <status>` printed by `wally publish` — and an exit code of `0` | A failed publish that reported success. The status is in the output and nowhere else. | Grep the output in CI; never trust the exit code. |

Four errors, and only the fourth is dangerous because of what it does *not* say.

## The one that reads like a typo

**A realm mistake**

```text
No packages were found that matched (Shared) biff/profile-store@>=1.0.0, <2.0.0.
Are you sure this is a Shared dependency?
```

The first line sends you to check the spelling and the version. The second line is the one that tells you what happened.

`biff/profile-store` is in the index, at a version that satisfies the requirement, and the install still cannot find it. The reason is that the entry sits in `[dependencies]`, and `[dependencies]` accepts only packages whose own manifest declares `realm = "shared"`. This one declares `server`, so it is removed from the pool of candidates *before* any version matching happens — and what is left to report is that nothing matched.

Read the second line first. `(Shared)` in the first line is not describing the package; it is describing the table you put it in.

The resolver conflict is the second one, and it is the one that looks like Wally's fault. Two packages in your graph each depend on a third, with requirements that cannot both be satisfied. Cargo would backtrack here — unpick the version it activated first, and try a different one. Wally does not: resolution is breadth-first and first-fit, an activated version stays activated, and the later requirement simply fails.

The practical consequence is that a **stale lockfile** causes conflicts that the manifest alone would not, because locked versions are preferentially reused and get activated before the requirement that contradicts them is ever read. Deleting `wally.lock` and reinstalling is the standard repair, and it works more often than it has any right to.

The third is quieter. A package in `ServerPackages/` may depend on a shared package, which lives in `Packages/` — a different directory, in a different service, under a different realm. Wally writes that link by interpolating datamodel paths straight into the generated Luau, and the only place it can get those paths is `[place]`. Leave the table out and the link cannot be written.

**wally.toml**

```toml
[place]
shared-packages = "game.ReplicatedStorage.Packages"
server-packages = "game.ServerScriptService.ServerPackages"
```

These two strings must name exactly where your `default.project.json` put the directories. Nothing checks that they agree; the link just breaks at runtime if they do not.

## The one that exits 0

**A publish that failed and said it succeeded**

```bash
$ wally publish
Error: 403 Forbidden
$ echo $?
0
```

The status is printed. It is not returned.

> **CI that trusts the exit code passes on a failed publish**
>
> `wally publish` prints `Error: <status>` and then returns success. A release job whose last step is `run: wally publish` goes green, the tag is marked shipped, and the version is not in the index. Module 5 covers what to do about it; for now, know that the only signal is the text on stdout.

**Key points**

- A realm mistake reads as `No packages were found that matched`; the line about `Shared` is the real symptom.
- The resolver never backtracks, so delete `wally.lock` and reinstall, or loosen one of the two requirements.
- A link that crosses realms needs `[place]`, and its paths must agree with `default.project.json`.
- `wally publish` prints `Error:` and exits 0 — read the output, never the exit code.
