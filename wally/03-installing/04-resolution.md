# How a version gets chosen

*Greedy, first-fit, and no backtracking, which is why a stale lockfile can wedge an install.*

Wally · Installing and the Tree · 11 min

---

The resolver works **breadth-first** and **first-fit**. It starts from your manifest's direct dependencies, takes each requirement in turn, picks the first candidate version that satisfies it and **activates** it — then queues that package's own dependencies and carries on until the queue is empty. Versions already in `wally.lock` are seeded in first, so an unchanged lockfile keeps handing you the same tree.

Once a version is activated it is what everyone gets. There is at most **one semver-compatible copy** of any package in the tree, so a later requirement in the same compatible range has to be satisfied by the version already chosen. Incompatible majors are the exception — `1.4.0` and `2.1.0` are not compatible, so both may sit in `_Index` at once and each consumer links to the one it asked for.

**Two requirements on one package**

```toml
[dependencies]
Signal = "sleitnick/signal@=2.0.0"
Widgets = "biff/widgets@^1.0.0"

# biff/widgets@1.0.0 declares, in its own manifest:
#   Signal = "sleitnick/signal@^2.3.0"
```

Both requirements are reasonable on their own, and 2.3.0 exists. Nothing here is unsatisfiable in principle.

**What the failure looks like**

```bash
$ wally install
No packages were found that matched (Shared) sleitnick/signal@>=2.3.0, <3.0.0.
```

The pool has already been narrowed to the activated 2.0.0, so from the resolver's point of view nothing matches and the message reads like a missing package. A realm mistake produces the same sentence, which is worth remembering when you meet it again in Module 4.

## Why Cargo would have succeeded

Wally's version requirements come from Cargo's semver crate, so it is tempting to expect Cargo's resolver as well. Cargo **backtracks**: at a dead end it undoes a choice and re-solves with a different version. Wally has no such step. The first-fit choice is final, so the already-activated version wins and the later, stricter requirement simply fails — even though a solution existed and one different pick would have found it.

|  | Cargo | Wally |
| --- | --- | --- |
| Strategy | Search, with backtracking | Breadth-first, first-fit, single pass |
| At a dead end | Undoes a choice and re-solves | Reports the requirement it could not meet |
| Copies of one package | One per incompatible major | One per incompatible major |
| Your repair | Rarely needed | Delete `wally.lock` and install again |

The difference is one step, and it is the step that makes a resolver feel intelligent.

> **Delete the lockfile first**
>
> When an install that used to work starts reporting a package it cannot find, delete `wally.lock` and run `wally install` again before you touch the manifest. Locked versions are preferentially reused, so a stale one can be activated early and contradict a requirement that only arrived later. A resolver that could backtrack would recover from that by itself; this one needs you to clear the seed.

**Key points**

- Resolution is breadth-first and first-fit: the first version that satisfies a requirement is activated for the whole tree.
- At most one semver-compatible copy of a package exists; incompatible majors may coexist in `_Index`.
- There is no backtracking, so a later, stricter requirement fails rather than forcing an earlier pick to be redone.
- A conflict surfaces as `No packages were found that matched ...`, the same sentence a realm mistake produces.
- Locked versions are seeded first, so deleting `wally.lock` and reinstalling is the standard repair.
