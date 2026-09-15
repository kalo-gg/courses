# Blink in CI

*The compiler exits 0 on every error, so a CI job has to read the output instead.*

Blink · Tooling & Ecosystem · 10 min

---

Two things are worth proving on every push: that the description still compiles, and that the generated modules committed beside it still match it. Neither comes free, because a Blink failure does not look like a failure to a shell.

**.github/workflows/net.yml (the naive job)**

```text
- name: Compile the network description
  run: blink net -y
```

Green on a broken description. This is the job most projects ship.

Reading the errors showed why: Blink prints the diagnostic and leaves with status 0, so the shell sees a success. `-q` is no help either; it silences the four progress lines, not the diagnostic, which means the text you need is still on the output for the job to read.

> **A green tick on a broken description is worse than no tick**
>
> It is the check people stop reading. If the job cannot fail, delete it rather than trust it.

**.github/workflows/net.yml (the working shape)**

```text
- name: Compile the network description
  run: |
    out=$(blink net -y 2>&1)
    echo "$out"
    if echo "$out" | grep -qE '^\[E[0-9]{4}\]|\[Stack Begin\]'; then exit 1; fi
```

Two patterns, because two kinds of failure exist: a Blink diagnostic starts `[E####]`, and a bare Luau error prints `[Stack Begin]`.

## The drift check

Compiling proves the description parses. It does not prove that the `Server.luau` and `Client.luau` in the repository came from it. Regenerate in the job, then diff the output paths against what is committed: one line, and it is what stops the description changing while nobody re-runs Blink.

**The drift step**

```text
- name: Fail if the generated modules are stale
  run: git diff --exit-code src/shared/Network
```

`--exit-code` is the whole trick: a non-empty diff becomes a non-zero status, and the diff itself is printed in the log as the review you wanted.

**Key points**

- Blink prints diagnostics and exits 0; a CI step has to read the output, not the status.
- `-q` silences progress, not errors. `-y` accepts the prompts a terminal would ask.
- Grep for two prefixes: `[E####]` for a Blink diagnostic, `[Stack Begin]` for a bare Luau error.
- Regenerate and `git diff --exit-code` so the committed modules cannot drift from the description.
