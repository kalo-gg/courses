# 3. Buffers & Vectors

*Packed bytes and native three-component maths, for the data a table is the wrong shape for.*

Luau Advanced · Medium

---

A table of tables is the right shape for most data and the wrong shape for a lot of it. This module covers the two types Luau added for the rest: buffers, which are raw bytes you read and write by offset, and vectors, which are three numbers the VM treats as one value. It ends with the packed store the final project's simulation runs on.

## Lessons

1. [Raw bytes](01-buffers.md) — A buffer is a fixed block of bytes with no keys and no iteration: you decide what the layout means.
2. [Offsets and strides](02-reading-and-writing.md) — Index times stride plus field offset is the whole technique, and an out-of-range read errors rather than lying.
3. [Three numbers, one value](03-vectors.md) — vector is a VM primitive with its own library, read-only components, and no heap cost at all.
4. [Frozen tables](04-immutability.md) — table.freeze is shallow and in place, table.clone is shallow and never frozen, and both matter.
