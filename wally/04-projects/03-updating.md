# Updating without wally update

*The update command exists in the docs and panics in the release, so here is what to do instead.*

Wally · Wally in a Real Project · 9 min

---

A dependency you wrote as `^2.0.0` has shipped `2.4.0` with the fix you have been waiting for. Your lockfile still says `2.0.1`, because a lockfile's whole job is to keep saying what it said. Every package manager you have ever used has one command for this, and Wally's README — the one on the default branch, the one GitHub shows you — documents one too.

**What actually happens**

```bash
$ wally update
thread 'main' panicked at 'not yet implemented'
```

The subcommand parses. Then its body runs, and its body is a `todo!()`.

> **`wally update` is not implemented in 0.3.2**
>
> The subcommand's body is a `todo!()`, so running it panics. It **is** implemented on the unreleased `main`, and the README on the default branch documents it as though it worked — which is the version of the README a learner lands on. Until a release carries the implementation, treat `wally update` as a command that does not exist.

## The upgrade that works

Everything `update` would have done, you can do by hand, because both of its inputs are text files you own. Change the requirement in `wally.toml` if the version you want is outside it. Delete `wally.lock` so the old resolution stops being reused. Run `wally install`, which resolves from scratch and writes a new lockfile.

**The upgrade, by hand**

```bash
$ $EDITOR wally.toml     # Signal = "sleitnick/signal@^2.4.0"
$ rm wally.lock
$ wally install
   Resolved 3 dependencies
  Generated lockfile
$ git diff wally.lock
```

Three commands and a review. The `git diff` is the step people skip and regret.

Deleting the lockfile matters more than it looks. A locked version is preferentially reused during resolution, so widening a requirement on its own often changes nothing: if `2.0.1` is already pinned and still satisfies the requirement you just wrote, the resolver keeps it, and you are left concluding that the upgrade did not take. Removing the lockfile removes that possibility entirely — there is nothing left to prefer.

The cost is that you re-resolve the **whole** graph, not one package. That is why the diff review is not optional: `git diff wally.lock` is the only place the other versions that moved are written down.

> **The install after this one wipes your types**
>
> `wally install` deletes and rebuilds the package directories, which throws away everything `wally-package-types` wrote. If you upgraded from a terminal rather than from the script you wrote in the last lesson, run the sourcemap and the type restore again before you wonder where your autocomplete went.

**Key points**

- `wally update` is documented on the default branch and its body is a `todo!()` in 0.3.2, so it panics.
- The implementation exists on the unreleased `main`; nothing you can install today has it.
- Upgrade by hand: edit the requirement, delete `wally.lock`, run `wally install`.
- Deleting the lockfile re-resolves the whole graph, so read `git diff wally.lock` before committing.
- That install wipes the package directories, so restore the types afterwards.
