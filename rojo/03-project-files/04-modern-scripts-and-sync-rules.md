# Modern scripts and sync rules

*emitLegacyScripts: false for RunContext scripts, custom syncRules, and JSON with comments.*

Rojo · Project Files in Depth · 9 min

---

Roblox added `RunContext` so a single `Script` class can run on the server, the client or in a plugin. Rojo still emits the classic pair (`Script` and `LocalScript`) by default for compatibility, because a `LocalScript` in `StarterPlayerScripts` and a client `Script` in `ReplicatedStorage` behave differently. Set `"emitLegacyScripts": false` and `.server.luau` becomes a `Script` with `RunContext = Server`, `.client.luau` a `Script` with `RunContext = Client`. ModuleScripts are unchanged either way.

**One key**

```json
{
  "name": "arena",
  "emitLegacyScripts": false,
  "tree": { "$className": "DataModel", "ServerScriptService": { "$path": "src/server" } }
}
```

| File | emitLegacyScripts: true (default) | emitLegacyScripts: false |
| --- | --- | --- |
| `X.server.luau` | Script, RunContext Legacy | Script, RunContext Server |
| `X.client.luau` | LocalScript | Script, RunContext Client |
| `X.plugin.luau` | Script, RunContext Plugin | Script, RunContext Plugin |
| `X.luau` | ModuleScript | ModuleScript |

## Your own file rules

Since 7.5.0, `syncRules` lets a project add rules that run before the defaults. Each rule has a glob `pattern`, a `use` naming one of Rojo's file handlers (`serverScript`, `clientScript`, `moduleScript`, `pluginScript`, `json`, `jsonModel`, `project`, `toml`, `yaml`, `csv`, `text`, `rbxm`, `rbxmx` or `ignore`; four more, `legacyServerScript`, `legacyClientScript`, `runContextServerScript` and `runContextClientScript`, force one script form regardless of `emitLegacyScripts`), an optional `exclude` glob and an optional `suffix` to strip from the name.

```json
"syncRules": [
  { "pattern": "*.md", "use": "text", "suffix": ".md" },
  { "pattern": "*.story.luau", "use": "ignore" }
]
```

Use them to sync `.md` files as StringValues, treat `*.spec.luau` as ignored, or make `.script.luau` mean server script in a codebase that prefers it. Without `suffix`, Rojo strips only the extension, so a multi-part ending such as `.story.luau` needs it spelled out.

> **These two settings stop at every project file**
>
> `syncRules` and `emitLegacyScripts` are **not** project-wide. Every time Rojo reaches another `*.project.json` through a `$path` it builds a fresh context: `syncRules` are cleared outright (7.5.0's changelog: “All sync rules are reset between project files, so they must be specified in each one when nesting them”), and `emitLegacyScripts` is overwritten by that file's value — which means a nested project that simply omits the key snaps back to `true`. This bites where you least expect it, because almost every Wally package ships a `default.project.json`: your `.client.luau` files become RunContext Scripts, and the package's stay LocalScripts. `globIgnorePaths` are the odd one out — those accumulate, each relative to the project file that declared them. If you need a rule everywhere, copy it into every project file you own.

> **Comments in JSON**
>
> Project, meta and model files accept `//` and `/* */` comments plus trailing commas since 7.6.1, and `.jsonc` extensions since 7.7.0. Explain a strange `syncRules` entry right where it lives. `$schema` is accepted too (7.5.0) in project, model and meta files, so an editor's schema link is legal rather than an unknown key.

**Key points**

- `emitLegacyScripts: false` emits `Script` with `RunContext` Server/Client instead of Script/LocalScript.
- `syncRules` adds `pattern`/`use`/`exclude`/`suffix` rules checked before the defaults.
- Comments and trailing commas are fine in project, meta and model JSON since 7.6.1.
