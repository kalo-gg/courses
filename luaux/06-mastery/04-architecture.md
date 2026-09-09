# Planning the settings screen

*Draw the file tree, decide where state lives, and write the config before the first tag.*

LuauX · Best Practices & Capstone · 10 min

---

The final project is a settings screen: toggles, a keybind list, a confirm dialog, wired into Rojo and ported to a second backend. Before any of it is typed there are three questions to answer, and each one is answered by a *file*: which library does the markup lower to, where does the state live, and what does Studio actually see. Get those three written down and the components are an afternoon.

**The tree, before the first tag**

```text
settings/
  luaux.toml              [build] src -> build · [factory] vide · [lints]
  default.project.json    Rojo: StarterPlayerScripts -> build/client
  src/client/
    Styles.luau           shared prop tables            (copied)
    Toggle.luaux          one control
    Row.luaux             one container
    SettingsPanel.luaux   the panel; forwards children
    KeybindList.luaux     one row per bind
    ConfirmDialog.luaux   a message and two buttons
    App.luaux             every source, and the whole tree
    main.client.luau      mounts App                    (copied)
  ports/fusion/
    luaux.toml            the same screen, Fusion's [factory]
    src/SettingsPanel.luaux
  build/                  generated: never edited, never committed
```

The layering is the one the Backends module built a screen with: values, then leaves, then containers, then `App`, then the mount. The new part is `ports/`. A `luaux.toml` describes one build, so a second target is a second build root with its own config — run `luaux build` from that directory and it compiles that tree with that block. Finding the config is automatic: LuauX walks **up** from the source root — or from the working directory when no path is given — taking the first `luaux.toml` it meets, and falling back to built-in defaults if it never meets one. So `cd ports/fusion && luaux build` is the whole of it.

**Two build roots, two configs**

```toml
# luaux.toml — the game
[build]
in = "src"
out = "build"

[factory]
backend = "table"
create = "create"

# ports/fusion/luaux.toml — the same markup, a different target
[build]
in = "src"
out = "build"

[factory]
backend = "table"
create = "scope:New"
children = "Children"
```

Two files, never merged: the paths inside each are relative to the directory that holds it, and the block is the only thing that differs. The Fusion block is longer for the reason the Backends module gave — its children go under a named key.

## The contract each file signs

| File | Owns | Must not |
| --- | --- | --- |
| `Styles.luau` | Shared prop tables | Markup, state or a require of a component |
| `Toggle.luaux` | One control's markup | Create its own source |
| `SettingsPanel.luaux` | Arrangement, and forwarding children | Know what a row contains |
| `App.luaux` | Every source, and the tree | Per-element styling |
| `luaux.toml` | Where the output goes, and the target | Renames nobody agreed to |
| `default.project.json` | What Studio sees | Point at `src/` |

Six files, six decisions, no overlap. Every argument about where something goes is settled by this table.

One thing is left before the first tag: the props. Three types, one per component the screen still needs, written where the analyzer can check them. They are the plan in its shortest form — and because a prop is either a value or a `vide.Source<T>`, writing them down settles the decision every lesson in this module turned on: which parts of the screen are allowed to change.

**Key points**

- Answer three questions before the first tag: which library, where the state lives, what Studio sees.
- One component per file, values in a copied `.luau` module, every source in `App`.
- A second backend is a second build root: its own `luaux.toml`, its own `in` and `out`.
- Write the prop types first — they are the plan, and they are the only part the analyzer checks.
