# Installing Wally

*Rokit installs the pinned CLI, and the version you get is the one this course is written against.*

Wally · Wally Foundations · 9 min

---

Wally is a single binary, and you could download it from the releases page and put it on your `PATH`. Do not. A Roblox project already needs Rojo, probably a formatter and a linter, and everyone on the team needs the *same* versions of all of them — which is exactly the problem a package manager solves, applied one level up.

A **toolchain manager** solves it. You commit a small manifest naming your tools and their versions; it installs those versions and shims them onto your `PATH`. A new contributor clones the repository, runs one command, and has your toolchain. The one to use today is **Rokit**.

**rokit.toml**

```toml
[tools]
wally = "UpliftGames/wally@0.3.2"
rojo = "rojo-rbx/rojo@7.7.0"
```

`rokit install` in this directory gets both, at exactly these versions. The file belongs in version control.

**0.3.2** is the version this course teaches. It was released on 5 June 2023 and it is still the latest tagged release. There are nine tags going back to 2021 and a rolling "Nightly" prerelease that tracks the unreleased default branch; the course teaches the tag, because that is what `rokit add UpliftGames/wally` gives you.

> **Both sets of install instructions are stale**
>
> Wally's install page and the README on its default branch recommend **Aftman**, which is archived and says so on its own first line. The README at the **v0.3.2 tag** — the release you are installing — recommends **Foreman**, which is Roblox's and has turned inward. Neither is the current answer. Rokit is a drop-in replacement for both, and it is what the rest of this course assumes.

## Checking it works

**The smoke test**

```bash
$ rokit install
$ wally --version
wally 0.3.2
```

If the version printed is not 0.3.2, something on your `PATH` is shadowing the pinned binary.

That is the whole installation. Wally keeps no global state you need to think about beyond an auth file, which only appears once you run `wally login` in Module 5. Everything else it does happens inside the project directory.

**Key points**

- Install Wally through a toolchain manager so the whole team gets one version; the manifest is `rokit.toml`.
- This course teaches **0.3.2**, released 5 June 2023 and still the latest tagged release.
- Wally's own install instructions recommend Aftman (archived) or Foreman (inward-facing) depending on which document you read. Use Rokit.
- `wally --version` should print `wally 0.3.2`; anything else means a stray binary is winning.
