# Installing Rojo

*Pin the CLI with Rokit, install the matching Studio plugin, and check the versions line up.*

Rojo · Rojo Foundations · 8 min

---

Install the CLI through **Rokit**, the toolchain manager from the Rojo team. Rokit pins the exact Rojo version in a `rokit.toml` next to your project, so every collaborator and every CI run gets the same binary. Install Rokit once per machine, then add tools per project.

**Install Rokit, then Rojo**

```bash
# macOS / Linux
curl -sSf https://raw.githubusercontent.com/rojo-rbx/rokit/main/scripts/install.sh | bash
# Windows (PowerShell)
Invoke-RestMethod https://raw.githubusercontent.com/rojo-rbx/rokit/main/scripts/install.ps1 | Invoke-Expression

cd my-game
rokit init                  # writes an empty rokit.toml
rokit add rojo-rbx/rojo     # pins the latest release and installs it
rojo --version              # Rojo 7.7.0
```

**rokit.toml**

```toml
[tools]
rojo = "rojo-rbx/rojo@7.7.0"
```

Commit this file. On a fresh clone, `rokit install` reads it and fetches the pinned versions of every tool listed.

## The Studio plugin

The CLI can install its own plugin: `rojo plugin install` drops the matching `.rbxm` into Studio's local plugins folder. Restart Studio and a **Rojo** button appears in the Plugins tab.

If you prefer, the plugin is also on the Creator Store as “Rojo”, but installing from the CLI guarantees the versions match, which matters more since 7.7.0 changed the connection protocol.

> **Editor extensions**
>
> The Rojo VS Code extension adds a menu for starting and stopping the server and can manage the plugin, but it does **not** put `rojo` on your PATH. Install the CLI with Rokit either way, then let the extension call it.

**Key points**

- Install Rokit once, then `rokit add rojo-rbx/rojo` pins Rojo in `rokit.toml`; `rokit install` restores it on any clone.
- `rojo plugin install` installs the Studio plugin that matches the CLI.
- CLI and plugin versions must match; the VS Code extension does not install the CLI.
