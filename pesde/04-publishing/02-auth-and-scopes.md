# Authentication and scopes

*Logging in through GitHub, keeping a token per index, and how a scope is owned, shared and transferred.*

pesde · Publishing to the Registry · 9 min

---

The official registry authenticates you through GitHub. `pesde auth login` prints a code; you paste it into GitHub's device prompt in the browser, and pesde stores a token for that index. Everything else under `pesde auth` reads or clears that token — and `token` prints it, which is how a CI secret is made.

**The four auth commands**

```bash
pesde auth login          # a code is printed; paste it into GitHub
pesde auth whoami         # the GitHub user the stored token belongs to
pesde auth token          # print the token, e.g. to store as a CI secret
pesde auth logout         # forget it
```

`pesde auth token` prints a whole line — `token for https://github.com/pesde-pkg/index: "Bearer gho_…"` — and the value inside the quotes, `Bearer ` prefix included, is what a CI secret holds. Copy the quoted string, not the line.

Tokens are **per index**. `-i` sits on `pesde auth` itself, before the subcommand, and takes a URL or an alias from your `[indices]`; with no `-i` it means the current project's default index, or the machine-wide one from `pesde config default-index`. `login --token` stores a token directly and skips the browser — the CI form.

**A private index**

```bash
pesde auth -i internal login             # browser flow, for the internal index
pesde auth -i internal login --token "$PESDE_TOKEN"
pesde auth -i internal whoami
```

## Scopes

A scope is claimed by the first person to publish to it, and the claim lives in the index repository: each scope has a `scope.toml` whose `owners` lists GitHub user IDs, the true owner's first. To add a teammate, publish at least one package to the scope, then send a pull request to `pesde-pkg/index` adding their ID — and they must confirm being added. Transfer is the same pull request with a new first owner and their confirmation. Claiming a scope you have no intention of using is squatting and can cost you the scope and its packages.

**scope.toml, in `pesde-pkg/index`**

```toml
owners = [1234567, 7654321]
```

That is the whole file: `scope.toml` holds one field, `owners`, a list of GitHub user IDs with the true owner's first. There is nothing else in it to get wrong.

**Key points**

- `pesde auth login` is a GitHub device-code flow; `whoami`, `token`, `logout` manage the stored token.
- Tokens are per index: `pesde auth -i <alias> …`; `login --token` skips the browser.
- The first publisher owns the scope; team members are added by pull request to the index and must confirm.
- The true owner is first in `owners`; transfer is a PR plus confirmation; squatting loses the scope.
