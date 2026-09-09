# 2. Entities, Components & Systems

*The core JECS API: worlds, entity lifetimes, components, tags, singletons and your first systems.*

JECS · Easy

---

This module is the heart of everyday JECS. You will learn how entity ids work (and why a deleted entity's id is safe to hold), how to add, read, update and remove component data, when to use tags and singletons, how to preregister components, and how to organize systems around the Roblox frame.

## Lessons

1. [Worlds and entities](01-worlds-and-entities.md) — Creating a world, spawning and deleting entities, and what an entity id really contains.
2. [Components and data](02-components-and-data.md) — set, get, has and remove; typing components; and the fact that components are entities too.
3. [Tags, singletons and preregistration](03-tags-and-singletons.md) — Zero-data components, world-level resources, and declaring ids before the world exists.
4. [Systems and the frame](04-systems-and-the-frame.md) — Writing systems as functions and running them from RunService every frame.
