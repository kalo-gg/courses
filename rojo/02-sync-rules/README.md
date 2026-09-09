# 2. Sync Rules

*How file names become classes, how folders and init files nest, and what meta, model and data files add.*

Rojo · Easy

---

The project file says where a folder lands; the files inside decide what they become. A handful of naming rules cover every case: a suffix picks the script class, a directory is a Folder unless an init file claims it, a meta file adds properties to its neighbor, and JSON, text or CSV files turn into the matching value instances. This module walks through each rule with the sync explorer and has you lay out files by hand.

## Lessons

1. [Scripts and suffixes](01-scripts-and-suffixes.md) — .server.luau is a Script, .client.luau a LocalScript, plain .luau a ModuleScript; the suffix is stripped from the name.
2. [Folders and init files](02-folders-and-init-files.md) — A directory is a Folder, unless an init file turns the directory itself into a script with the other files as children.
3. [Meta files](03-meta-files.md) — Name.meta.json adds properties and attributes to its neighbor; init.meta.json can also change a Folder's class.
4. [Models and data files](04-models-and-data.md) — Describe small trees in .model.json, turn data files into ModuleScripts and values, and drop in saved .rbxm models.
