# Linking instances: $id and Ref properties

*Give an instance an id and point a Ref property at it from JSON, the way Rojo has done since 7.5.0.*

Rojo · Structuring a Real Game · 10 min

---

Some properties hold another instance: an `ObjectValue`'s `Value`, a `Weld`'s `Part0`, a `Model`'s `PrimaryPart`. JSON has no way to write a pointer, so until 7.5.0 those properties could only be set in Studio. Now a project can name an instance with `$id` and, on the instance that refers to it, set an attribute called `Rojo_Target_<Property>` whose value is that id. Rojo resolves the pair into the real `Ref` when it builds or syncs; the attribute is the instruction, the property is the result.

**A spawn and something that points at it**

```json
"Workspace": {
  "Spawn": {
    "$className": "SpawnLocation",
    "$id": "spawn",
    "$properties": { "Anchored": true }
  },
  "SpawnRef": {
    "$className": "ObjectValue",
    "$attributes": { "Rojo_Target_Value": "spawn" }
  }
}
```

`$id` goes on the target; the attribute goes on the referrer and names the property to fill.

## Ids from meta files

File-backed instances get their id from a meta file instead: `id` in `Name.meta.json` for a script or value, in `init.meta.json` for a directory. Ids are plain strings and must be unique across the project, so name them after what they are — `spawn`, `hud-root` — not after where they sit.

The attribute side is the same whatever produced the referrer: `$attributes` in the project file, `attributes` in a meta file. A `.model.json` node may carry `attributes` too, and may declare an `id` of its own, so a model can be a target and point at one without leaving the file.

**src/shared/Remotes/init.meta.json**

```json
{
  "className": "Folder",
  "id": "remotes"
}
```

Anything can now point at `ReplicatedStorage.Shared.Remotes` with `Rojo_Target_<Property>: "remotes"`.

> **What the Build button shows**
>
> The in-browser build stores attributes as you wrote them and does not resolve the link, so the tree prints `Attributes={ Rojo_Target_Value = "spawn" }` rather than a filled `Value`. In Studio the property is set. Check the attribute here; check the property there. (There is a third spelling for the target side: an instance carrying a `Rojo_Id` attribute is marked as a target by that attribute's value, which is how something that arrived from a place file or a model can be pointed at without a `$id` or a meta file.)

**Key points**

- `$id` in the project file, or `id` in a meta file, names a target.
- `Rojo_Target_<Property>` on the referrer, holding the id, fills that Ref property.
- Ids are project-wide strings; name them by role.
- The course's build shows the attribute; Studio shows the resolved property.
