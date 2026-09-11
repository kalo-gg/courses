# Scopes and signing in

*The registry checks a token and a scope, and owners.json is the file that decides who may publish.*

Wally · Publishing and Registries · 10 min

---

Installing needs nothing from you: the index is a public git repository and the archives are behind a plain HTTP service. Publishing is the other direction, so the registry has to answer two questions before it will write anything. **Who are you**, and **may you publish under that scope?**

The first is answered by `wally login`. It runs GitHub's **device-code flow**: Wally prints a short code, you paste that code into a GitHub page in your browser and approve the app, and GitHub hands Wally a token back. Nothing is typed into the terminal, and no password ever passes through Wally.

**Signing in**

```bash
$ wally login
# Wally prints a code; paste it into GitHub and approve the app.
# The token it gets back is written to Wally's auth file.

$ wally logout
# Removes the stored token.
```

Once per machine, not once per project. The token lives outside your repository, which is the point — it must never be committed.

That auth file is the only global state Wally keeps. Everything else it does happens inside a project directory, which is why Module 1 could say installation was finished without mentioning it. `wally logout` removes the token again, and on a machine you share that is worth doing.

## Who owns a scope

Scopes are **claimed, not assigned**. There is no application form and no reservation list. The first time you publish to a scope that nobody owns yet, and whose name matches your GitHub login, the registry creates an `owners.json` for that scope and writes your numeric GitHub user id into it. From then on only the ids in that file may publish under it.

**owners.json**

```json
[
  1234567,
  7654321
]
```

One file per scope, at the top of that scope's directory in the index. Two ids means two people may publish here.

Sharing maintenance means adding ids to that file. They are **numeric user ids, not usernames**, so someone renaming their GitHub account does not lose their packages — and it also means you cannot add a collaborator by typing their handle. There is no web UI for `owners.json`; a change to it is a change to the index repository.

> **The scope has to match your login**
>
> You cannot claim `awesome-games` from an account called `biff`. An unclaimed scope is claimable only by the account whose login is that scope's name, so the scope in your manifest is a decision about which GitHub account will publish it — make it before the first publish, not after.

**Key points**

- `wally login` is a GitHub device-code flow; the token it returns is stored in Wally's auth file and removed by `wally logout`.
- The auth file is machine-wide and outside your repository, which is the only global state Wally keeps.
- A scope is claimed by its first successful publish, and only by the account whose GitHub login matches it.
- `owners.json` holds **numeric** GitHub user ids, so a rename costs nobody their packages.
- Sharing maintenance means adding another id to that file — there is no web UI for it.
