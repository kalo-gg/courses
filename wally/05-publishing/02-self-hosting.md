# Running your own registry

*A registry is a git index plus an HTTP backend, and the index holds more of the decisions.*

Wally · Publishing and Registries · 10 min

---

"The registry" sounds like one service, and it is two things that barely know about each other. There is the **index**, a git repository of manifests that anyone can clone, and there is the **backend**, an HTTP service that authenticates publishes, stores archives and serves downloads. Running your own means running both.

**The shape of an index**

```text
wally-index/
  config.json              # the root: where the API is, which GitHub app to use
  bi/
    ff/
      owners.json          # who may publish under the scope "biff"
      biff-drop-table      # one file per package, one JSON line per version
      biff-minimal
```

Nothing here is a database. Every decision the index makes is a text file you can read in a diff.

`config.json` at the root is what a client reads first. It names the **API URL** the CLI should send publishes and downloads to, and the **GitHub OAuth app** it should authenticate against. Point a `registry` key at your index and those two values are how the CLI finds everything else — which is why a private registry needs no client configuration at all.

**config.json**

```json
{
  "api": "https://api.wally.run",
  "github_oauth_id": "…",
  "fallback_registries": [
    "https://github.com/UpliftGames/wally-index"
  ]
}
```

`fallback_registries` is the key that makes a private index practical: your packages resolve locally, everything else falls through to the public index.

That last key is worth pausing on, because people look for it in the wrong file. **`fallback_registries` is declared in the index, never in your `wally.toml`.** A consumer names exactly one `registry`; whether that registry knows how to fall back somewhere else is the registry's business, not the project's. It is also why a private index can be tiny: it holds only the packages you publish.

## The backend

The backend is the half with moving parts. It holds the GitHub OAuth credentials it checks tokens against, a handle on the index repository it appends to, and a **storage back end** for the archives themselves — a directory on disk is enough for a small team, and object storage is the answer once more than one instance is serving. Publishing and downloading both go through it; cloning the index does not.

> **The configuration is not written down**
>
> The registry backend's own README has a **"TODO"** where the configuration section should be. That is the honest state of self-hosting Wally: the pieces are open source and people do run them, but you will be reading source and environment-variable names rather than a guide. Budget for that before you promise anyone an internal registry.

So: is it worth it? Rarely, and the reason is the split above. Most people reaching for a private registry want *private packages*, and a git index is already an access-controlled git repository — the hard part is not the index, it is running and maintaining a backend whose configuration is undocumented. Understanding the shape is still useful even if you never run one, because it explains every error the public registry gives you.

**Key points**

- A Wally registry is a **git index** of manifests plus an **HTTP backend** that authenticates, stores and serves.
- `config.json` at the index root names the API URL and the GitHub OAuth app; a client needs nothing else.
- `fallback_registries` is declared in the **index**, never in `wally.toml`, so private packages and public ones can coexist.
- The backend holds OAuth credentials, index write access and a storage back end for archives.
- Its configuration is undocumented — the README says "TODO" — so self-hosting means reading source.
