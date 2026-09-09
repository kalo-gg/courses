# Engines and CI

*Pinning the runtimes a project needs, and the install, test and publish steps a pipeline should run.*

pesde · Workspaces, Tooling & CI · 11 min

---

Because pesde runs binary packages with a Luau runtime, it needs one — so it can install and manage them. An **engine** is a Luau runtime, or pesde itself, and declaring one both pins what your project uses and warns consumers whose version does not match.

**[engines]**

```toml
[engines]
pesde = "^0.6.0"
lune = "^0.8.9"
```

Only `pesde` and `lune` can be named. `pesde install` sets them up.

**What a mismatched consumer sees**

```text
warn: package acme/bar@0.1.0 lune requires lune ^0.8.9, but 0.9.0 is installed
```

> **This is how a team agrees on a pesde version**
>
> Rather than a toolchain manager — which the installation docs advise against for pesde — put `[engines.pesde]` in the manifest. Everyone who installs the project gets the check, and `pesde self-upgrade` remains the way to move.

## In CI

A pipeline runs the same commands you do, with three differences. It installs with `--locked`, so a manifest edited without its lockfile fails instead of resolving something new. It builds release artifacts with `--prod`, so dev tooling never ships.

And it publishes without a human: `pesde auth login --token` with a secret you made once from `pesde auth token`, `pesde publish --dry-run` on every pull request, `pesde publish -y` only on a release tag. Cache `~/.pesde` (or wherever `PESDE_HOME` points) between runs and the store keeps installs fast.

**ci.sh**

```bash
pesde install --locked        # fails on a drifted lockfile
pesde run test
pesde publish --dry-run       # every pull request

# on a release tag only
pesde auth login --token "$PESDE_TOKEN"
pesde publish -y
```

The token is a bearer credential for the index, not a device binding: the value `pesde auth token` printed on your laptop authenticates the same GitHub identity from a CI runner, which is exactly why it belongs in a secret store and is revoked from GitHub if it leaks.

**Key points**

- `[engines]` names `pesde` and `lune`; `pesde install` sets them up and consumers are warned on a mismatch.
- Pipelines install with `--locked`, build with `--prod`, dry-run on PRs and publish with `-y` on tags.
- `pesde auth login --token` with a secret from `pesde auth token` — the token works from any machine.
- Cache `PESDE_HOME` between runs.
